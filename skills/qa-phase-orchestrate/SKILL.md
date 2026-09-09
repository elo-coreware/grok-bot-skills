---
name: qa-phase-orchestrate
description: >-
  Use this when a develop CI run failed, a phase finished, an audit returned, a
  GitHub cursor[bot] review landed, or the daily brief is due
---
# qa-phase-orchestrate

## WHEN TO USE

A develop CI run failed, a phase finished, an audit returned, Margaret handed a draft PR, a GitHub `cursor[bot]` review landed, or the daily brief is due.

## REQUIRED INPUTS AND ACCESS

- docs/ for the current plan. Direct messages to Aaron, Tindall, Margaret, Katherine.
- Read-only git and gh. You never edit the repo and never run test commands.
- Authority: `.cursor/rules/test-failure-triage.mdc` on develop. Repo rules win if they conflict with this skill.
- Related skills: `qa-fix-plan-build` (Aaron), `qa-module-test-inventory` (Tindall),
  `qa-coverage-plan-build` (Tindall), `qa-suite-hygiene-plan-build` (Tindall),
  `qa-root-cause-investigate` (Aaron / Tindall), `qa-phase-fix` (Margaret).

## MODULE BACKLOG RANKING (coverage / hygiene)

Coverage work has no CI trigger. Aaron/Bill **may** draft plans (VERIFY none)
while Margaret implements — Angelo 2026-09-09. When ranking modules for Tindall
(idle or alongside a Fix implement), rank modules:

1. Bug-fix commit density over trailing 6 months on `app/` under module globs
   (`:bug:`, `fix(` in subject)
2. Surface symbol count from a light inventory pass (or prior Tindall inventory)
3. Inverse of current test file count (under-tested modules rank higher)
4. Float **payments** and **auth** modules to the top when scores tie

Assign Tindall one module at a time: inventory → coverage and/or hygiene plan PRs.

## SEQUENCE OF WORK

1. Read the newest plan in docs/ matching one of:
   - `*-FAILING-TESTS-FIX-PLAN.markdown`
   - `*-TEST-COVERAGE-PLAN.markdown`
   - `*-TEST-HYGIENE-PLAN.markdown`

   **Fix plan:** If none covers the latest develop CI run, hand the run URL to Aaron
   for qa-ci-log-pull → qa-validity-scan → qa-fix-plan-build (plan-build **must**
   include the per-phase feature-history / git archaeology pass), and wait for the
   plan PR.

   **Coverage / hygiene:** If no module plan is in flight and the fix loop is idle,
   pick the top module from the backlog ranking and assign Tindall
   qa-module-test-inventory → qa-coverage-plan-build and/or qa-suite-hygiene-plan-build.

2. When a plan PR is open:

   **Fix plan (Aaron):** Reject back to Aaron if any phase lacks symbol + cited SHA/PR
   (or explicit UNKNOWN with search notes) + classification + Implementer instructions.
   Assign Katherine qa-validity-scan mode (b) on every test file listed **and** ask her
   to challenge feature-history classifications.

   **Coverage plan (Tindall):** Assign Katherine qa-validity-scan mode (d) on every
   proposed phase. Reject back to Tindall if proposals lack app/ file:line citations
   or propose prohibited service unit tests.

   **Hygiene plan (Tindall):** Assign Katherine qa-validity-scan mode (b) on every
   existing test file listed, plus verify authorization records (expected assertion
   delta, rename records, merge equivalence).

   Authority: `.cursor/commands/automated-tests-validity-detection.md`. Her verdict is
   binding: FAIL returns to the plan author (PR stays draft); PASS or PASS WITH NOTES —
   she marks the PR ready, then Gene's plan-content approval. Plan PRs are not blocked
   on Bugbot unless `cursor[bot]` already commented; then she triages those threads.

3. On Gene approval, notify Angelo that the plan PR is ready to merge, with the comment URL. Never merge it yourself. For a **new** plan, wait until Angelo merges once. After that, phase work does **not** wait on his merges. Gene does **not** approve plan content when feature-history is missing or every ESCALATED/UNKNOWN cluster is unevidenced.

4. **ESCALATED / UNKNOWN / unclear phases — deepen before implement.** Plan-build's light history pass is necessary but not always sufficient. Before assigning Margaret an ESCALATED or UNKNOWN phase (or any phase where product intent is still unclear after plan-build), assign Aaron `qa-root-cause-investigate`. He must:
   - Classify INTENTIONAL / REGRESSION / UNKNOWN with cited SHAs/PRs
   - **Edit the fix plan** with an explicit Implementer instructions block for that phase
   - Open a draft docs plan-update PR
   Do **not** assign Margaret until that plan update is merged (or Angelo/Gene explicitly waives merge and pastes the instructions into Margaret's assign). Scaffolding-only OPEN phases with clear plan fixes and SCAFFOLDING history notes may skip the deeper investigate.

5. Assign Margaret the next **implementable** OPEN phase whose plan row already has
   clear implementer instructions and work type (Fix / Author / Reconcile). Skip
   infra-only phases Angelo owns. Prefer the plan's recommended order among remaining
   OPEN phases. Fix, coverage, and hygiene phases share one implement slot — exactly
   one phase **implementing** at a time (shared computer and `test_tenant_1` /
   `test_landlord_1`). Ready-but-unmerged PRs do not count as in flight.

   Base branch for her: the latest unmerged phase PR that Katherine dual-PASSed, else
   `origin/develop`. Tell her that base explicitly, the work type, and the branch
   naming convention. Point her at the **plan section** for this phase (not only chat).
   Remind her: draft, handoff commit with work-type prefix, `cursor review` as Angelo
   via GitHub MCP (not WIP), implement in-scope Bugbot items, re-invoke after any new
   commit, cloud-agent `composer test:single` with quoted output, do not mark ready.
   Angelo himself runs/observes `run-tests-phase.yaml` for now — do not block Margaret
   on workflow_dispatch. Authority: `.cursor/rules/test-failure-triage.mdc`.
   Scaffolding failures: fix the test. Contract failures: escalate, write an ESCALATED
   plan row, leave the test red. Author: do not invent assertions beyond the plan.
   Reconcile: match the plan's authorized assertion delta. Never invert assertions to
   match a bug. "No app changes" means escalate, not invert. Framework-semantics
   (PR 5778) are allowed. She must not patch `app/` unless Angelo assigned that
   specific bug (or the plan's implementer instructions explicitly authorize named app
   files).

6. When Margaret reports the PR is open, **do not assign Katherine yet**. Confirm `cursor[bot]` has a review whose `commit_id` equals HEAD. If not, send Margaret back to comment `cursor review`. If in-scope Bugbot findings are unfixed, send her back to implement (she must not skip by calling them false positives). If the phase inverted contract assertions or added `app/` without Angelo's go-ahead / plan authorization, send it back before audit. Do not gate audit on `run-tests-phase` Actions while Angelo owns that dispatch.

7. When HEAD has a `cursor[bot]` review, assign Katherine qa-fix-audit. Her verdict is binding.

8. WAITING → Margaret comments `cursor review`. FAIL → Margaret stays on **this** phase (do not start the next one), implements, pushes, comments `cursor review`, Katherine re-audits. PASS or PASS WITH NOTES **and** Bugbot-on-this-SHA clean → Katherine marks ready. Never mark ready yourself.

9. On that dual PASS, notify Angelo that the PR is ready to merge (comment URL, merge order: lowest phase number first). **Do not wait for him.** Immediately return to step 4/5 for the next implementable OPEN phase. Stack ready PRs overnight.

10. After Angelo merges one or more phase PRs, have Aaron re-baseline against the next develop CI and record actual deltas. A miss of more than a third means Aaron re-diagnoses before more phases start **only if Margaret is idle**. If she is mid-phase, let her finish, then pause if Aaron's re-diagnosis says so.

There is no GitHub listener yet. The Bugbot invoke is Margaret's `cursor review` on the handoff commit and on FAIL-fix commits. Phase Pest on Actions is Angelo-operated `workflow_dispatch` of `run-tests-phase.yaml` for now.

## APPROVALS YOU OWN

You approve qa-validity-scan results and plan content — only when no BLOCKER is unaddressed, every HIGH is fixed or justified in writing, Katherine's plan-validity PASS (or PASS WITH NOTES) is on record for a new plan, **and every phase has feature-history evidence** (SHA/PR or explicit UNKNOWN). For investigate plan-updates: approve content when classifications are evidenced and implementer instructions are unambiguous; escalate UNKNOWN or app-fix recommendations to Angelo. Escalate instead of approving when: a fix needs app/ business-logic changes without Angelo go-ahead; assertion count dropped; a root cause is unknown; the same file failed audit twice; a run was served by a model other than Composer 2.5 Fast; or Margaret and Katherine disagree. You never approve a merge.

## HOW TO VALIDATE

Exactly one phase implementing. Multiple ready unmerged phase PRs are expected when Angelo is away. Every ready phase PR has Katherine PASS **and** `cursor[bot]` on that head SHA. Merge order is lowest phase number first. Phase diffs do not invert contract assertions. They do not introduce application-behavior changes unless Angelo approved them or the merged plan's implementer instructions authorize named app files. New plans without per-phase feature-history are incomplete. ESCALATED/UNKNOWN phases do not reach Margaret without deepened `qa-root-cause-investigate` + plan update when still unclear after plan-build (unless waived).

## WHAT TO RETURN

A status table — phase, owner, state, failures addressed, PR, blocker — plus the single next action and its owner. The daily brief is five lines: shipped, in flight, ready (unmerged stack), needs Angelo, next.

## WHAT REQUIRES APPROVAL

Nothing you do requires approval, because you take no repo actions. Never run composer format. Never merge.
