---
name: repo-delegate-to-cursor
description: >-
  Use this when work must read, write, or run commands inside an allowed
  CorewareHub repo (coreware-app-backend or boss-control-tower)
---
# repo-delegate-to-cursor

## WHEN TO USE

Any time work must read, write, or run commands inside an **allowed** CorewareHub
repo. Grok Bot never edits the repo directly. It launches a Cursor Cloud Agent via
this skill's configured launcher settings and supervises it. Do not pin a specific
Composer model version unless Angelo says otherwise.

This skills repo (`elo-coreware/grok-bot-skills`) is **not** a product allow-list
target. Do not delegate product edits here.

## ALLOWED REPOS

| owner/repo | Base branch | DEV notes |
|------------|-------------|-----------|
| `CorewareHub/coreware-app-backend` | `develop` | Base remains `develop` |
| `CorewareHub/boss-control-tower` | `develop/develop` | DEV testing: `develop/<feature-slug>` on https://dev.coreware.app — never tip-push experiments onto `develop/develop` (main) |

Refuse any other owner/repo unless Angelo explicitly expands the allow-list. NASA CI
test-health work stays on `coreware-app-backend`. Grace, Raye, Gene/Wernher
orchestrating them, and feature engineers (Susan Kare, Jean Bartik) may be delegated
on either allowed product repo.

## REQUIRED INPUTS AND ACCESS

- A task brief, self-contained: the agent cannot see the Grok Bot conversation.
- **owner/repo** (required) — one of the allowed repos above.
- **Base branch** matching that repo (`develop` or `develop/develop`) and the exact
  target branch name. Do not default blindly to `develop` when the repo is
  boss-control-tower. For Control Tower DEV work, prefer `develop/<feature-slug>`.
- Cursor Cloud Agents access.
- gh authenticated for the chosen owner/repo.

## SEQUENCE OF WORK

1. Launch using this skill's configured Cursor Cloud Agent launcher settings (do not pin a specific Composer model version unless Angelo says otherwise). Verify the actually served model matches those settings. If the launcher substitutes another model or the served model is unexpected, STOP and report to Angelo.

2. Write the brief with all six sections in this order:

   GOAL — one sentence, the outcome.

   BRANCH — owner/repo, base branch, and the branch name to create. **Existing-branch mode:**
   when the work is remediation on a branch that already exists (Raye/Grace on an
   assigned feature PR, or Kare/Bartik continuing a plan/implement branch), state the
   existing branch to check out and say explicitly "do not create a new branch".
   Everything else in this skill applies unchanged.

   SCOPE — the explicit file list or phase table from the plan doc.

   CONSTRAINTS — paste verbatim: never commit on develop / develop/develop / main /
   master; never merge a PR; never run composer format; one test command at a time;
   follow .cursor/rules/codebase.mdc, .cursor/rules/test-isolation.mdc, and
   .cursor/rules/test-failure-triage.mdc when those files exist in the target repo;
   scaffolding failures: fix the test; contract failures: do not invert assertions to
   match a bug, escalate, write an ESCALATED plan row, leave the test red; do not
   patch `app/` unless Angelo assigned that specific bug. "No app changes" means
   escalate, not invert. Framework-semantics corrections (PR 5778) are allowed;
   product-behavior rewrites (PR 5785 class) are not.

   **Feature-PR babysit exception (Raye/Grace):** on a feature PR they were assigned,
   the `app/` restriction above is replaced by: edit only files already in
   `git diff <base>...HEAD --name-only` on that PR (`<base>` = the PR's base branch);
   new files or product-behavior changes beyond the committed implementation plan
   require Angelo's go-ahead. Every other constraint stands verbatim. Do not apply
   this exception to CI phase work — the NASA engineers' `app/` restriction is
   unchanged.

   **Feature engineer exception (Kare/Bartik):** on an assigned feature-plan-build or
   feature-implement branch, scope follows the committed plan. Product-behavior beyond
   the plan requires Angelo. Local Pint after PHP edits. Request the shared test slot
   before Pest.

   VERIFY — the exact verification commands to run, or `none` for read-only/planning work.
   When the delegation is Garman's, prefix verify with `TEST_TOKEN=9` (e.g.
   `TEST_TOKEN=9 composer test:single -- <paths>`).

   REPORT BACK — files changed, tests now passing, commands run with real output, anything unfixed and why, ESCALATED rows.

3. Launch one agent from the stated base branch using this skill's configured launcher settings.

4. Supervise until it finishes. **Concurrency (Angelo 2026-09-09 / standing rule 2026-09-11):** Exactly one agent
   may run **tests / migrate / schema dump** per database pair at a time. Margaret,
   Grace, Raye, Susan Kare, and Jean Bartik share token 1 (`test_tenant_1` /
   `test_landlord_1`); Gene or Wernher arbitrates (GRANTED / QUEUED / RELEASED) and
   Margaret takes precedence unless Angelo prioritizes the feature PR. Garman uses
   `TEST_TOKEN=9` for his DB pair but stays in the same shared slot — no parallel Pest
   until Angelo lifts the standing rule. The `scripts/test-lib.sh` ephemeral-sweep fix
   alone does not restore independence. Aaron, Bill, Raye, and Grace **may** launch
   additional agents **in parallel** only when VERIFY is `none` (read-only / planning /
   docs plan PRs / bugbot sweeps — no Pest, no migrate, no `test:generate-schema-dump`).
   Never launch a second test-running agent on the **same database pair** while another
   is live.

5. Capture the summary, branch name, and diff.

## HOW TO VALIDATE

- The run reports a serving model that matches this skill's launcher settings (not an unexpected substitute).
- owner/repo is on the allow-list and base matches that repo.
- The branch exists and is not develop / develop/develop / main / master. In create
  mode it came from the stated base; in existing-branch mode it is the branch named
  in BRANCH and no new branch was created. boss-control-tower DEV work should use
  `develop/<feature-slug>`, not tip-pushes onto `develop/develop`.
- Every touched file is inside SCOPE. Out-of-scope edits are a finding, not a bonus.
- No polarity inversion. No new application-behavior change unless Angelo already approved that specific patch.
- VERIFY output is quoted real output, not a claim that it passed.
- If VERIFY is not `none`, confirm no other test-running agent was live on the same database pair.

## WHAT TO RETURN

owner/repo, base branch, branch name, serving model, files changed with line counts,
verify output, unresolved items, ESCALATED rows, and any constraint the agent violated.

## WHAT REQUIRES APPROVAL

Pushing and opening a PR need no approval. Escalate to Angelo when the run is served by an unexpected model, or when the scope includes `app/` rather than tests/ without a written go-ahead (NASA track) or beyond the feature plan (feature track). Never merge a PR.
