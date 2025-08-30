# OperatorGuide — Running Your Virtual Pro Team (Context Engineering)

This guide shows you how to run each professional role in separate chats, how they continue each other’s work via repo files, how to use the Annex, and the exact flow to build and ship features reliably.

- Protocols live in: protocols/
- Shared Annex: protocols/ANNEX.md
- Role outputs live in: docs/ (plus code in your repo)

Contents
1) How multi-role chats work
2) Role dependency map (who reads what, who writes what)
3) The Annex (what/why/how/who)
4) Kickoff flow (new project)
5) Feature cycle flow (repeat per feature)
6) Prompt templates (copy/paste)
7) Walk-through example
8) Handoff card (paste between roles)
9) Scenarios playbook
10) Pitfalls to avoid
11) FAQ
12) Glossary

---

## 1) How multi-role chats work

- Each role runs in its own chat/session. They don’t rely on chat memory.
- They coordinate by reading/writing shared artifacts in your repo (docs/ and protocols/).
- You are the approver. Use the Annex handshake to approve or request changes.

- Project roots (for MCP add_roots): absolute path(s) agents/tools are allowed to operate under.
- If Flutter/Dart MCP is used: provide a fresh DTD URI (use “Copy DTD Uri to clipboard”). Do not fabricate URIs.

“Context packet” to start any role:
- Repo: attach files or link (GitHub/local).
- Key docs (paths):
  - PRD: docs/PRD.md
  - Roadmap: docs/Roadmap.md
  - Plan: docs/Plan.md
  - RiskLog: docs/RiskLog.md
  - DecisionLog: docs/DecisionLog.md
  - ArchitectureOverview: docs/ArchitectureOverview.md
  - CodingStandards: docs/CodingStandards.md
  - QA: docs/TestPlan.md, docs/RegressionChecklist.md
  - DevOps CI: .github/workflows/ci.yml
  - AnalyticsSpec: docs/AnalyticsSpec.md
  - Security: docs/SecurityChecklist.md, docs/DataMap.md, docs/PrivacyPolicy.md
  - Docs: README or docs/README-skeleton.md, docs/Support.md, docs/FAQ.md
- Protocol(s) to load (as system prompt/instructions): e.g., protocols/COG-PO.md + protocols/ANNEX.md
- Clear request: “Run Phase 1…” or “Proceed to Phase 2 for Module X…”

---

## 2) Role dependency map

| Role | Reads (Inputs) | Writes (Outputs) |
|---|---|---|
| Product (Cog-PO-1) | Any vision/notes (optional) | docs/PRD.md, docs/Roadmap.md, docs/Backlog.md, docs/DecisionLog.md (product) |
| Project/Delivery (Cog-PM-1) | PRD, Roadmap, DecisionLog | docs/Plan.md, docs/RiskLog.md, docs/WeeklyStatus.md, updates DecisionLog (delivery) |
| Architecture (Cog-ARCH-1) | PRD, Roadmap, (Plan), DecisionLog | docs/ArchitectureOverview.md, docs/CodingStandards.md, docs/adr/ADR-*.md, updates DecisionLog (technical) |
| UX/UI (Cog-UX-1) | PRD, Roadmap, (CodingStandards UI notes) | docs/UX/<module>/{flow.md, wireframes.md, components.md} |
| QA/Test (Cog-QA-1) | PRD, UX docs (for module), CodingStandards | docs/TestPlan.md, docs/RegressionChecklist.md, docs/TestCases-<module>.md, bug issues |
| DevOps/Release (Cog-DevOps-1) | Plan, DecisionLog | .github/workflows/ci.yml, docs/ReleaseChecklist.md, CHANGELOG.md template |
| Data/Analytics (Cog-Data-1) | PRD (success metrics), UX flows | docs/AnalyticsSpec.md, docs/WeeklyInsights.md (optional) |
| Security/Privacy (Cog-Sec-1) | PRD (data/permissions), DecisionLog | docs/SecurityChecklist.md, docs/DataMap.md, docs/PrivacyPolicy.md |
| Docs/Support (Cog-Docs-1) | PRD + outputs from other roles | docs/README-skeleton.md (or README updates), docs/Support.md, docs/FAQ.md |
| Engineering/Flutter (Cog-Flutter-1) | Roadmap, UX module docs, TestCases-<module>.md, CodingStandards, Plan, DecisionLog | Code, feature docs/notes, updates DecisionLog for impl tradeoffs |

---

## 3) The Annex (protocols/ANNEX.md)

What it is:
- A tiny, shared rulebook so everyone uses the same approval phrases, priority/risk scales, bug severity, and cross-role Definition of Done.

Why it matters:
- Without it, roles invent their own terms (confusion). With it, all roles align on language and gates.

Who uses it:
- You and every role. Always load protocols/ANNEX.md alongside a role protocol.

How to use:
- Approve or reject with the exact phrases:
  - APPROVED: <module/name or doc>
  - NOT APPROVED: <reason — what’s missing/needs change>
- Priority scale: P0..P3
- Risk matrix: Likelihood L1..L3 x Impact I1..I3, Score = LxIx
- Bug severity: S1..S4
- Cross-role DoD: docs updated, analyzer/lints clean, tests pass for critical paths, a11y/perf sanity, release impact noted.

---

## 4) Kickoff flow (new project)

1) Product (Cog-PO-1) → PRD v1 + Roadmap + Backlog → APPROVED
2) Project (Cog-PM-1) → Plan + RiskLog + DecisionLog baseline → APPROVED
3) Architecture (Cog-ARCH-1) → ArchitectureOverview + CodingStandards + first ADRs → APPROVED
4) UX (Cog-UX-1) → Flow + Wireframes + Components for Module 1 → APPROVED
5) QA (Cog-QA-1) → TestPlan + RegressionChecklist → APPROVED
6) DevOps (Cog-DevOps-1) → CI + ReleaseChecklist + CHANGELOG template → APPROVED
7) Security (Cog-Sec-1) → SecurityChecklist + DataMap + PrivacyPolicy draft → APPROVED
8) Data (Cog-Data-1) → AnalyticsSpec → APPROVED
9) Docs (Cog-Docs-1) → README skeleton + Support/FAQ → APPROVED
10) Engineering (Cog-Flutter-1) → Build Module 1 (Think → Act → Verify)

Tip: You can run some steps in parallel, but keep approvals in order (e.g., UX after PRD, PM after Roadmap).

---

## 5) Feature cycle flow (repeat per feature/module)

1) PO: refine AC for the feature → APPROVED
2) PM: slot into Plan, note risks/mitigations → APPROVED
3) ARCH: ADR only if needed (new dep, DB change, etc.) → APPROVED
4) UX: flow/wireframes/states for this feature → APPROVED
5) QA: TestCases-<module>.md (happy, error, offline) → APPROVED
6) Engineering: build with Cog-Flutter-1 (ask before adding deps) → Verify
7) QA: verify against AC; log issues → APPROVED
8) Data: confirm events/validation plan → APPROVED
9) Security: confirm permissions/data/risks → APPROVED
10) Docs/DevOps: update CHANGELOG/README; ensure CI green → ship or queue

---

## 6) Prompt templates (copy/paste)

General pattern:
- System: load the role protocol + ANNEX.
- User: provide context packet + request + approval gates.

Product (COG-PO)
```
System: Use protocols/COG-PO.md and protocols/ANNEX.md
User: Repo attached. Run Phase 1 for <ProjectName>. If PRD gaps exist, ask me. Produce PRD v1, Roadmap, Backlog with Facts vs Inspiration (links). Stop for approval.
```

Project/Delivery (COG-PM)
```
System: Use protocols/COG-PM.md and protocols/ANNEX.md
User: Inputs: docs/PRD.md, docs/Roadmap.md. Run Phase 1. Produce Plan.md, RiskLog.md, DecisionLog.md baseline, WeeklyStatus.md template. Stop for approval.
```

Architecture (COG-ARCH)
```
System: Use protocols/COG-ARCH.md and protocols/ANNEX.md
User: Inputs: PRD.md, Roadmap.md. Draft ArchitectureOverview.md, CodingStandards.md, and any needed ADRs. Stop for approval.
```

UX (COG-UX)
```
System: Use protocols/COG-UX.md and protocols/ANNEX.md
User: Module: <Name>. Inputs: PRD.md, Roadmap.md. Produce UX/<module>/flow.md, wireframes.md, components.md with states. Stop for approval.
```

QA (COG-QA)
```
System: Use protocols/COG-QA.md and protocols/ANNEX.md
User: Run Phase 1 and then derive TestCases for <module> from PRD and UX docs. Stop for approval.
```

DevOps (COG-DEVOPS)
```
System: Use protocols/COG-DEVOPS.md and protocols/ANNEX.md
User: Run Phase 1. Propose .github/workflows/ci.yml, ReleaseChecklist.md, and CHANGELOG template. Stop for approval.
```

Data (COG-DATA)
```
System: Use protocols/COG-DATA.md and protocols/ANNEX.md
User: Run Phase 1. Produce AnalyticsSpec.md (KPIs, events, params, validation). Stop for approval.
```

Security (COG-SEC)
```
System: Use protocols/COG-SEC.md and protocols/ANNEX.md
User: Run Phase 1. Produce SecurityChecklist.md, DataMap.md, PrivacyPolicy draft with Facts citations. Stop for approval.
```

Docs (COG-DOCS)
```
System: Use protocols/COG-DOCS.md and protocols/ANNEX.md
User: Run Phase 1. Produce README-skeleton.md, Support.md, FAQ.md. Stop for approval.
```

Engineering (COG-FLUTTER)
```
System: Use protocols/COG-FLUTTER.md and protocols/ANNEX.md
User: Module: <Name>. Inputs: UX/<module>/*, TestCases-<module>.md, CodingStandards.md. Start Phase 2. Ask before adding dependencies. Output minimal diffs + commands. Stop for verification.
```

---

## 7) Walk-through example (condensed)

- You → PO: “Run Phase 1 for ‘TaskFlow’.”  
  PO asks questions → writes PRD, Roadmap, Backlog with citations →  
  You: APPROVED: PRD v1 and Roadmap.

- You → PM: “Run Phase 1 with PRD/Roadmap.”  
  PM writes Plan, RiskLog, DecisionLog baseline, WeeklyStatus →  
  You: APPROVED: Plan baseline.

- You → ARCH: “Draft ArchitectureOverview, CodingStandards, ADR for go_router & Riverpod Notifier.”  
  ARCH delivers → You: APPROVED.

- You → UX (Module: Auth): UX flow/wireframes/components →  
  You: NOT APPROVED: missing error/empty states. → UX revises → APPROVED.

- You → QA: TestCases-Auth.md → APPROVED.

- You → Engineering: Build Auth module → Verify → QA checks → Data/Sec/Docs/DevOps finalize → Ship.

---

## 8) Handoff card (paste between roles)

```
Project: <name>
Module: <module name> (if applicable)
Inputs to read:
- docs/PRD.md
- docs/Roadmap.md
- docs/Plan.md
- docs/DecisionLog.md
- Role-specific docs (e.g., docs/UX/<module>/*)
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
  Run full kickoff order (PO → PM → ARCH → UX → QA → DEVOPS → SEC → DATA → DOCS → ENGINEERING).

- Add “Favorites” feature  
  PO (AC) → PM (plan) → ARCH (skip unless new dep) → UX (flow/states) → QA (test cases) → ENG build → QA verify → DATA events → DOCS update.

- Mid-sprint change request  
  PO updates PRD/Backlog → PM updates Plan/Risks → ARCH/UX/QA re-check scope → ENG adjusts. Record in DecisionLog.

- External API change  
  ARCH ADR (mitigation) → PM risk update → QA regression case → ENG patch → DOCS changelog → DATA/SEC if data/permissions changed.

- Performance regression  
  ARCH sets budget/tuning ADR → ENG fixes/instruments → QA adds perf checks → DEVOPS ensures CI passes → DOCS note limits.

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
- Stale docs → Update artifacts in Phase 2 Verify; PM references in WeeklyStatus.
- Dependency sprawl → Engineering must ask; ARCH/PM log decision/tradeoffs.
- Ambiguous approvals → Use exact Annex phrases (“APPROVED: …” / “NOT APPROVED: …”).
- Long chats, no artifacts → Always write/update files; files are the contract.
- Skipping MCP discipline → Don’t run shell for tasks covered by MCP tools; always add_roots first; never make up a DTD URI.
---

## 11) FAQ

- Can I run roles in parallel?  
  Yes, if they read latest docs first and you sequence approvals to avoid conflicts.

- Do I need to paste protocols every time?  
  Prefer loading files (protocols/*.md + ANNEX.md) as system instructions. If your tool can’t load files, paste once per chat.

- How do they “continue” each other in separate chats?  
  Through docs in the repo. Each role reads inputs and writes outputs; the next role picks them up.

- What if a role disagrees with me?  
  They propose options with pros/cons; you decide via Annex approval handshake. Record in DecisionLog.

---

## 12) Glossary

- PRD: Product Requirements Document (what/why/for whom, with acceptance criteria).
- ADR: Architecture Decision Record (context, options, decision, consequences).
- AC: Acceptance Criteria (testable).
- DoD: Definition of Done (what must be true to call it done).
- WIP: Work in Progress (limit to keep focus).
- RAG: Red/Amber/Green project health status.

---

How to start today

1) Ensure protocols/ and docs/ exist (use your scaffold script if needed).
2) Open a new chat with Product (COG-PO) and Annex; run Phase 1.
3) Approve with Annex phrases. Continue down the kickoff order.
4) For each feature, follow the feature cycle flow (PO → PM → ARCH → UX → QA → ENG → QA → DATA → SEC → DOCS/DEVOPS).
