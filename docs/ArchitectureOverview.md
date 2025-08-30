# Architecture Overview

Modules & Dependencies
- Feature-first modules; MVVM with Riverpod Notifier/AsyncNotifier.
- Data: Dio+Retrofit (remote), Drift (local); mappers; repositories.

Error Taxonomy
- network/http, serialization, validation, cache/db, permission, unknown.

Offline Strategy
- Read-through cache; optimistic writes; reconcile on reconnect.

Navigation
- go_router with nested routes; deep links as needed.

Non-Functional Budgets
- Cold start ≤ 2.0s on mid-tier device; frame time ≤ 16ms typical; analyzer clean.

(Optional) Diagram (Mermaid)
graph TD
  UI[UI Widgets] --> VM[ViewModels (Riverpod Notifier)]
  VM --> Repo[Repository]
  Repo --> Remote[Retrofit Service (Dio)]
  Repo --> Local[Drift DAO]
  Remote -->|JSON| Mapper
  Local -->|Entities| Mapper
