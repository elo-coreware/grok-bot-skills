---
name: qa-phase-fix
description: >-
  Use when Gene assigns exactly one OPEN implementable phase (fix, author, or
  reconcile), or when Katherine FAILs that phase PR
---
# qa-phase-fix

## WHEN TO USE

Gene assigns exactly one OPEN implementable phase from the current plan (fix plan,
coverage plan, or hygiene plan). Never start a second phase while you are still
implementing or waiting on Katherine for the current one. Ready unmerged PRs from
earlier phases are not a reason to stop. Also use this when Katherine FAILs the
current phase PR — stay on that branch, implement, re-invoke Bugbot.

## WORK TYPES

Each phase row names one work type:

- **Fix** — repair failing tests from a fix plan (existing behavior, unchanged).
- **Author** — create or extend tests from a coverage plan phase. Use only assertions
  and structure authorized in that phase row. Escalate rather than invent assertions
  the plan did not authorize.
- **Reconcile** — execute authorized merges, deletions, renames, and `git mv`
  relocations from a hygiene plan. Report actual assertion delta against the plan's
  stated number.

## ENGINEER LANES AND WORK LOCKS

You hold a whole plan document — never a phase of someone else's plan. You never edit
across the lock: no plan markdown, module globs, or branches belonging to the other
engineer. Base branches only within your own lane (origin/develop or your lane's
latest dual-PASS unmerged branch). Escalate to Gene to re-partition rather than
reaching into the other lock.

## SAME-MODULE HYGIENE + COVERAGE (Angelo 2026-09-09)

When Tindall shipped both a hygiene plan and a coverage plan for the same module:

1. **CI Fix phases still win the implement slot** — finish the active fix-plan phase
   before any Forms/Reconcile/Author work from Tindall plans.
2. **Hygiene (Reconcile) before Coverage (Author)** — clean/relocate/delete weak
   existing tests before writing new ones onto those paths.
3. **One phase at a time** — never run a hygiene phase and a coverage phase in
   parallel. Stack ready unmerged PRs as usual.
4. **Coverage Critical rows first** when Gene starts Author work (destroy/status/
   submission destroy/notification templates/category gaps as the plan ranks them).
5. **Skip or ESCALATE** coverage rows that depend on open Angelo product NOTES
   (e.g. missing controller methods, unknown module-permission) — do not invent
   product behavior.
6. Honor Katherine implementer notes on the plan PRs (assertion-delta caveats,
   dependency order such as "home for X before delete Y").

Gene names the exact phase and work type each assign. Do not self-start the other
plan while one phase is open.

## REQUIRED INPUTS AND ACCESS

- Phase number, work type, its file/failure table, verify commands, and the **base
  branch** Gene names (`origin/develop` or the latest dual-PASS unmerged phase branch
  **in your lane**).
- repo-delegate-to-cursor for all repo work.
- Authority: `.cursor/rules/test-failure-triage.mdc` (alwaysApply). If it conflicts with this skill, the repo rule wins.
- On a Katherine FAIL return: her triage table (valid in-scope items only). Do not re-triage those. Do not dismiss a finding she marked valid.
- Angelo's GitHub identity for any PR comment (see IDENTITY).

## IDENTITY — READ THIS EVERY TIME YOU COMMENT

Every PR comment you post — especially `cursor review` / `bugbot run` **and the verify evidence comment** — must appear under Angelo's GitHub account (`elo-coreware`), never under `cursor[bot]` or any integration bot.

Preferred posting path (in order):
1. **GitHub MCP** `add_issue_comment` with `owner=CorewareHub`, `repo=coreware-app-backend`, `issue_number=<PR>`, `body=...` — this uses Angelo's connected GitHub account.
2. Else `gh pr comment <number> --repo CorewareHub/coreware-app-backend --body "..."` only when `gh auth status` shows Angelo / `elo-coreware`.

After every comment, verify authorship of the newest issue comment. Accept only when `user.login` is `elo-coreware` (or Angelo's current login). If the author is `cursor[bot]`, or posting fails with 403 / not authenticated, STOP. Do not leave a bot-authored invoke as done. Tell Gene. Never fall back to the Cursor PR-management API or any integration token that posts as `cursor[bot]`.

The Bugbot invoke body is exactly `cursor review` (or `bugbot run`) — nothing else on that comment.

## CURSOR REVIEW INVOKE GATE (Angelo 2026-09-21; example CorewareHub/coreware-app-backend#6487)

Bots must not spam bare `cursor review` / `bugbot run` while Bugbot is still pending
(e.g. #6487: invoke at 23:38 UTC, another at 00:27 UTC, then cursor[bot] CLEAN at
00:29 on SHA 435e6130). **If a cursor Bugbot review is still pending** (existing
Bugbot run in flight, or a prior `cursor review` / `bugbot run` comment not yet
processed for current HEAD), do **NOT** post another `cursor review` yet.

**Before posting `cursor review` or `bugbot run`:**
1. Read current HEAD SHA.
2. List PR issue comments whose body trims to exactly `cursor review` or `bugbot run` (case-insensitive).
3. List reviews by `cursor[bot]`.
4. If any non-stale cursor[bot] review has `commit_id` == HEAD → already reviewed; do **not** post.
5. If the newest invoke comment has no later cursor[bot] review that covers HEAD (`commit_id` == HEAD with `submitted_at` after that comment, or any review on HEAD after the comment) → treat as **PENDING**; stay WAITING-BUGBOT; do **not** post another invoke.
6. Only post exactly one new invoke when: HEAD is not yet reviewed by cursor[bot] **AND** there is no pending unprocessed invoke for this PR.
7. Never post a second `cursor review` for the same HEAD SHA.
8. After a new commit, you may invoke once only after confirming no pending invoke remains from before that push (or the prior invoke was for an older SHA and Bugbot already reported / timed out — prefer wait for pending to clear or confirm stale before re-invoke).

Apply this gate on steps 10, 12, and 15 (and any fold re-invoke). Never freely re-post `cursor review` while PENDING.

## BRANCH NAMES

Prefer by plan type (lowercase, hyphens):

- Fix plan: `fix/ci-tests-phase-N-<slug>`
- Coverage plan: `test/coverage-<module>-phase-N-<slug>`
- Hygiene plan: `test/hygiene-<module>-phase-N-<slug>`

Garman appends `-ii` to each convention (e.g. `fix/ci-tests-phase-N-<slug>-ii`).
Both engineers run `git ls-remote --heads origin '<name>'` before creating a branch.

Cursor Cloud Agent may rename or push under `cursor/**` — that is acceptable when the
launcher forces it; do not fight a rename mid-flight. Tell Gene the actual branch name.
Do not treat `cursor/**` as a reason to skip verify.

## ACTIONS `run-tests-phase` — ANGELO OWNS FOR NOW

`.github/workflows/run-tests-phase.yaml` is `workflow_dispatch` only (no push branch filter). Cloud-agent `gh` cannot dispatch it (integration 403). **Angelo runs and observes that workflow himself when he wants Actions evidence.** Do not block handoff, audit, or the next phase on a missing Actions Pest run.

**Required verify path:** always run cloud-agent verify and quote real output. That is
the gate the owning engineer owns — not Actions.

- Margaret: `composer test:single -- <changed paths>`
- Garman: `TEST_TOKEN=9 composer test:single -- <changed paths>`

Angelo standing rule 2026-09-11: Garman always requests the test slot from Gene
and does not run concurrently with Margaret or Grace — even on `TEST_TOKEN=9`.
Slot independence after the `scripts/test-lib.sh` ephemeral-sweep fix is suspended
until Angelo lifts the standing rule.


## LOCAL PINT (Angelo 2026-09-15 / develop #6326) — mandatory

CI no longer auto-commits Pint. `.github/workflows/lint.yml` runs **`pint --test`**
on every pull_request and fails if style is dirty. You must format on the branch
before handoff.

- After PHP edits (implement, Bugbot fix, Katherine FAIL fix, or fold that touches
  PHP): on the cloud agent / box with the repo, run one of:
  - `./vendor/bin/pint --dirty` (preferred when vendor present)
  - or `pint --dirty` if Pint is on PATH
  - path-scoped: `./vendor/bin/pint -- <changed.php paths>`
- If Pint rewrites files: commit on **this** branch (e.g. `:art: pint` or fold into
  the handoff commit), push, then continue.
- Before Gene/Katherine handoff, confirm `./vendor/bin/pint --test` exits 0 (or
  `pint --test`). Quote that in the verify evidence comment when PHP changed.
- **Still never run `composer format`.** Pint replaces the old auto-commit path;
  `composer format` stays banned.
- Docs-only commits with no PHP diff: Pint optional (skip).

## SEQUENCE OF WORK

1. git fetch origin. **Stack inherit:** Gene names the base — latest dual-PASS
   unmerged phase branch **in your lane** (Phase N tip), else `origin/develop`.
   For a **new** Phase N+1 branch: create from that named tip (never from develop
   alone while a dual-PASS parent exists). Merge `origin/develop` into the tip
   first only if Gene asks or develop has moved past it. On conflicts: list
   `git diff --name-only --diff-filter=U`, run `git merge --abort`, notify Gene,
   STOP. Never resolve conflicts yourself.

2. Branch from that base using the naming convention for the work type. On a
   FAIL return, stay on the existing branch. Open the PR as **draft** targeting
   `develop`. Never mark it ready. Never merge.

   **Post-merge fold:** When Gene assigns a fold after Angelo merges, merge
   `origin/develop` into **this** PR only, push, re-verify / re-`cursor review` as
   needed, report MERGEABLE or hand for re-gate, and **stop**. Do not fold sibling
   PRs in the same assign.

3. Build the delegation brief with the phase table / plan Implementer instructions as SCOPE and launch one Cursor Cloud Agent via repo-delegate-to-cursor as configured in that skill's launcher settings (do not pin a specific Composer model version unless Angelo says otherwise).

4. Per file, classify using `.cursor/rules/test-failure-triage.mdc` **before editing anything**.
   - **Scaffolding failure:** setup is wrong; the asserted contract is untouched. Fix the test (`uses()`, fixtures, fakes, dates, uniqid scoping, teardown ordering, parallel collisions).
   - **Contract failure:** the assertion describes user-visible behavior, authorization, visibility, or data integrity. Do **not** rewrite it to match the app. Do not invert polarity. Escalate to Gene (who notifies Angelo), record an **ESCALATED** row in the current `docs/*-FAILING-TESTS-FIX-PLAN.markdown` (test path, assertion, suspected `app/` file and function, expected vs actual), and leave the test red.
   - "No app changes" / tests-only bounds what you may patch without approval. It never authorizes making a test agree with a bug.
   - Hard stops (never as a standalone test edit): polarity inversions (`toBeTrue`↔`toBeFalse`, `assertOk`↔`assertForbidden`, `assertDatabaseHas`↔`assertDatabaseMissing`, 403→200, accepting a 500); renaming `it(...)` to different product behavior; adding teardown for a record the test asserts should not exist; changing expected `''` / `null` / hidden to a default; deleting or weakening an assertion.
   - Allowed: framework-semantics corrections (PR 5778 — Inertia 409 vs 302). Not allowed: PR 5785-class product-behavior rewrites (AnonymousGuestBooking, WebsiteIntegration, ProductSearchPage).
   - Do not patch `app/` unless Angelo assigned that specific bug or the merged plan's Implementer instructions authorize named app files. An unstated test-vs-app choice is an audit failure.

5. Verify with the per-engineer command from ACTIONS above. Exactly one test command
   at a time for your own runs. Quote the real output. Do not start another phase's
   tests while this command runs.

6. Only if a migration changed, regenerate the schema dump as a separate ":robot: regenerate test schema dump" commit:
   `php -d memory_limit=2G artisan test:generate-schema-dump --env=testing`
   Commit all three schema files together. Keep the dump delta surgical — do not mass-rewrite unrelated dump noise (Bugbot TOO_LARGE risk). In DUAL mode only one engineer regenerates the dump at a time — Gene serializes migration phases; the other merges `origin/develop` afterward.

7. Update **your locked** plan doc: mark **this** phase COMPLETE with the actual delta (failure
   count for Fix; tests added for Author; assertion delta for Reconcile) and the
   branch name, or PARTIAL with ESCALATED rows for leftover contract failures. Do not
   touch other phases' rows except as already stacked on the base branch. Do not edit
   the other engineer's locked plan document. Hygiene Reconcile: assertion drop is FAIL
   unless it matches the plan's authorized delta.

8. **Pint gate:** if any PHP files changed on this branch since the last Pint-clean
   tip, run LOCAL PINT (`./vendor/bin/pint --dirty` then `./vendor/bin/pint --test`).
   Commit style fixes on this branch before the handoff commit (or include them in it).

9. Commit with the work-type prefix and push. This is the **handoff commit**. Do not
   comment `cursor review` on WIP commits before this.

   - Fix: `:bug: fix phase N <slug>`
   - Author: `:white_check_mark: add <module> <slug> tests`
   - Reconcile: `:recycle: <module> test hygiene <slug>`

   For Reconcile phases, include in the handoff message the actual assertion-count
   delta vs the plan's authorized number.

10. On the handoff commit only, run the **CURSOR REVIEW INVOKE GATE**, then post exactly one `cursor review` under Angelo's identity per IDENTITY above (skip if HEAD already reviewed or an invoke is still PENDING). Wait until `cursor[bot]` has a review whose `commit_id` equals HEAD. Stay WAITING-BUGBOT while PENDING — do not re-comment.

11. Implement in-scope GitHub Bugbot findings on this branch. In-scope means the finding is about a file or hunk in this phase's diff. Do not classify a finding as a false positive to skip it. Unclear items: leave for Katherine. Re-verify with one test command if tests changed. Do not implement a Bugbot item that would change app behavior or invert a contract assertion. If PHP changed again, re-run LOCAL PINT before the next `cursor review`.

12. If step 11 produced a new commit, run the **CURSOR REVIEW INVOKE GATE** on the new HEAD, then post at most one `cursor review` under Angelo's identity (only if HEAD is not yet reviewed and no invoke is PENDING). Wait for `cursor[bot]` on the new SHA. Never a second invoke for the same HEAD. Repeat until HEAD has a Bugbot review and you are not adding commits.

13. **Verify evidence comment (required before Gene/Katherine handoff):** as elo-coreware, post a PR issue comment that Angelo can skim while reviewing. Include:
    - That verify ran on the **cloud agent** (Composer / Cursor cloud computer), not Actions `run-tests-phase.yaml`
    - Exact verify command(s) (`composer test:single` or `TEST_TOKEN=9 composer test:single`)
    - Verbatim `Tests: … passed (… assertions)` line(s)
    - HEAD SHA this verify covers
    - If PHP changed: note that `./vendor/bin/pint --test` (or `pint --test`) was clean on that SHA
    Do this after the final successful verify on the handoff SHA (and again after any Bugbot-fix re-verify). This is separate from `cursor review`.

14. Hand the still-draft PR to Gene for Katherine's audit. Report the head SHA, the Bugbot review SHA, the author login of your `cursor review` and verify comments, cloud-agent verify output, lane, and the base branch you used. Do not start the next phase until Gene assigns it (after this one is dual-PASS). Gene may assign your next phase **before Angelo merges** this PR — stack on your dual-PASS tip when he says so (Phase N+1 inherits this tip). Waiting on Bugbot is a valid hold; stay on this phase until Gene says dual-PASS. Non-Pest work (implement/prep/`cursor review`/fold) does **not** need the test slot.

15. When Katherine FAILs: implement every valid in-scope item from her triage table on the same branch, re-verify with one test command, run LOCAL PINT if PHP changed, push, run the **CURSOR REVIEW INVOKE GATE**, then post at most one `cursor review` under Angelo's identity on that new commit (skip if PENDING or HEAD already reviewed), wait for Bugbot, post an updated verify evidence comment, then hand back to Gene. Never mark a Katherine-valid item false-positive to skip it. Still do not invert contract assertions or patch `app/` unless Angelo assigned it / the plan authorizes it.


## BOT VERIFY / HANDOFF (Angelo standing rule 2026-09-21, via Gene)

Phase PR handoff and dual-PASS must **not** wait on the full self-hosted CI
`Tests` suite (~60 min). Engineer verify = **touched tests only** on the shared
machine under Gene Pest GRANT + Bugbot CLEAN==HEAD + local Pint/lint.
Ambient full-suite CI red ≠ handoff/babysit blocker unless tip-caused.
Do not tell engineers to wait on Actions / self-hosted full Tests for phase PRs.

## HOW TO VALIDATE

- Every listed failure passes **or** is an ESCALATED contract failure with a plan-doc row.
- No file outside this phase's scope changed, except files already on the stacked base branch.
- No new application-behavior change unless Angelo assigned that specific bug or the plan authorizes named app files.
- No polarity inversion or product-behavior assertion rewrite.
- Verify output is pasted, not summarized — both in the Gene handoff **and** in the PR verify evidence comment.
- Total assertion count across **this phase's** changed tests did not drop, except
  Reconcile phases where the hygiene plan authorized the exact delta.
- `cursor review` was posted only on the handoff commit and on FAIL-fix commits, never on WIP, at most once per HEAD SHA, only after CURSOR REVIEW INVOKE GATE, and every such comment's author is Angelo (not `cursor[bot]`).
- A verify evidence comment exists on the PR for the handoff HEAD.
- HEAD has a `cursor[bot]` review before handoff to Gene.
- The PR is still draft.
- If PHP changed: `./vendor/bin/pint --test` (or `pint --test`) is clean on HEAD.

## WHAT TO RETURN

Branch, PR number, lane, base used, head SHA, Bugbot review SHA, author login of each `cursor review` and verify comment, per-file table of classification (scaffolding vs contract) and fix chosen, verify output, failures resolved vs listed, ESCALATED rows, plan-doc delta, Bugbot items implemented vs left for Katherine.

## WHAT REQUIRES APPROVAL

Nothing in the default scaffolding path. App/` patches need Angelo (or merged plan Implementer authorization). Never run `composer format` (banned). **Do** run local Pint per LOCAL PINT. Never merge. Never mark the PR ready (Katherine does that on dual PASS).
