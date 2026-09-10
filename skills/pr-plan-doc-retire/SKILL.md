---
name: pr-plan-doc-retire
description: >-
  Use after pr-merge-readiness-audit gates 1–4 pass to archive and remove the
  committed implementation plan before MERGE-READY
---
# pr-plan-doc-retire

## WHEN TO USE

pr-merge-readiness-audit gates 1–4 are green on HEAD. Grace must retire the
implementation plan file so it is not merged into develop (repo bloat), while
preserving its content in PR history.

If no implementation plan exists on the branch, skip steps 2–4 and confirm gate 5
as N/A — proceed directly to pr-merge-verdict-comment MERGE-READY.

## REQUIRED INPUTS AND ACCESS

- PR number, branch name, reviewed-sha from pr-merge-readiness-audit (HEAD when
  gates 1–4 passed).
- gh / GitHub MCP for PR comments and body edit.
- repo-delegate-to-cursor for git rm, commit, push.

Plan identification — same rules as `.cursor/commands/git-commit.md` step 5:
1. Plan assigned or attached for this PR in Grace's context.
2. Else `.cursor/plans/*.plan.md` modified since merge-base with develop.
3. Exclude all BugBot-generated plans (filename, frontmatter, body markers per
   git-commit.md step 5b).

## SEQUENCE OF WORK

1. **Identify plan.**
   ```bash
   git ls-files .cursor/plans/
   git merge-base develop HEAD
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
     -m "Archive preserved in PR comment. Plan must not land on develop."
   git push origin HEAD
   ```

4. **Strip PR body section.** Remove the `## Implementation plan` section from the
   PR description:
   ```bash
   gh pr view <PR> --json body --jq .body
   gh pr edit <PR> --body "<updated body without ## Implementation plan section>"
   ```
   Preserve `## Summary` and `## Test plan` sections intact.

5. **Deletion-only delta assertion.**
   ```bash
   git diff <reviewed-sha>..HEAD --name-only
   ```
   - PASS when the diff contains **only** the plan file path (deletion).
   - If other files changed, FAIL — investigate before MERGE-READY.
   - Record post-retire HEAD SHA as `final-sha`.

6. **Re-check CI on final-sha.**
   ```bash
   gh pr checks <PR>
   ```
   All required checks must be green on final-sha before MERGE-READY. A
   deletion-only commit does not require a new Bugbot sweep, but CI must pass.

## RULES

- Never delete the plan without posting the archive comment first.
- Never merge the PR.
- MERGE-READY verdict must state both reviewed-sha and final-sha explicitly.

## HOW TO VALIDATE

- Archive comment exists under elo-coreware with full plan content.
- Plan file absent from `git ls-files` on final-sha.
- PR body no longer contains `## Implementation plan`.
- `git diff reviewed-sha..final-sha --name-only` shows only the plan path.
- CI green on final-sha.

## WHAT TO RETURN

Plan path (or "none"), archive comment URL, reviewed-sha, final-sha, CI status on
final-sha, deletion-only assertion result.

## WHAT REQUIRES APPROVAL

Push of plan removal commit needs no approval when posting as Angelo. Escalate if
deletion-only assertion fails or CI fails on final-sha.
