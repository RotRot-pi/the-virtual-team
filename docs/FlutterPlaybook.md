# Flutter Engineering Playbook

Status: Reference
Version: v1.0
Owner: Cog-Flutter-1 (Engineering)
Date: 2025-08-30
Related: PRD (docs/PRD.md) • Roadmap (docs/Roadmap.md) • ArchitectureOverview (docs/ArchitectureOverview.md) • CodingStandards (docs/CodingStandards.md)

Purpose
- Make Cog-Flutter-1 behave like a reliable senior engineer: consistent structure, offline-first, robust error handling, solid tests, and clean diffs.
- This playbook adds tactical specifics on top of protocols/COG-FLUTTER.md. Follow both.

Scope & assumptions
- Stack: Flutter (latest stable), Riverpod Notifier/AsyncNotifier with codegen, go_router, Drift (SQLite), Dio + Retrofit, Freezed.
- Gate = Baseline: docs can be drafted anytime; code/external ops only after approval (see protocols/ANNEX.md).
- Dependency discipline: ask before adding packages beyond the default stack.

------------------------------------------------------------
1) Feature Module skeleton (structure + naming)
------------------------------------------------------------

Module = vertical slice: presentation + state + data + tests.

Prefer:
- lib/src/features/<module>/{presentation,state,data}/
- test/src/features/<module>/ mirrors lib structure
- snake_case files; one public type per file; small barrel exports only if helpful

Example tree (todos):
```
lib/
  src/
    features/
      todos/
        presentation/
          screens/
            todos_screen.dart
            todo_detail_screen.dart
          widgets/
            todo_list.dart
            todo_item.dart
          routing.dart
          l10n.dart (opt.)
        state/
          todos_vm.dart
          providers.dart
        data/
          dtos/
            todo_dto.dart
          mappers/
            todo_mapper.dart
          repository.dart
          sources/
            remote_service.dart
            local_dao.dart
          drift/
            todos_table.dart
        // Optional if you keep domain separate
        // domain/
        //   models/todo.dart
```

Tests mirror lib:
```
test/
  src/
    features/
      todos/
        data/
          repository_test.dart
          mappers_test.dart
        state/
          todos_vm_test.dart
        presentation/
          // optional: widget/golden tests
```

Naming
- Providers: <entity>RepositoryProvider, <module>VmProvider, <service>Provider
- ViewModels: <module>Vm extends Notifier or AsyncNotifier
- Files: <thing>_vm.dart, repository.dart, remote_service.dart, local_dao.dart, routing.dart

------------------------------------------------------------
2) Routing (go_router)
------------------------------------------------------------

Patterns
- Define all routes for the module in presentation/routing.dart
- Use typed params and path segments; prefer name-based navigation
- Guards/redirection live in small functions; use Riverpod to read auth/state

Example:
```dart
// routing.dart
import 'package:go_router/go_router.dart';
import 'package:flutter/widgets.dart';
import 'screens/todos_screen.dart';
import 'screens/todo_detail_screen.dart';

enum TodosRoute { list, detail }

List<GoRoute> todosRoutes() => [
  GoRoute(
    name: TodosRoute.list.name,
    path: '/todos',
    builder: (ctx, st) => const TodosScreen(),
  ),
  GoRoute(
    name: TodosRoute.detail.name,
    path: '/todos/:id',
    builder: (ctx, st) => TodoDetailScreen(id: st.pathParameters['id']!),
  ),
];
```

------------------------------------------------------------
3) State (Riverpod) — Notifier vs AsyncNotifier
------------------------------------------------------------

When to use:
- Notifier<T>: pure local state, synchronous logic
- AsyncNotifier<T>: IO-bound state (repo calls), handles loading/error/data

Guidelines
- Use autoDispose for transient/detail screens; keep top-level screens non-autoDispose
- Use ref.watch.select((s) => s.field) to minimize rebuilds
- UI renders states; ViewModel handles errors and maps to user messages

Example AsyncNotifier:
```dart
// todos_vm.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../data/repository.dart';
part 'todos_vm.g.dart';

@riverpod
class TodosVm extends _$TodosVm {
  @override
  FutureOr<List<Todo>> build() async {
    final repo = ref.read(todosRepositoryProvider);
    return repo.getTodos(); // cache-first refresh pattern
  }

  Future<void> addTodo(Todo input) async {
    final repo = ref.read(todosRepositoryProvider);
    // optimistic update
    final current = state.value ?? [];
    state = AsyncData([...current, input.copyWith(isPending: true)]);
    final res = await repo.createTodo(input);
    // on success refresh; on error revert + surface error
    state = await AsyncValue.guard(() async => repo.getTodos(forceRefresh: true));
  }
}
```

------------------------------------------------------------
4) Data layer (Freezed + Retrofit + Drift + Repository)
------------------------------------------------------------

DTOs (Freezed) and mapping:
```dart
// dtos/todo_dto.dart
import 'package:freezed_annotation/freezed_annotation.dart';
part 'todo_dto.freezed.dart';
part 'todo_dto.g.dart';

@freezed
class TodoDto with _$TodoDto {
  const factory TodoDto({
    required String id,
    required String title,
    @Default(false) bool completed,
    String? updatedAt,
  }) = _TodoDto;

  factory TodoDto.fromJson(Map<String, dynamic> json) => _$TodoDtoFromJson(json);
}
```

Retrofit service (Dio):
```dart
// sources/remote_service.dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import '../dtos/todo_dto.dart';
part 'remote_service.g.dart';

@RestApi()
abstract class TodosRemoteService {
  factory TodosRemoteService(Dio dio, {String baseUrl}) = _TodosRemoteService;

  @GET('/todos')
  Future<List<TodoDto>> getTodos();

  @POST('/todos')
  Future<TodoDto> create(@Body() TodoDto dto);
}
```

Drift schema + DAO:
```dart
// drift/todos_table.dart
import 'package:drift/drift.dart';

class TodosTable extends Table {
  TextColumn get id => text()();
  TextColumn get title => text()();
  BoolColumn get completed => boolean().withDefault(const Constant(false))();
  TextColumn get updatedAt => text().nullable()();
  @override
  Set<Column> get primaryKey => {id};
}

// local_dao.dart
import 'package:drift/drift.dart';
part 'local_dao.g.dart';

@DriftAccessor(tables: [TodosTable])
class TodosDao extends DatabaseAccessor<AppDb> with _$TodosDaoMixin {
  TodosDao(AppDb db) : super(db);

  Future<List<TodoRow>> getAll() => (select(todosTable)).get();
  Future<void> upsert(List<TodosTableCompanion> rows) async {
    await batch((b) => b.insertAllOnConflictUpdate(todosTable, rows));
  }
}
```

Repository glue:
```dart
// repository.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'sources/remote_service.dart';
import 'sources/local_dao.dart';
import 'dtos/todo_dto.dart';
part 'repository.g.dart';

abstract class TodosRepository {
  Future<List<Todo>> getTodos({bool forceRefresh = false});
  Future<Todo> createTodo(Todo input);
}

class TodosRepositoryImpl implements TodosRepository {
  TodosRepositoryImpl(this._remote, this._dao);
  final TodosRemoteService _remote;
  final TodosDao _dao;

  @override
  Future<List<Todo>> getTodos({bool forceRefresh = false}) async {
    final local = await _dao.getAll();
    if (!forceRefresh && local.isNotEmpty) {
      _refreshInBackground();
      return local.map(mapRowToDomain).toList();
    }
    final remote = await _remote.getTodos();
    await _dao.upsert(remote.map(mapDtoToCompanion).toList());
    final fresh = await _dao.getAll();
    return fresh.map(mapRowToDomain).toList();
  }

  @override
  Future<Todo> createTodo(Todo input) async {
    // Outbox handles offline; see Section 5
    final dto = mapDomainToDto(input);
    final created = await _remote.create(dto);
    await _dao.upsert([mapDtoToCompanion(created)]);
    return mapDtoToDomain(created);
  }

  void _refreshInBackground() async {
    try {
      final remote = await _remote.getTodos();
      await _dao.upsert(remote.map(mapDtoToCompanion).toList());
    } catch (_) {/* swallow */}
  }
}

@riverpod
TodosRepository todosRepository(TodosRepositoryRef ref) {
  final dio = ref.read(dioProvider); // interceptor-configured
  final remote = TodosRemoteService(dio);
  final dao = ref.read(todosDaoProvider);
  return TodosRepositoryImpl(remote, dao);
}
```

Mapping guidelines
- Keep DTO ↔ domain mappers pure and test them
- Convert timestamps to DateTime in domain; store as ISO-8601 strings in DB

------------------------------------------------------------
5) Offline-first: Outbox (optimistic writes)
------------------------------------------------------------

Outbox table (Drift):
```
id (TEXT, PK)
operation (TEXT) // e.g., create_todo
entity_type (TEXT) // e.g., todo
payload_json (TEXT)
attempt_count (INTEGER)
status (TEXT) // pending|retry|failed|done
last_error (TEXT, nullable)
created_at (TEXT ISO-8601)
updated_at (TEXT ISO-8601)
```

Flow
- On write (create/update/delete):
  1) Enqueue item in outbox; apply optimistic UI state
  2) Try to send immediately; on failure → backoff and mark retry
  3) On success → mark done and reconcile server copy to DB
  4) On unrecoverable error → mark failed; revert optimistic state; surface error to ViewModel

Worker
- Trigger on app resume, connectivity regain, or periodic timer
- Exponential backoff with jitter; cap attempts (e.g., 5)
- Idempotency: use a client-generated UUID per operation; server should accept duplicate-safe requests

Conflict policy
- Default: server-wins (accept server version and refresh local)
- Exceptions: if user has unsynced edits, prompt or merge (document in ADR if needed)

------------------------------------------------------------
6) Error mapping contract
------------------------------------------------------------

DomainError (example):
```dart
sealed class DomainError {
  const DomainError();
  factory DomainError.network() = _Network;
  factory DomainError.http(int code, {String? message}) = _Http;
  factory DomainError.serialization() = _Serialization;
  factory DomainError.validation(String message) = _Validation;
  factory DomainError.cache() = _Cache;
  factory DomainError.permission() = _Permission;
  factory DomainError.unknown([Object? cause]) = _Unknown;
}
```

Dio translation (interceptor or helper):
- SocketException/timeout → network
- 4xx → http(code) or validation
- 5xx → http(code)
- Format exceptions → serialization
- Map to user-friendly messages in ViewModel (copy from Architecture Error Mapping guidelines)

------------------------------------------------------------
7) Networking defaults (Dio/Retrofit)
------------------------------------------------------------

- Timeouts: connect 5s, receive 10s (tune per API)
- Interceptors:
  - auth header injector
  - error translation to DomainError
  - logging (debug only; hide secrets)
- Retries: at most 2 for idempotent GETs with jitter
- Cancellation: pass CancelToken tied to widget lifecycle for in-flight calls
- BaseUrl: injectable via provider; keep env/config outside code where possible

------------------------------------------------------------
8) Codegen and commands
------------------------------------------------------------

- Do not edit generated files (*.g.dart, *.freezed.dart, *.gr.dart)
- Run (human or CI):
```
dart run build_runner build --delete-conflicting-outputs
```

------------------------------------------------------------
9) Testing expectations
------------------------------------------------------------

Repo tests
- In-memory Drift; fake network (e.g., stub Dio/Retrofit)
- Cover happy path, offline cache, remote error handling

ViewModel tests
- Use ProviderContainer; override repo provider with fake/spy
- Verify loading → data/error; optimistic update flows and revert-on-failure

Mapper tests
- DTO ↔ domain roundtrips; edge cases (missing/extra fields)

Optional dev dependencies (ask before adding)
- mocktail for fakes/mocks
- http_mock_adapter for Dio
- golden_toolkit for widget goldens

Example ViewModel test skeleton:
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:riverpod/riverpod.dart';

void main() {
  test('loads todos', () async {
    final container = ProviderContainer(overrides: [
      todosRepositoryProvider.overrideWithValue(FakeTodosRepo()),
    ]);
    addTearDown(container.dispose);

    final vm = container.read(todosVmProvider.notifier);
    final value = await container.read(todosVmProvider.future);
    expect(value, isNotEmpty);
  });
}
```

------------------------------------------------------------
10) Web/Desktop notes
------------------------------------------------------------

- Drift on web: use sqlite3/wasm; ensure proper bundling; or fall back to memory for demos
- CORS: ensure server allows web origins; handle Dio browser specifics
- Plugins: avoid platform-only plugins without guards; keep conditional imports where needed

------------------------------------------------------------
11) Performance & A11y checklist (quick)
------------------------------------------------------------

- Rebuilds: use select(); const constructors; split widgets; keys where needed
- Lists: prefer ListView.builder; cache images/icons; avoid heavy sync work in build
- A11y: semantic labels on primary controls; contrast; focus traversal; tap targets ≥ 48dp; respect reduced motion
- Start-up: aim cold start ≤ 2s mid-tier

------------------------------------------------------------
12) PR / Self-review checklist
------------------------------------------------------------

- Analyzer clean; codegen updated; tests pass
- UI states covered: loading/empty/error/content
- Error surfaces present; messages mapped via DomainError
- Offline-first paths exercised; outbox reconciles
- Providers scoped narrowly; minimal rebuilds
- No unapproved dependencies; commands documented
- Feature README updated (purpose, decisions, commands, limitations)
- Diffs minimal and well-scoped

------------------------------------------------------------
13) Commands cheat sheet
------------------------------------------------------------

- Get deps: flutter pub get
- Analyze: dart analyze
- Format: dart format .
- Codegen: dart run build_runner build --delete-conflicting-outputs
- Tests: flutter test

Notes
- Never run the app in agent flows; humans can run: flutter run
- Ask before adding dependencies; reflect decisions in ADR + DecisionLog
