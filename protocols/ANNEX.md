# Annex A — Shared Handshake & Scales

[InstABoost: ATTENTION :: This annex is common to all role protocols. Where conflicts arise, a role’s Constitution and this Annex override other instructions.]

Approval handshake
- APPROVED: <doc/step> — proceed to the next step per docs/HandoffQueue.md (reference module slug if applicable).
- NOT APPROVED: <reason> — revise and resubmit; do not proceed.
- Gate (Baseline mode) — Draft/update docs/ files is allowed without prior approval. No source-code or external-state changes (pub/git/GitHub/workflows) until APPROVED. Present plan + diffs + commands first for any such changes.
- HANDOFF OPEN: <modules/list> — authorizes PM to create issues from ENG Brief(s).
- Loop Mode handshake (before any Phase 2 “Module Loop”):
  • Ask: “Loop mode? [run-until-done | soft-cap | stop-per-module]”
  • If soft-cap, ask: “cap = <N modules this run>”
  • Confirm: “LOOP MODE: <run-until-done|soft-cap|stop-per-module> [cap=N]”
  • Default if not specified: stop-per-module (safer).
  • In run-until-done (or soft-cap), proceed through modules for your role; pause early only if a high-risk decision/ADR is triggered or information is missing.

Role Identity Lock (applies to every role)
- At the start of each run, the role must explicitly confirm:
  • Identity & Mission (role name and scope)
  • Allowed outputs (docs/ files per role protocol)
  • Forbidden actions (e.g., coding, issue creation, external ops) unless explicitly permitted
  • Read boundaries (protocols/ and docs/ only by default; reading code/pubspec.yaml or other paths requires explicit operator permission)
  • Current Phase/Step and the approval gate where it will stop

Document status header (recommended for every produced doc)
- At top: Status: Draft | In Review | Approved; Version; Owner; Date; Related: links to PRD/Roadmap/ADRs.

Priority scale (Roadmap/Backlog)
- P0 = must for v1; P1 = high; P2 = nice-to-have; P3 = backlog.

Risk matrix
- Likelihood: L1 low, L2 medium, L3 high.
- Impact: I1 low, I2 medium, I3 high.
- Score = LxIx. Mitigate ≥ 6; Monitor 3–5; Accept ≤ 2.

Bug severity (QA)
- S1 Critical (blocks core flows/data loss), S2 Major (workaround exists), S3 Minor (cosmetic), S4 Trivial.

Definition of Done (cross-role)
- Docs updated (README/CHANGELOG or feature doc).
- Analyzer/lints clean; tests for critical paths passed.
- Accessibility & performance sanity check completed.
- Release impact noted (if user-facing).
- For dev handoff: docs/eng_briefs/<module>.md exists and is linked from issues/PRs.

Commit convention (suggested)
- type(scope): summary — e.g., feat(auth): optimistic login cache
- types: feat, fix, refactor, perf, test, docs, chore, ci.

Change control
- Any scope change is captured in Backlog and referenced in DecisionLog with rationale.

MCP-First (Gemini CLI)
- Use MCP tools where available; avoid raw shell for covered tasks:
  - Tests: run_tests
  - Format: dart_format
  - Analyze: analyze_files
  - Fixes: dart_fix
  - Pub: pub (get/add/remove)
- Project roots: call add_roots at start; tools run only under these roots. Use remove_roots when done.
- Flutter runtime (DTD): only connect via connect_dart_tooling_daemon if the user provides a DTD URI (suggest “Copy DTD Uri to clipboard”). Do not fabricate URIs. Default to code-only workflows; do not run the app.
- GitHub: use the GitHub MCP server; call get_me early to scope context; enable only needed toolsets.
- Docs: when up-to-date API examples are required, request Context7 docs for packages/versions found in pubspec.yaml.
- Thinking: for complex/ambiguous tasks, optionally use sequential_thinking briefly (1–3 thoughts) before acting.
