---
name: qa-fix-plan-build
description: >-
  Use this after qa-ci-log-pull and qa-validity-scan when no current plan covers
  the latest develop CI run, or the newest plan's baseline is stale.
---
# QA fix plan build

## When to use

After qa-ci-log-pull and qa-validity-scan, when no current plan covers the latest develop CI run, or the newest plan's baseline is stale.

## Required inputs and access

- Metrics table, failure inventory, and seed from qa-ci-log-pull.
- Findings from qa-validity-scan.
- The previous plan in `docs/` for the Supersedes line and delta comparison.
- Template of record: `docs/20260831-FAILING-TESTS-FIX-PLAN.markdown`.
- Repo read access via `repo-delegate-to-cursor` (Composer 2.5 Fast) for **feature-history / git archaeology** (required — not optional).
- Authority: `.cursor/rules/test-failure-triage.mdc`. Deeper ESCALATED follow-up still uses `qa-root-cause-investigate` when Gene assigns it.

## Sequence of work

1. Name it `docs/YYYYMMDD-FAILING-TESTS-FIX-PLAN.markdown`. Open with a Supersedes link to the prior version and a link to the source log.
2. Baseline table — total, passed, failed, skipped, failure rate, duration, branch, seed, failing-file count — each with a delta column vs the prior version.
3. Progress audit — net failure change; prior clusters cleared; clusters still open with updated counts; what is new since the last plan.
4. Cleared / Escalated / New sections as tables. New failures get module, root cause, and proposed fix per file.
5. Test validity summary — verdict counts, plus a validity-blockers table for files where the test code must change before the app can be judged.
6. Root-cause categories with counts, percentages, and a fix pattern each.
7. Module breakdown with top files.
8. Phased plan. Group by shared root cause, never by module — one HTTP-fake fix cleared 59 Coreforce failures in Phase 2 and 26 Mailgun failures in Phase 3, and that only works when grouping is causal. Every failure lands in exactly one phase. Size each phase to one agent session. Order by ROI: test-infrastructure quick wins first, then largest cluster descending. Each phase gets a file/failure/root-cause/solution table and its verify commands.
9. **Feature-history pass (required before opening the plan PR).** For every phase, study how the feature under test is supposed to work — do not invent intent from the failure message alone:
   - Name the **symbol / feature** under dispute (route, Inertia prop, column, service method, validation rule, HTTP contract).
   - **Git archaeology** — cite at least one commit SHA or PR number per phase (or explicitly `UNKNOWN` with what was searched): `git log` / `git log -S'<symbol>'` / blame on the app + test paths; open linked PR bodies when a merge is the change.
   - Brief **call-path note** (controller/route → service → model) for non-scaffolding phases.
   - Classify the cluster: **INTENTIONAL** (tests stale) / **REGRESSION** (app should fix) / **SCAFFOLDING** (test infra only) / **UNKNOWN** (needs Angelo or a later `qa-root-cause-investigate`).
   - Write an **Implementer instructions** block for that phase: do / do not; named files; verify command; never invert contract assertions to match a bug.
   - Pure scaffolding phases may use a short history note (`SCAFFOLDING — no product-intent dispute`) but must still not invent app behavior.
10. Full failure inventory sorted by count, with a phase column.
11. Recommended fix order and the per-phase AI session checklist.

## How to validate

- Per-phase failure counts sum to the CI total. State the arithmetic.
- Every failing file appears in exactly one phase and once in the inventory.
- Every count traces to a citable log line. Never estimate silently.
- Every phase has a runnable `composer test:single` command.
- **Every phase has a Feature-history subsection** with: symbol, cited SHA/PR or explicit UNKNOWN + search notes, classification, and Implementer instructions. A plan without this is incomplete — do not open the PR until it is present.

## What to return

The plan file, shipped as its own PR via repo-delegate-to-cursor: branch `docs/YYYYMMDD-failing-tests-fix-plan`, commit `:memo: add <month day> failing tests fix plan vN`. Summarize the phase table **and** the per-phase INTENTIONAL/REGRESSION/SCAFFOLDING/UNKNOWN counts in chat for Gene.

## What requires approval

Push and PR creation need no approval. Angelo merges the plan PR on GitHub. Gene may release Margaret to start Phase 1 once it is merged. Escalate UNKNOWN or app-fix recommendations to Gene/Angelo rather than guessing.
