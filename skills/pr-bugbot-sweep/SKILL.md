---
name: pr-bugbot-sweep
description: >-
  Use during pr-babysit-loop to detect and triage bugs from local BugBot commands
  and GitHub cursor[bot] on the PR head SHA
---
# pr-bugbot-sweep

## WHEN TO USE

Every SWEEPING pass in pr-babysit-loop, and after each remediation push once
cursor[bot] has reviewed the new SHA.

## REQUIRED INPUTS AND ACCESS

- PR number, branch name, current HEAD SHA.
- Committed implementation plan path on the branch (if any), per git-commit.md
  step 5 identification rules. Supply it to the delegated agent as the
  Implementation Plan input for /bugbot.
- repo-delegate-to-cursor for read-only BugBot runs (plan mode only — no fixes).
- gh / GitHub MCP for cursor[bot] review threads on this PR.

Authority files:
- `.cursor/commands/bugbot.md` (lowercase only, never .cursor/BUGBOT.md)
- `.cursor/commands/bugbot-triage.md`

## SEQUENCE OF WORK

1. **Branch scope.** At the PR head:
   ```bash
   git fetch origin develop 2>/dev/null || true
   git checkout <branch>
   git merge-base develop HEAD
   git diff develop...HEAD --name-status
   ```

2. **Local BugBot (delegated).** Launch repo-delegate-to-cursor with:
   - GOAL: Run .cursor/commands/bugbot.md in plan mode on develop...HEAD. Attach
     the committed implementation plan at `<path>` as the Implementation Plan.
     Do not implement fixes.
   - VERIFY: none (read-only plan output)
   - REPORT BACK: full Bug Report plan output

3. **Local triage (delegated).** Launch repo-delegate-to-cursor with:
   - GOAL: Run .cursor/commands/bugbot-triage.md on the Bug Report from step 2.
     Plan mode only. Do not implement fixes.
   - VERIFY: none
   - REPORT BACK: triage table with verdict per item

4. **GitHub cursor[bot].** Fetch review threads on this PR where the review author's
   login is `cursor[bot]` and `commit_id` equals current HEAD SHA. Include unresolved
   and resolved threads on that SHA. Stale reviews (different commit_id) are noted
   but not triaged as open work unless the finding may still apply to HEAD hunks.

5. **Merge triage.** Feed GitHub cursor[bot] thread bodies into the same triage
   classification as step 3 (valid / false positive / out of scope / needs
   clarification). Grace may run a second delegated triage pass with the combined
   list, or triage GitHub items herself using bugbot-triage.md rules — never skip
   scope gate per item.

6. **Ledger.** Produce one combined ledger:

   | # | Source | Finding | Scope | Verdict | Fixed? |
   |---|--------|---------|-------|---------|--------|
   | 1 | local /bugbot | ... | In hunk | Valid | no |
   | 2 | cursor[bot] | ... | In hunk | False positive | n/a |

   Split sources: `local /bugbot` vs `cursor[bot]`. Only **Valid + in-scope + Fixed?=no**
   items enter the remediation queue.

## RULES

- Never resolve a cursor[bot] review thread on GitHub. Reply in a PR comment if
  needed after fix; let Angelo or the app refresh threads.
- Never implement fixes in this skill.
- Never treat untriaged BugBot output as actionable.
- BugBot-generated `.cursor/plans/*.plan.md` files are never the Implementation Plan.

## HOW TO VALIDATE

- develop...HEAD three-dot diff used throughout.
- Both local bugbot and bugbot-triage ran via delegation on Composer 2.5 Fast.
- Every ledger row has file:line, scope evidence, and verdict.
- GitHub threads filtered to HEAD commit_id for open items.

## WHAT TO RETURN

Combined ledger, counts (valid unfixed / valid fixed / false positive / out of scope /
needs clarification), HEAD SHA, implementation plan path or "none".

## WHAT REQUIRES APPROVAL

Read-only delegation needs no approval. Escalate when the delegated agent served a
model other than Composer 2.5 Fast.
