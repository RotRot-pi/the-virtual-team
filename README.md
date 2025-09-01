# Context‑Engineered Virtual Team for Flutter Projects

Turn a solo workflow into a professional, multi‑role delivery system using Context Engineering (Foundation First, Module Loop, Safe Edit, Approval Gates).

This repo includes:
- Role protocols (Product, Project/Delivery, Architecture, UX, QA, DevOps, Data, Security, Docs, Flutter Engineering)
- Shared Annex (approvals, priority/risk scales, cross‑role DoD)
- Operator Guide (how to run roles in separate chats and hand off via repo files)
- Ready‑to‑use templates (PRD, Roadmap, Plan, Risks, Decisions, UX flows, Test plans, CI, Security, Analytics)

Optional: Orchestrator protocol (Cog‑Orch‑1) to coordinate multiple roles in sequence.

**Note**
This project follows the “Context Engineering” workflow from: [هندسة السياق (Context Engineering) 2 : أهم مهارة للمبرمجين في 2025 وما بعدها !]. 
Watch: [https://www.youtube.com/watch?v=PhkPIj7MDWg&t=1s].
Original repo: [https://github.com/Pythonation/Context-Engineering-for-AI-Coding].

## Why
Reduce chaos and rework by enforcing:
- Foundation First (plan before build), Module Loop (one unit at a time), Safe Edit (Read → Think → Edit), Approval gates, Facts vs Inspiration research.

## What’s inside

- protocols/
  - COG-PO.md (Product/PO)
  - COG-PM.md (Project/Delivery)
  - COG-ARCH.md (Architecture/Tech Lead)
  - COG-UX.md (UX/UI)
  - COG-QA.md (QA/Test)
  - COG-DEVOPS.md (DevOps/Release)
  - COG-DATA.md (Analytics)
  - COG-SEC.md (Security/Privacy)
  - COG-DOCS.md (Docs/Support)
  - COG-FLUTTER.md (Flutter Engineering)
  - COG-ORCH.md (Orchestrator — optional)
  - ANNEX.md (shared approval phrases, scales, cross‑role DoD)
- docs/
  - PRD.md, Roadmap.md, Backlog.md
  - Plan.md, RiskLog.md, DecisionLog.md, WeeklyStatus.md
  - ArchitectureOverview.md, CodingStandards.md, adr/
  - UX/ (flows, wireframes, components per feature)
  - TestPlan.md, RegressionChecklist.md, TestCases-<feature>.md
  - AnalyticsSpec.md
  - SecurityChecklist.md, DataMap.md, PrivacyPolicy.md
  - Support.md, FAQ.md, OperatorGuide.md
- .github/workflows/ci.yml (format/analyze/test)
- analysis_options.yaml (lints)

## Philosophy (why this works)
- Foundation First: plan before build (PRD/Roadmap → Plan/Risks → Design/Standards).
- Module Loop: one feature module at a time; explicit approval gates.
- Safe Edit: Read → Think → Edit for any modification.
- Facts vs Inspiration: correctness first; polish second.

## Quick start (how to use)
- Run each role in a separate chat/session. Load its protocol (protocols/COG-*.md) + protocols/ANNEX.md as system instructions.
- Provide a “context packet”: repo path/link, which docs to read, and what to produce now.
- Approve or request changes using Annex phrases:
  - APPROVED: <item>
  - NOT APPROVED: <reason>

## Using Gemini CLI & MCP (agents)
- Add roots first: use the add_roots tool to register your project path(s). Tools run only under these roots.
- Prefer MCP tools over raw shell for covered tasks:
  - Tests: run_tests
  - Format: dart_format
  - Analyze: analyze_files
  - Fixes: dart_fix
  - Pub: pub (get/add/remove)
- Flutter runtime: agents do not run the app. Only connect to the Dart Tooling Daemon when the operator provides a fresh DTD URI (use “Copy DTD Uri to clipboard”). Never fabricate URIs.
- GitHub: use the GitHub MCP server; call get_me to scope context; enable only needed toolsets.
- Context7: request package names/versions from pubspec.yaml to pull current docs/examples.
- Sequential Thinking: for complex tasks, use briefly (1–3 thoughts) before acting.

## Recommended kickoff order (new project)
1) Product (COG-PO) → PRD v1, Roadmap, Backlog → APPROVED  
2) Project (COG-PM) → Plan, RiskLog, DecisionLog baseline → APPROVED  
3) Architecture (COG-ARCH) → ArchitectureOverview, CodingStandards, ADRs → APPROVED  
4) UX (COG-UX) → Flow, Wireframes, Components for Module 1 → APPROVED  
5) QA (COG-QA) → TestPlan, RegressionChecklist → APPROVED  
6) DevOps (COG-DEVOPS) → CI workflow, ReleaseChecklist, CHANGELOG template → APPROVED  
7) Security (COG-SEC) → SecurityChecklist, DataMap, PrivacyPolicy draft → APPROVED  
8) Data (COG-DATA) → AnalyticsSpec → APPROVED  
9) Docs (COG-DOCS) → README/Support/FAQ updates → APPROVED  
10) Engineering (COG-FLUTTER) → Build Module 1 (Think → Act → Verify)

## Feature workflow (repeat per feature)
PO (AC) → PM (plan/risks) → ARCH (contract/ADR if needed) → UX (flow/states) → QA (test cases) → Engineering (build) → QA (verify) → Data (instrument) → Security (review) → Docs/DevOps (docs/changelog/CI green).

## Role dependency map (who reads what, who writes what)

| Role | Reads | Writes |
|---|---|---|
| Product (COG-PO) | (vision/notes) | PRD, Roadmap, Backlog, DecisionLog (product) |
| Project (COG-PM) | PRD, Roadmap, DecisionLog | Plan, RiskLog, WeeklyStatus; DecisionLog (delivery) |
| Architecture (COG-ARCH) | PRD, Roadmap | ArchitectureOverview, CodingStandards, ADRs |
| UX (COG-UX) | PRD, Roadmap | UX/<feature>/{flow, wireframes, components}.md |
| QA (COG-QA) | PRD, UX, Standards | TestPlan, RegressionChecklist, TestCases-<feature>.md |
| DevOps (COG-DEVOPS) | Plan, DecisionLog | CI workflow, ReleaseChecklist, CHANGELOG template |
| Data (COG-DATA) | PRD, UX | AnalyticsSpec, WeeklyInsights (opt.) |
| Security (COG-SEC) | PRD, DecisionLog | SecurityChecklist, DataMap, PrivacyPolicy draft |
| Docs (COG-DOCS) | PRD + outputs | README/Support/FAQ updates, feature docs |
| Flutter Eng (COG-FLUTTER) | Roadmap, UX, QA, Standards, Plan, DecisionLog | Code, per‑feature notes (DoD checklist) |

## Feature docs and API contracts
- Feature docs = PRD (requirements/AC) + UX (flows/states) + QA (test cases) + (if needed) ARCH’s contract notes. This is enough for backend/frontend/mobile to start.
- API docs (if backend is involved): ARCH defines the service contract inside ArchitectureOverview or an ADR. If you prefer OpenAPI, ask ARCH to author it; DOCS links it. Add a “Contract Freeze” approval before Engineering starts.

## Local development (human)
- Get dependencies: `flutter pub get`
- Codegen (if used): `dart run build_runner build --delete-conflicting-outputs`
- Lint/analyze: `dart analyze`
- Tests: `flutter test`
- Run: `flutter run` (humans only; agents will not run the app)

## CI
- PRs must pass format, analyze, and tests. See .github/workflows/ci.yml

## Optional Orchestrator
- protocols/COG-ORCH.md can act as a “control tower” to sequence roles and emit the exact prompts/handoffs. Optional; see docs/OperatorGuide.md.

## License
MIT (add a LICENSE file if you haven’t yet)
