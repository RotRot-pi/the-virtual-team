# OperatorGuide — Running Your Virtual Pro Team (Context Engineering)

This guide shows you how to run each professional role in separate chats, how they continue each other’s work via repo files, how to use the Annex, and the exact flow to build and ship features reliably.

- Protocols live in: protocols/
- Shared Annex: protocols/ANNEX.md
- Role outputs live in: docs/ (plus code in your repo)
- Approval phrases: use Annex (“APPROVED: …” / “NOT APPROVED: …”)

Contents
1) How multi-role chats work
2) Role dependency map (who reads what, who writes what)
3) The Annex (gate, loop mode, identity lock)
4) Kickoff flow (new project)
5) Feature cycle flow (repeat per feature)
6) Prompt templates (copy/paste)
7) Walk-through example (condensed)
8) Handoff card (paste between roles)
9) Scenarios playbook
10) Pitfalls to avoid
11) FAQ
12) Glossary
13) Document status header (mini-template)
14) Optional: ApprovalsLog.md

---

## 1) How multi-role chats work

- Each role runs in its own chat/session. They don’t rely on chat memory.
- They coordinate by reading/writing shared artifacts in your repo (docs/ and protocols/).
- You are the approver. Use the Annex handshake to approve or request changes.

Identity Lock (every role opens with this)
- At the start of a run, roles must restate:
  1) Identity & Mission
  2) Allowed outputs (docs they will write)
  3) Forbidden actions (what they will not do)
  4) Read boundaries (what they will read)
  5) Current Phase/Step and where they will stop for approval

Loop Mode handshake (before any Phase 2 “Module Loop”)
- Ask: “Loop mode? [run-until-done | soft-cap | stop-per-module]”
- If soft-cap: ask “cap = <N>”
- Confirm: “LOOP MODE: <mode> [cap=N]”
- Default if not specified: stop-per-module (safer)
- Note: FLUTTER engineering enforces Single WIP (one module per run); Loop Mode batching does not apply to FLUTTER.

MCP roots and execution discipline
- Project roots (for MCP add_roots): absolute path(s) agents/tools are allowed to operate under.
- If Flutter/Dart MCP is used: provide a fresh DTD URI (use “Copy DTD Uri to clipboard”). Agents do not fabricate URIs and do not run the app.

---

## 2) Role dependency map (who reads what, who writes what)

| Role | Reads (Inputs) | Writes (Outputs) |
|---|---|---|
| Product (COG-PO) | (vision/notes) | docs/PRD.md, docs/Roadmap.md, docs/Backlog.md, docs/DecisionLog.md (product) |
| Project (COG-PM) | PRD, Roadmap, DecisionLog | docs/Plan.md, docs/RiskLog.md, docs/WeeklyStatus.md; DecisionLog (delivery) |
| Architecture (COG-ARCH) | PRD, Roadmap | docs/ArchitectureOverview.md, docs/CodingStandards.md, docs/adr/ADR-*.md |
| UX (COG-UX) | PRD, Roadmap | docs/UX/<module>/{flow.md, wireframes.md, components.md} |
| QA (COG-QA) | PRD, UX, Standards | docs/TestPlan.md, docs/RegressionChecklist.md, docs/TestCases-<module>.md, docs/Bugs-<module>.md (default) |
| DevOps (COG-DEVOPS) | Plan, DecisionLog | .github/workflows/ci.yml, docs/ReleaseChecklist.md, CHANGELOG.md template (DevOps owns CHANGELOG) |
| Data (COG-DATA) | PRD, UX, Security docs | docs/AnalyticsSpec.md, docs/WeeklyInsights.md (opt.) |
| Security (COG-SEC) | PRD, DecisionLog | docs/SecurityChecklist.md, docs/DataMap.md, docs/PrivacyPolicy.md |
| Docs (COG-DOCS) | PRD + outputs | docs/README-skeleton.md, docs/Support.md, docs/FAQ.md, docs/Feature-<name>.md (links to DevOps’ CHANGELOG) |
| Flutter Eng (COG-FLUTTER) | Roadmap, eng_briefs/, UX, QA, Standards, Plan, DecisionLog | Code, feature notes (DoD checklist), tests |

Module = Feature Module: a vertical slice across UX, ViewModel, Repo/API/DB, tests, and docs.

---

## 3) The Annex (gate, loop mode, identity lock)

Gate (Baseline mode)
- Draft/update docs/ without prior approval is allowed.
- No source code or external state changes (pub/git/GitHub/workflows) until APPROVED.
- Present plan + diffs + commands first for any code/external changes.

Approval handshake
- APPROVED: <doc/step> — proceed to next step per docs/HandoffQueue.md
- NOT APPROVED: <reason> — revise and resubmit; do not proceed
- HANDOFF OPEN: <modules/list> — authorizes PM to create issues from ENG Brief(s)

Loop Mode (before any Module Loop)
- Ask and confirm one of:
  - run-until-done — proceed through all planned modules for this role (pause early only for high-risk decisions/ADRs or missing info)
  - soft-cap — proceed up to cap=N modules this run
  - stop-per-module — stop after each module
- Default: stop-per-module
- Note: FLUTTER = Single WIP (no batching)

Identity Lock (applies to all roles)
- Restate identity, allowed outputs, forbidden actions, read boundaries, current phase/step, and stop point at the start of a run.

Document status header (recommended)
- At the top of every produced doc: Status: Draft | In Review | Approved; Version; Owner; Date; Related: links to PRD/Roadmap/ADRs.

---

## 4) Kickoff flow (new project)

1) Product (COG-PO) → PRD v1, Roadmap, Backlog → APPROVED  
2) Project (COG-PM) → Plan, RiskLog, DecisionLog baseline → APPROVED  
3) Architecture (COG-ARCH) → ArchitectureOverview, CodingStandards, ADRs (as needed) → APPROVED  
4) UX (COG-UX) → Flow, Wireframes, Components for Module 1 → APPROVED  
5) QA (COG-QA) → TestPlan, RegressionChecklist → APPROVED  
6) DevOps (COG-DEVOPS) → CI workflow, ReleaseChecklist, CHANGELOG template (DevOps-owned) → APPROVED  
7) Security (COG-SEC) → SecurityChecklist, DataMap, PrivacyPolicy draft → APPROVED  
8) Data (COG-DATA) → AnalyticsSpec (links to Security’s DataMap/PrivacyPolicy) → APPROVED  
9) Docs (COG-DOCS) → README-skeleton, Support, FAQ → APPROVED  
10) Engineering (COG-FLUTTER) → Build Module 1 (Think → Act → Verify)

Notes
- Foundation/Planning steps may be batched using Loop Mode run-until-done or soft-cap if you prefer. Sequence approvals to avoid conflicts.
- Implementation/verification (UX→QA→ENG→QA→DATA→SEC→DOCS/DEVOPS) honors Single WIP.

---

## 5) Feature cycle flow (repeat per feature/module)

1) PO: refine AC → APPROVED  
2) PM: slot into Plan, note risks/mitigations → APPROVED  
3) ARCH: ADR only if needed (triggers) → APPROVED  
4) UX: flow/wireframes/states → APPROVED  
5) QA: TestCases-<module>.md (happy, error, offline) → APPROVED  
6) Engineering (FLUTTER): build with Cog-Flutter-1 (ask before new deps) → Verify  
7) QA: verify against AC; record bugs in docs/Bugs-<module>.md (or issues if allowed) → APPROVED  
8) Data: confirm events/validation plan → APPROVED  
9) Security: confirm permissions/data/risks → APPROVED  
10) Docs/DevOps: update docs and CHANGELOG; ensure CI green → ship or queue

---

## 6) Prompt templates (copy/paste)

General pattern
- System: load the role protocol + ANNEX.
- User: provide context packet + request + approval gates.
- Let the role ask Loop Mode when entering Phase 2, or you can specify it up front.

Role Guard wrapper (paste at the top of any role chat)
- You are EXACTLY <Role> per protocols/COG-<ROLE>.md + protocols/ANNEX.md.  
  Before acting, restate: identity, allowed outputs, forbidden actions, read boundaries, current Phase/Step, and where you will stop for approval.  
  Respect Gate (Baseline): docs OK; no code/external ops without approval.  
  If entering Phase 2, ask: “Loop mode? [run-until-done | soft-cap | stop-per-module]”.

Product (COG-PO)
System: Use protocols/COG-PO.md and protocols/ANNEX.md  
User: Repo attached. Run Phase 1 for <ProjectName>. If PRD gaps exist, ask me. Produce PRD v1, Roadmap, Backlog with Facts vs Inspiration (links). Stop for approval. If continuing to Phase 2, ask Loop Mode first.

Project/Delivery (COG-PM)
System: Use protocols/COG-PM.md and protocols/ANNEX.md  
User: Inputs: docs/PRD.md, docs/Roadmap.md. Run Phase 1. Produce Plan.md, RiskLog.md, DecisionLog.md baseline, WeeklyStatus.md template. Stop for approval.

Architecture (COG-ARCH)
System: Use protocols/COG-ARCH.md and protocols/ANNEX.md  
User: Inputs: PRD.md, Roadmap.md. Draft ArchitectureOverview.md, CodingStandards.md, and ADRs only if triggers apply. Define Error Mapping + Offline Reconciliation stubs. No code or dependency changes. Stop for approval. Ask Loop Mode before Phase 2.

UX (COG-UX)
System: Use protocols/COG-UX.md and protocols/ANNEX.md  
User: Module: <Name>. Inputs: PRD.md, Roadmap.md. Produce UX/<module>/flow.md, wireframes.md, components.md with states. Stop for approval. Ask Loop Mode before Phase 2.

QA (COG-QA)
System: Use protocols/COG-QA.md and protocols/ANNEX.md  
User: Run Phase 1 (TestPlan, RegressionChecklist). Then for Module <name>, produce TestCases-<module>.md. By default log bugs in docs/Bugs-<module>.md; ask me before filing GitHub issues. Stop for approval.

DevOps (COG-DEVOPS)
System: Use protocols/COG-DEVOPS.md and protocols/ANNEX.md  
User: Run Phase 1. Propose .github/workflows/ci.yml, ReleaseChecklist.md, and CHANGELOG template (DevOps-owned). Stop for approval.

Data (COG-DATA)
System: Use protocols/COG-DATA.md and protocols/ANNEX.md  
User: Run Phase 1. Produce AnalyticsSpec.md (KPIs, events, params, validation) and link to Security’s DataMap/PrivacyPolicy. Stop for approval.

Security (COG-SEC)
System: Use protocols/COG-SEC.md and protocols/ANNEX.md  
User: Run Phase 1. Produce SecurityChecklist.md, DataMap.md, PrivacyPolicy draft with Facts citations. Stop for approval.

Docs (COG-DOCS)
System: Use protocols/COG-DOCS.md and protocols/ANNEX.md  
User: Run Phase 1. Produce README-skeleton.md, Support.md, FAQ.md. Link to DevOps’ CHANGELOG.md. Stop for approval.

Engineering (COG-FLUTTER)
System: Use protocols/COG-FLUTTER.md and protocols/ANNEX.md  
User: Module: <Name>. Inputs: docs/eng_briefs/<module>.md, UX/<module>/*, TestCases-<module>.md, CodingStandards.md. Start Phase 2. Ask before adding dependencies. Output minimal diffs + commands. Stop for verification. Do not run the app.

---

## 7) Walk-through example (condensed)

- You → PO: “Run Phase 1 for ‘TaskFlow’.”  
  PO asks questions → writes PRD, Roadmap, Backlog with citations →  
  You: APPROVED: PRD v1 and Roadmap.

- You → PM: “Run Phase 1 with PRD/Roadmap.”  
  PM writes Plan, RiskLog, DecisionLog baseline, WeeklyStatus →  
  You: APPROVED: Plan baseline.

- You → ARCH: “Draft ArchitectureOverview, CodingStandards; ADRs only if triggers.”  
  ARCH delivers with Error Mapping + Offline stubs →  
  You: APPROVED.

- You → UX (Module: Auth): UX flow/wireframes/components →  
  You: NOT APPROVED: missing error/empty states → UX revises → APPROVED.

- You → QA: TestPlan + RegressionChecklist → APPROVED.  
  QA → TestCases-Auth.md → APPROVED.

- You → DevOps: CI + ReleaseChecklist + CHANGELOG template → APPROVED (DevOps owns CHANGELOG).  
  You → Security/Data/Docs: baselines → APPROVED.

- You → Engineering: Build Auth module → Verify → QA checks → Data/Sec/Docs/DevOps finalize → Ship.

---

## 8) Handoff card (paste between roles)

```
Project: <name>
Scenario: <kickoff/feature/refactor/hotfix/release>
Step: <N of M> — <Role>
Module (if any): <module name>
Loop Mode: <run-until-done | soft-cap (cap=N) | stop-per-module>
Inputs to read:
- docs/PRD.md
- docs/Roadmap.md
- Role-specific docs (e.g., docs/UX/<module>/*, docs/TestCases-<module>.md)
- If Role = FLUTTER: docs/eng_briefs/<module>.md, docs/CodingStandards.md
Outputs you must produce:
- <per protocol>
Annex: protocols/ANNEX.md
MCP roots: <absolute paths added via add_roots>
DTD URI (if Flutter): <paste from “Copy DTD Uri to clipboard”>
Approval gate: Stop and ask before proceeding.
```

---

## 9) Scenarios playbook

- New app from scratch  
  Run full kickoff order (PO → PM → ARCH → UX → QA → DEVOPS → SEC → DATA → DOCS → ENGINEERING). Use Loop Mode run-until-done for planning if you want speed.

- Add “Favorites” feature  
  PO (AC) → PM (plan) → ARCH (skip unless new dep/contract) → UX (flow/states) → QA (test cases) → ENG build → QA verify → DATA events → SEC review → DOCS/DEVOPS.

- Mid-sprint change request  
  PO updates PRD/Backlog → PM updates Plan/Risks → ARCH/UX/QA re-check scope → ENG adjusts. Record in DecisionLog.

- External API/Dependency change  
  ARCH ADR (mitigation) → PM risk update → QA regression case → ENG patch → DOCS changelog → DATA/SEC if data/permissions changed.

- Performance regression  
  ARCH sets budget/tuning ADR → ENG instruments/fixes → QA adds perf checks → DEVOPS ensures CI passes → DOCS notes limits.

- Store rejection  
  PO/SEC research reason (Facts) → PRD/PrivacyPolicy/UX update → ENG fix → DEVOPS resubmit via ReleaseChecklist.

- Security vulnerability in dependencies  
  SEC logs risk/mitigation → PM schedules hotfix → ARCH confirms safe version → ENG bumps → QA regression → DEVOPS release.

- Hotfix release  
  QA repro + AC → ENG minimal fix → QA regression → DEVOPS tagged release → DOCS changelog → PM updates plan.

- Failing CI  
  DEVOPS triage → ENG fix lints/tests → PM notes plan impact → QA regression → DOCS note fix.

---

## 10) Pitfalls to avoid

- Starting coding without AC → Always get PO/UX/QA approvals first.
- Identity leakage → Enforce Identity Lock; roles do only their job.
- Skipping Loop Mode → Always ask/confirm before Phase 2 loops.
- Gate misuse → Docs OK pre-approval; no code/external ops until APPROVED.
- Dependency sprawl → Engineering must ask; ARCH/PM log decision/tradeoffs (ADR + DecisionLog).
- Ambiguous approvals → Use exact Annex phrases.
- Long chats, no artifacts → Always write/update files; files are the contract.

---

## 11) FAQ

- Can I run roles in parallel?  
  Yes for planning/foundation using Loop Mode (run-until-done or soft-cap). Implementation/verification stays Single WIP.

- Do I need to paste protocols every time?  
  Prefer loading files (protocols/*.md + ANNEX.md) as system instructions. If your tool can’t load files, paste once per chat.

- How do they “continue” each other in separate chats?  
  Through docs in the repo. Each role reads inputs and writes outputs; the next role picks them up.

- What’s the difference between Baseline vs Strict Gate?  
  This guide uses Baseline: docs allowed pre-approval; code/external ops need approval. Strict is an optional variant where roles propose diffs before writing any docs.

- Who owns CHANGELOG?  
  DevOps. Docs links to it in README/Support/FAQ and focuses on user-facing guides.

- How does QA file bugs?  
  Default to docs/Bugs-<module>.md. If you want GitHub issues, say “file issues” and QA will use MCP GitHub tools.

- Does FLUTTER use Loop Mode?  
  No. FLUTTER enforces Single WIP: one module per run, per HANDOFF OPEN.

---

## 12) Glossary

- PRD: Product Requirements Document (what/why/for whom, with acceptance criteria).
- ADR: Architecture Decision Record (context, options, decision, consequences).
- AC: Acceptance Criteria (testable).
- DoD: Definition of Done (what must be true to call it done).
- WIP: Work in Progress (limit to keep focus).
- RAG: Red/Amber/Green project health status.
- Identity Lock: The role’s self-check at start (identity, outputs, forbidden actions, read boundaries, stop point).
- Loop Mode: How a role proceeds through modules in Phase 2 (run-until-done, soft-cap, stop-per-module).
- HANDOFF OPEN: Authorization from PO to PM to create issues from ENG Brief(s).

---

## 13) Document status header (mini-template)

Add this to the top of every produced doc:
```
Status: Draft | In Review | Approved
Version: v0.1
Owner: <role/person>
Date: YYYY-MM-DD
Related: [PRD](docs/PRD.md) • [Roadmap](docs/Roadmap.md) • [ADR-001](docs/adr/ADR-001.md)
```

---

## 14) Optional: ApprovalsLog.md

Keep a lightweight trail of approvals and decisions.
```
| Date/Time | Item | Decision | By | Link/Notes |
|---|---|---|---|---|
| 2025-01-10 14:02 | PRD v1 | APPROVED | You | docs/PRD.md |
| 2025-01-10 16:18 | ArchitectureOverview | APPROVED | You | docs/ArchitectureOverview.md |
| 2025-01-11 10:05 | UX/Auth | NOT APPROVED | You | Missing error states; revised v1.1 |
```

---

How to start today

1) Ensure protocols/ and docs/ exist.
2) Open a new chat with Product (COG-PO) and Annex; run Phase 1.
3) Approve or request changes using Annex phrases.
4) Use Loop Mode for batching in planning phases; keep Single WIP for build/verify.
5) Proceed down the kickoff order. For each feature, follow the feature cycle flow (PO → PM → ARCH → UX → QA → ENG → QA → DATA → SEC → DOCS/DEVOPS).
