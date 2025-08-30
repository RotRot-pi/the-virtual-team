# Coding Standards

Language & Lints
- Dart (latest stable), Flutter (stable). Use flutter_lints. No print; prefer logger. Prefer const.

Structure
- Feature-first under lib/features/<feature> with presentation/state/data subfolders.
- Core under lib/core (router, theme, errors, utils).

State & DI
- Riverpod Notifier/AsyncNotifier. Keep providers scoped; avoid global state leaks.

Networking
- Dio with interceptors (timeouts, auth, logging). Retrofit for typed clients.

Data
- Drift for offline; platform-aware init (mobile vs web IndexedDB). Mappers for DTO↔domain.

Models
- Freezed + json_serializable for DTOs and unions; avoid manual boilerplate.

Error Handling
- Map errors to domain-safe types; user-friendly messages; surface retry where appropriate.

Testing
- Unit tests for mapping/repo/viewmodel critical paths. Analyzer must be clean pre-commit.
