---
name: pr-finding-remediate
description: >-
  Use when pr-bugbot-sweep has valid in-scope findings to fix on the assigned
  feature PR branch
---
# pr-finding-remediate

## WHEN TO USE

pr-babysit-loop is in REMEDIATING state and the combined ledger has one or more
**Valid + in-scope + Fixed?=no** items. Fix them in severity order: Critical,
then High, then Medium.

## REQUIRED INPUTS AND ACCESS

- owner/repo + base (`develop` for coreware-app-backend, or `develop/develop` for
  boss-control-tower), PR number, branch name, current HEAD SHA.
- **DEV branch rule (boss-control-tower):** Tips default to `feature/<name>` or `fix/<name>`; use `develop/<feature-name>` only for visual confirmation on https://dev.coreware.app. Never tip-push experiments onto `develop/develop`
  (main). PROD web hosts — observe / peek only (Angelo 2026-09-18) for BOTH https://controltower.coreware.app (landlord Control Tower PRODUCTION) and https://coreware.coreware.app (backend/tenant PRODUCTION): bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Backend base remains `develop`.
- The finding ledger row(s) to fix, with file:line, hunk, and source.
- Committed implementation plan on the branch (scope boundary for product behavior).
- repo-delegate-to-cursor for implementation work.
- Wernher or Gene must grant the test slot before any composer test:single run.

Authority:
- `.cursor/rules/codebase.mdc`, `.cursor/rules/test-isolation.mdc`,
  `.cursor/rules/test-failure-triage.mdc`
- `.cursor/rules/currency-display.mdc`, `.cursor/rules/datetime-display.mdc`,
  `.cursor/rules/pricing.mdc`, `.cursor/rules/credit-gate.mdc`,
  `.cursor/rules/model-auditing-enforcement.mdc`, `.cursor/rules/realtime-reverb.mdc`
- `.cursor/commands/git-commit.md`

## SCOPE RULES

- Edit only files already in `git diff <base>...HEAD --name-only` on this PR
  (`<base>` = PR base: `develop` or `develop/develop`), plus minimal supporting
  changes in those same files.
- New files or product-behavior changes beyond the committed implementation plan
  require escalation to Angelo before proceeding.
- Never call a finding a false positive to skip it — the triage verdict from
  pr-bugbot-sweep is binding.
- If tests are touched: scaffolding failures fix the test; contract failures
  fix the app. Never invert assertions to match a bug.

## SEQUENCE OF WORK

1. **Batch.** Group related findings into one remediation pass when they touch
   the same files or root cause. Do not split unnecessarily.

2. **Delegate.** Launch repo-delegate-to-cursor with:
   - GOAL: Fix the listed valid in-scope findings on branch `<branch>`. Cite each
     finding by file:line and hunk. Stay within PR diff scope and the implementation
     plan at `<path>` or "none".
   - BRANCH: do not create a new branch — work on the assigned PR branch.
   - SCOPE: explicit file list from the findings + implementation plan files.
   - CONSTRAINTS: never commit on develop / develop/develop / main / master; never merge;
     never run composer format; follow all .cursor/rules listed above; if a fix needs
     behavior beyond the plan, stop and report ESCALATED.
   - VERIFY: `composer test:single -- <affected test paths>` when tests exist for
     the changed area — **only after Gene grants the test slot**. If no slot,
     set VERIFY to `none` and note tests deferred.
   - REPORT BACK: files changed, fix per finding, verify output, anything unfixed.

3. **Commit.** Per `.cursor/commands/git-commit.md`:
   - Extract issue number from branch name (or state "No issue number found").
   - Use gitmoji short message, no trailers.
   - Never stage a BugBot-generated plan (git-commit step 5 exclusion rules).
   - Do not stage the implementation plan on fix commits unless it was modified
     as part of the fix (normally leave the plan untouched until pr-plan-doc-retire).

4. **Push.**
   ```bash
   git push origin HEAD
   ```

5. **Re-invoke Bugbot app (gated).** Run the **CURSOR REVIEW INVOKE GATE** in
   pr-babysit-loop first (HEAD SHA, bare invoke comments, cursor[bot] reviews).
   If HEAD is already reviewed by cursor[bot], or a prior `cursor review` /
   `bugbot run` is still PENDING for this HEAD, do **not** post — return to
   WAITING-BUGBOT. Only when the gate allows, comment on the PR as Angelo:
   ```
   cursor review
   ```
   Exactly one invoke per HEAD SHA. Use GitHub MCP `add_issue_comment` with the
   assigned `owner`/`repo` (CorewareHub/coreware-app-backend or
   CorewareHub/boss-control-tower). Verify comment author is `elo-coreware` (or
   Angelo's current login). If `gh`/MCP returns 403, is unauthenticated, or it
   lands as `cursor[bot]`, STOP and tell Gene. Never fall back to the Cursor
   PR-management API or any integration token that posts as `cursor[bot]` — a
   bot-authored invoke is not a completed invoke.

6. **Update ledger.** Mark fixed items Fixed?=yes with the new commit SHA. Return
   to pr-babysit-loop WAITING-BUGBOT state.

## FORBIDDEN

- Posting a second `cursor review` / `bugbot run` for the same HEAD SHA, or
  re-invoking while a prior bare invoke is still PENDING (Angelo 2026-09-21;
  example #6487).
- Skipping a valid finding by reclassifying it without a fresh bugbot-triage pass.
- Resolving cursor[bot] review threads on GitHub.
- Running composer test:single without Gene granting the test slot.
- Running `composer format` (banned — use local Pint instead).
- Committing BugBot-generated plan files.

## HOW TO VALIDATE

- Every listed finding addressed or explicitly reported unfixed with reason.
- Commit follows git-commit conventions.
- Push succeeded; new HEAD SHA recorded.
- At most one `cursor review` posted as Angelo for this HEAD, and only after
  CURSOR REVIEW INVOKE GATE (or explicitly skipped because PENDING / already reviewed).
- Test output quoted when slot was granted and tests ran.

## WHAT TO RETURN

New HEAD SHA, commit message, files changed, findings fixed (with SHA), findings
still open, verify output or "tests deferred — no slot".

## LOCAL PINT (Angelo 2026-09-23) — full-repo, match CI

CI `.github/workflows/lint.yml` ("Check Code Style") runs bare `pint --test` with **no** path filter — it scans the whole Pint-configured tree. Babysit / phase verify must match that gate.

**Fix (write):** run `./vendor/bin/pint` (or `pint`) with **no** `--dirty` and **no** path args so it reformats the entire tree. Commit `:art: pint` (or fold into the remedi commit) if files change. Never `composer format`. Do **not** use `--dirty` or path-scoped Pint as the only fix when Check Code Style is red or when preparing MERGE-READY.

**Verify:** `./vendor/bin/pint --test` must exit 0 on HEAD (full tree). Equivalently, confirm GitHub Actions job `lint (8.3)` / Check Code Style is **green** on this HEAD (job URL is fine evidence). Style issues anywhere under Pint paths on the tip **are** babysit blockers — fix them on this branch until full `pint --test` is green. Do **not** dismiss full-repo lint red as "ambient / not tip-caused."

Docs-only or non-PHP commits: skip only when `pint --test` (or the Actions lint job) is already green on HEAD.


## WHAT REQUIRES APPROVAL

Pushing fix commits needs no approval when posting as Angelo. Escalate when a fix
requires scope beyond the implementation plan, remediation fails twice on the same
finding, or the test slot is denied while tests are required.
