# Architecture Playbook

Status: Reference
Version: v1.0
Owner: Cog-ARCH-1
Date: 2025-08-30
Related: PRD (docs/PRD.md) • Roadmap (docs/Roadmap.md) • ArchitectureOverview (docs/ArchitectureOverview.md) • CodingStandards (docs/CodingStandards.md) • ADRs (docs/adr/)

Purpose
- Guardrails, not rails. This playbook reduces ambiguity and improves consistency across modules without limiting smart deviations.
- If you deviate, record why in an ADR (context → options → decision → consequences).

Mode toggle (choose per engagement)
- Minimal: default. Use checklists and examples here; only create ADRs when triggers hit.
- Extended: if the operator asks for deeper rigor, include more detail (benchmarks, alternatives, proofs).

----------------------------------------------------------------
1) What “good” looks like (ArchitectureOverview checklist)
----------------------------------------------------------------
Include, briefly:
- Modules & dependencies: feature-first boundaries; no cyclic deps.
- Data flow (high level): UI → VM → Repo → Sources(HTTP/DB), with cache/read-through.
- Error taxonomy & mapping: kinds → domain codes → UI guidance (see Section 3).
- Offline strategy: cache/read-through + outbox policy (see Section 4).
- Navigation contract: route naming, nesting, guards, deep links (Section 5).
- Quality budgets: cold start target, frame time, error budgets, network timeouts (Section 7).
- Codegen policy: Freezed/Retrofit/Drift/Riverpod codegen; never edit generated files.
- Platform notes: mobile/web deltas (Drift WASM, CORS).

Keep it tight; link detailed choices to ADRs when needed.

----------------------------------------------------------------
2) ADR quick reference (when and how)
----------------------------------------------------------------
Write an ADR when any of these triggers occur:
- New dependency or significant version bump
- DB schema/persistence strategy change
- Cross-cutting error policy/mapping change
- Public API/service contract change or navigation structure rewrite
- Notable performance/reliability/security tradeoff

ADR template (short):
```
# ADR-<id>: <title>
Status: Proposed | Accepted | Superseded
Date: YYYY-MM-DD

Context
- What problem are we solving? Constraints? Who/what is affected?

Options
- A) ...
- B) ...
- (cite facts/links when relevant)

Decision
- Chosen option and why (tradeoffs).

Consequences
- Positive/negative effects; roll-out/migration notes; test/monitoring implications.

Links
- PRD/Roadmap sections; related ADRs; relevant code/docs.
```

----------------------------------------------------------------
3) Error Mapping contract (taxonomy → domain codes → UI)
----------------------------------------------------------------
Goal: predictable errors and messages across layers.

- Taxonomy (minimum): network, http(code), serialization, validation, cache/db, permission, unknown.
- Domain codes: ER-<KIND>-<SUBCODE> (e.g., ER-HTTP-401, ER-NET-TO)
- UI guidance mapping (examples):
  - network → “Check your connection and retry.” (Retry CTA)
  - http:401 → “Session expired. Please sign in.” (Reauth)
  - validation → Show field-level errors; don’t retry
  - serialization → “We’re fixing an issue. Try later.” (Logs only)
  - cache/db → “Something went wrong. Try again.” (Retry CTA)

Deliverables:
- Put a small table in ArchitectureOverview with canonical mappings and copy guidelines.
- Engineering maps Dio/DB exceptions to domain codes early (interceptors/helpers).
- QA uses codes in test cases; PO aligns user-facing copy.

----------------------------------------------------------------
4) Offline-first & reconciliation (outbox policy)
----------------------------------------------------------------
Default policy (simple, robust):
- Reads: cache-first with background refresh (read-through).
- Writes: optimistic UI + outbox queue (Drift table) with exponential backoff.
- Reconciliation: server-wins by default; refresh local after success.
- Conflicts: if local pending edits exist, prefer server-wins unless product requires merge/prompt (record exceptions via ADR).

Outbox schema (minimal fields):
- id, operation, entity_type, payload_json, attempt_count, status(pending|retry|failed|done), last_error, created_at, updated_at

Processing triggers:
- App resume, connectivity regained, periodic timer. Cap attempts (e.g., 5) with jitter.

----------------------------------------------------------------
5) Navigation contract (go_router)
----------------------------------------------------------------
- Naming: FeatureRoute enum per module; name-based navigation preferred.
- Structure: feature routes live in feature/presentation/routing.dart; app-level router composes feature routes.
- Guards: pure functions; read auth/state via providers; keep guards small/testable.
- Deep links: stable paths (/todos/:id). Document params and types.
- Redirect patterns: centralize in router config; avoid side effects in builders.

----------------------------------------------------------------
6) Data & persistence (Drift, DTOs, Repos)
----------------------------------------------------------------
- DTOs: Freezed, explicit fromJson; map DTO ↔ domain explicitly; test roundtrips.
- Drift schema: primary keys set; indices for frequent queries; migrations described (bump schema, add companions).
- Repos: single place to orchestrate sources; keep mappers pure; no UI logic.
- Web: Drift WASM on web or memory fallback for demos; document choice.
- Performance: batch DB writes; use insertAllOnConflictUpdate; avoid chatty queries.

----------------------------------------------------------------
7) Networking & HTTP (Dio/Retrofit)
----------------------------------------------------------------
- Timeouts: connect ≤ 5s; receive ≤ 10s (tune per API).
- Interceptors: auth header, error translation (to domain codes), logging (dev-only).
- Retries: max 2 for idempotent GETs with jitter; no blind retries for writes.
- Cancellation: pass CancelToken from screen-level scope.
- Config: inject baseUrl/env via providers; don’t hardcode secrets.

----------------------------------------------------------------
8) Quality budgets (targets; adjust per project)
----------------------------------------------------------------
- Cold start: ≤ 2.0s on mid-tier device
- Frame time: 60fps typical; 95th percentile ≤ 16ms
- Crash-free sessions: ≥ 99.5% (when telemetry exists)
- Network: P95 GET ≤ 800ms on LTE (API-dependent)
- Error budget: visible error rate ≤ 1% on primary flows

Document current targets in ArchitectureOverview and revisit when telemetry is live.

----------------------------------------------------------------
9) Security & privacy (coordinate with Sec/Data)
----------------------------------------------------------------
- Secrets: never in repo; inject via env/CI; rotate regularly.
- Permissions: least privilege; document rationale; link to store disclosures.
- Data classes: classify PII vs operational data in DataMap; align Analytics with PrivacyPolicy.

----------------------------------------------------------------
10) Platform specifics (web/desktop)
----------------------------------------------------------------
- Drift web: sqlite3/wasm; ensure bundling; consider memory fallback during early dev.
- CORS: server must allow web origins; avoid assuming mobile-only headers.
- Plugins: guard platform-only APIs; conditional imports.

----------------------------------------------------------------
11) Diagrams (optional but helpful)
----------------------------------------------------------------
Mermaid example (trim as needed):
```mermaid
flowchart LR
  UI[Presentation] --> VM[ViewModel (Riverpod)]
  VM --> Repo[Repository]
  Repo -->|read/write| DB[(Drift)]
  Repo -->|HTTP| API[(Dio/Retrofit)]
  VM --> Err[Error Mapping]
  subgraph Offline
    Outbox[Outbox Queue]
  end
  Repo <-->|sync| Outbox
```

----------------------------------------------------------------
12) Deviations & evolution
----------------------------------------------------------------
- Deviate freely when it’s the right call—just write a short ADR capturing the why.
- Keep the ArchitectureOverview tight; push deep details into ADRs.
- Prefer evolutionary design: per-feature adjustments over big-bang rewrites.

