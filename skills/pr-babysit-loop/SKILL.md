---
name: pr-babysit-loop
description: >-
  Use when Gene or Angelo assigns Grace one feature PR to babysit until MERGE-READY
---
# pr-babysit-loop

## WHEN TO USE

Gene or Angelo assigns Grace exactly one feature PR (not a CI phase PR, not a
docs/*-FAILING-TESTS-FIX-PLAN.markdown plan PR). Grace runs this loop until the PR
is MERGE-READY or she hands it back with BLOCKED and a written reason.

## REQUIRED INPUTS AND ACCESS

- PR number and URL, branch name, head SHA at assignment.
- Read/write on the PR branch via repo-delegate-to-cursor.
- gh authenticated for CorewareHub/coreware-app-backend.
- Committed implementation plan on the branch when one exists (identification rules
  from .cursor/commands/git-commit.md step 5; BugBot-generated plans excluded).

## STATES

| State | Meaning |
|-------|---------|
| SWEEPING | Running pr-bugbot-sweep on current HEAD |
| REMEDIATING | Implementing valid in-scope findings (pr-finding-remediate) |
| WAITING-BUGBOT | Pushed a fix commit; waiting for cursor[bot] review on new SHA |
| WAITING-CI | Findings clean; waiting for gh pr checks on HEAD |
| BLOCKED | Cannot proceed without Angelo (conflicts, scope, slot denied, escalation) |
| READY | All five merge gates green; plan retired; MERGE-READY verdict posted |

Exactly one PR at a time. Never start a second PR until the first is MERGE-READY or
explicitly handed back to Gene/Angelo.

## SEQUENCE OF WORK

1. **Intake.** Record PR number, branch, initial HEAD SHA. Fetch origin. Confirm PR
   is open and targets develop. If merge conflicts with develop exist before work
   starts, post BLOCKED verdict and ask Angelo to resolve — do not merge develop
   yourself unless Gene or Angelo explicitly assigns conflict resolution.

2. **SWEEPING.** Run pr-bugbot-sweep. Produce the combined findings ledger (local
   /bugbot + /bugbot-triage + cursor[bot] HEAD threads).

3. **Decision.** If the ledger has unfixed valid in-scope items → REMEDIATING.
   If zero unfixed valid in-scope items → pr-merge-readiness-audit.

4. **REMEDIATING.** For each valid in-scope item (Critical first, then High, then
   Medium), run pr-finding-remediate. If verification needs tests, message Gene for
   the test slot. Do not run composer test:single until Gene grants GRANTED. If
   QUEUED, wait. Batch related fixes into one commit when sensible. After push,
   comment `cursor review` as Angelo → WAITING-BUGBOT.

5. **WAITING-BUGBOT.** Poll until cursor[bot] has a review whose commit_id equals
   HEAD, or timeout and report WAITING verdict to Gene. When review lands, return
   to SWEEPING (re-triage GitHub threads + optional fresh local sweep if findings
   changed materially).

6. **Merge readiness.** When sweep shows zero unfixed valid in-scope items, run
   pr-merge-readiness-audit (gates 1–4). Route on the outcome:
   - Gate 1 or 2 fail → back to SWEEPING or REMEDIATING.
   - Gate 3 pending, or gate 4 `mergeable == "UNKNOWN"` → WAITING-CI.
   - Gate 3 red or gate 4 conflicts → post BLOCKED.
   - Gates 1–4 pass → pr-plan-doc-retire → pr-merge-verdict-comment MERGE-READY →
     notify Gene and Angelo.

7. **WAITING-CI.** Poll `gh pr checks <PR>` until every required check resolves on
   HEAD, or GitHub finishes computing mergeability. Then re-run
   pr-merge-readiness-audit — do not skip straight to a verdict off a stale gate
   table. On red CI, post BLOCKED. If checks stay pending beyond one hour, report
   WAITING to Gene rather than polling silently.

   The plan retirement commit in step 6 starts a fresh CI run on the new HEAD.
   Re-enter WAITING-CI for that run before posting MERGE-READY.

8. **Handback.** On BLOCKED, post verdict, tell Gene the blocker, release the test
   slot if held, and stop. Angelo or Gene must re-assign to resume.

## HOW TO VALIDATE

- Only one PR active in Grace's state at any time.
- Every fix commit followed by `cursor review` on that SHA.
- Test runs only after Gene grants the slot.
- MERGE-READY never posted before pr-plan-doc-retire completes (when a plan existed).
- Never merged the PR.

## WHAT TO RETURN

State, PR number, branch, current HEAD SHA, findings ledger summary (valid remaining /
fixed / dismissed), gate table, verdict comment URL, blocker if BLOCKED.

## WHAT REQUIRES APPROVAL

Pushing fix commits and posting comments need no approval when posting as Angelo.
Never merge. Escalate to Angelo for scope beyond the implementation plan, unresolvable
conflicts, or repeated remediation failure on the same finding.
