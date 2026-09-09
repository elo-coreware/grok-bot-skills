---
name: qa-root-cause-investigate
description: >-
  Use this when an ESCALATED or unclear CI test failure needs git/product
  context before anyone implements a fix — intentional change vs regression,
  route/column/prop history, or domain traces — then update the fix plan so the
  implementer has explicit instructions.
---
# QA root-cause investigate

## When to use

- An ESCALATED plan row (contract / app suspicion) before assigning an implementer.
- A failure that might be intentional product/framework change (missing route, dropped Inertia prop, removed column, stricter validation).
- The orchestrator asks for investigate-first on a phase.
- Do **not** use for pure scaffolding already classified in the plan (factories, naming helpers, test credit setup) unless history is disputed.

## Required inputs and access

- Phase number / failure rows from the current `docs/*-FAILING-TESTS-FIX-PLAN.markdown`.
- Failing test path(s), assertion/exception text, suspected app area from the plan.
- Repo via `qa-delegate-to-cursor` (Composer 2.5 Fast).
- Authority: `.cursor/rules/test-failure-triage.mdc` (wins on scaffolding vs contract).

Read-only during diagnosis. Avoid long test runs that collide with an active implementer; prefer static/history analysis unless asked to reproduce with one `composer test:single`.

## Sequence of work

1. Fetch `origin/develop`. Confirm the plan file and quote the phase rows in scope.
2. Read the failing test(s) enough to name the **symbol under dispute** (route name, column, Inertia prop, service method, HTTP status expectation).
3. **Git archaeology** (cite SHAs / PR numbers):
   - `git log -p --all -- <paths>` / `git log -S'<symbol>' --oneline`
   - Blame the current omission or behavior change
   - Open linked PR bodies when a merge commit is the change
4. **Call-path trace** (read-only): controller/route → service → model/migration for the disputed behavior. Note how related domains use the same concept (e.g. how `location_id` flows in payment hydrate vs store vs connection test).
5. Compare sibling tests (same module) for the expected contract already on develop.
6. Classify each failure cluster:
   - **INTENTIONAL** — product/perf/security change; tests stale
   - **REGRESSION** — accidental break; app should be restored/fixed
   - **UNKNOWN** — escalate; do not guess
7. Recommend **one** next action:
   - Update tests (framework-semantics / stale assertion) — allowed class
   - App bug fix (file + function) — needs product go-ahead if not already given
   - Product sign-off required (do not change app or tests yet)
   - Leave red + keep ESCALATED row
8. **Update the fix plan before any implementer starts** (required):
   - Edit the current `docs/*-FAILING-TESTS-FIX-PLAN.markdown` on a docs branch
   - For each investigated phase: set classification, evidence (SHAs/PRs), and an explicit **Implementer instructions** block (do / do not; files; verify command)
   - Reclassify rows (ESCALATED → OPEN scaffolding, stay ESCALATED, or WONTFIX pending sign-off) so the phase table matches the decision
   - Open a **draft** plan-update PR via `qa-delegate-to-cursor` (docs-only). Do not merge. Summarize for the orchestrator
   - Never start `qa-phase-fix` yourself; the orchestrator assigns the implementer only after this plan update is approved/merged (or the orchestrator explicitly waives merge and pastes the instructions)

## How to validate

- Every conclusion cites at least one commit SHA or PR number, or explicitly says UNKNOWN with what was searched.
- INTENTIONAL requires a sibling test or PR description that matches current app behavior.
- Recommendations never invert contract assertions to match a bug.
- Plan PR contains implementer instructions specific enough that the implementer does not need the investigation chat.
- No app/test fix commits in the investigation PR (docs-only), unless the orchestrator ordered a combined docs+fix (default: docs-only).

## What to return

Chat report to the orchestrator, plus the draft plan-update PR URL:

| Field | Content |
|-------|---------|
| Symbol | route / prop / column / method |
| Classification | INTENTIONAL / REGRESSION / UNKNOWN |
| Evidence | SHAs, PRs, sibling tests |
| Recommend | test update / app fix / sign-off / leave red |
| Plan PR | URL of docs update with implementer instructions |
| Risk | what breaks if we choose wrong |

## What requires approval

Product owner (via orchestrator) before any app change or before updating tests when classification is UNKNOWN. INTENTIONAL + clear sibling precedent → plan update with test-only instructions, then orchestrator assigns implementer after plan merge (or explicit waiver).
