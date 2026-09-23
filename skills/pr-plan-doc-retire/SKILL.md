---
name: pr-plan-doc-retire
description: >-
  Use after pr-merge-readiness-audit gates 1–4 pass to archive and remove the
  committed implementation plan before MERGE-READY
---
# pr-plan-doc-retire

## WHEN TO USE

pr-merge-readiness-audit gates 1–4 are green on HEAD. Raye or Grace must retire the
implementation plan file so it is not merged into `<base>` (repo bloat), while
preserving its content in PR history.

If no implementation plan exists on the branch, skip steps 2–4 and confirm gate 5
as N/A — proceed directly to pr-merge-verdict-comment MERGE-READY.

## REQUIRED INPUTS AND ACCESS

- owner/repo + base (`develop` for coreware-app-backend, or `develop/develop` for
  boss-control-tower), PR number, branch name, reviewed-sha from
  pr-merge-readiness-audit (HEAD when gates 1–4 passed).
- **DEV branch rule (boss-control-tower):** Tips default to `feature/<name>` or `fix/<name>`; use `develop/<feature-name>` only for visual confirmation on https://dev.coreware.app. Never tip-push experiments onto `develop/develop`
  (main). PROD web hosts — observe / peek only (Angelo 2026-09-18) for BOTH https://controltower.coreware.app (landlord Control Tower PRODUCTION) and https://coreware.coreware.app (backend/tenant PRODUCTION): bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Backend base remains `develop`.
- gh / GitHub MCP for PR comments and body edit.
- repo-delegate-to-cursor for git rm, commit, push.

Plan identification — same rules as `.cursor/commands/git-commit.md` step 5:
1. Plan assigned or attached for this PR in Raye or Grace's context.
2. Else `.cursor/plans/*.plan.md` modified since merge-base with `<base>`.
3. Exclude all BugBot-generated plans (filename, frontmatter, body markers per
   git-commit.md step 5b).

## SEQUENCE OF WORK

1. **Identify plan.**
   ```bash
   git ls-files .cursor/plans/
   git merge-base <base> HEAD
   ```
   If multiple non-BugBot candidates, list them and ask Angelo which one — do not
   guess. If none tracked, gate 5 = N/A, skip to MERGE-READY verdict.

2. **Archive as PR comment.** Post the plan's full file content under Angelo's account:
   ```markdown
   ## Implementation plan (archived before merge)

   <full contents of .cursor/plans/<file>.plan.md>
   ```
   Use GitHub MCP `add_issue_comment`. Verify author is `elo-coreware`.

3. **Remove from branch.** Delegate repo-delegate-to-cursor:
   ```bash
   git rm .cursor/plans/<plan-file>.plan.md
   git commit -m ":memo: remove implementation plan before merge" \
     -m "Archive preserved in PR comment. Plan must not land on <base>."
   git push origin HEAD
   ```

4. **Strip PR body section.** Remove the `## Implementation plan` section from the
   PR description:
   ```bash
   gh pr view <PR> --repo <owner>/<repo> --json body --jq .body
   gh pr edit <PR> --repo <owner>/<repo> --body "<updated body without ## Implementation plan section>"
   ```
   Preserve `## Summary` and `## Test plan` sections intact.

5. **Deletion-only delta assertion.**
   ```bash
   git diff <reviewed-sha>..HEAD --name-only
   ```
   - PASS when the diff contains **only** the plan file path (deletion).
   - If other files changed, FAIL — investigate before MERGE-READY.
   - Record post-retire HEAD SHA as `final-sha`.

6. **Final-sha verify (Angelo standing rule 2026-09-21) — do NOT wait on full Tests.**
   A deletion-only plan-retire commit does **not** require a new Bugbot sweep and
   does **not** require the full self-hosted `Tests` suite green on final-sha.
   Do **not** poll `gh pr checks` for ~60 min as a MERGE-READY gate.

   Keep as applicable:
   - Bugbot CLEAN still asserted against `reviewed-sha` (gates 1–2); deletion-only
     delta means no new Bugbot on final-sha is required.
   - Pint/lint: N/A for docs-only deletion; if any PHP slipped into the retire
     commit, run full-repo local Pint (`./vendor/bin/pint` then `./vendor/bin/pint --test`; match Check Code Style) before MERGE-READY.
   - Touched tests: N/A for deletion-only plan retire.

   Ambient full-suite **Tests** red/pending on final-sha is OK unless tip-caused; full-repo **Pint** red is not.
   Optional informational `gh pr checks` note is fine; it is not a hard gate.

## RULES

- Never delete the plan without posting the archive comment first.
- Never merge the PR.
- MERGE-READY verdict must state both reviewed-sha and final-sha explicitly.

## HOW TO VALIDATE

- Archive comment exists under elo-coreware with full plan content.
- Plan file absent from `git ls-files` on final-sha.
- PR body no longer contains `## Implementation plan`.
- `git diff reviewed-sha..final-sha --name-only` shows only the plan path.
- Full self-hosted Tests green on final-sha is **not** required (Angelo 2026-09-21);
  ambient CI debt OK unless tip-caused. Bugbot/lint as applicable only.

## WHAT TO RETURN

Plan path (or "none"), archive comment URL, reviewed-sha, final-sha, optional ambient
CI note (not a gate), deletion-only assertion result.

## WHAT REQUIRES APPROVAL

Push of plan removal commit needs no approval when posting as Angelo. Escalate if
deletion-only assertion fails or tip-caused Pint/touched-test failure on final-sha.
