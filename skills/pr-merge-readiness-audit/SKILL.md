---
name: pr-merge-readiness-audit
description: >-
  Use when pr-bugbot-sweep shows zero unfixed valid in-scope findings and Grace
  must verify all merge gates before plan retirement
---
# pr-merge-readiness-audit

## WHEN TO USE

pr-babysit-loop has zero unfixed valid in-scope items on current HEAD and Grace
needs to verify merge readiness before pr-plan-doc-retire and MERGE-READY verdict.

Gate 5 (implementation plan retired) is evaluated **after** gates 1–4 pass, by
pr-plan-doc-retire — not in this audit pass.

## REQUIRED INPUTS AND ACCESS

- PR number, branch name, current HEAD SHA.
- Latest combined findings ledger from pr-bugbot-sweep.
- gh authenticated for CorewareHub/coreware-app-backend.

## THE FIVE GATES

| Gate | Check | Pass criterion |
|------|-------|----------------|
| 1 | Local sweep | Zero unfixed valid in-scope findings on HEAD |
| 2 | cursor[bot] | Review exists with commit_id == HEAD; all valid in-scope threads addressed in code |
| 3 | CI | All required checks green on HEAD (`gh pr checks`) |
| 4 | Mergeability | mergeable == true, no conflicts with develop |
| 5 | Plan retired | pr-plan-doc-retire completed (run after 1–4) |

This skill evaluates gates **1–4 only**. Gate 5 is confirmed after pr-plan-doc-retire.

## SEQUENCE OF WORK

1. **Gate 1 — Local sweep.**
   - Re-run pr-bugbot-sweep or confirm the latest ledger on current HEAD.
   - PASS when valid + in-scope + Fixed?=no count is zero.

2. **Gate 2 — cursor[bot].**
   ```bash
   gh api repos/CorewareHub/coreware-app-backend/pulls/<PR>/reviews \
     --jq '.[] | select(.user.login=="cursor[bot]") | {commit_id, submitted_at, state}'
   ```
   - PASS when a review exists whose commit_id equals HEAD SHA.
   - For each valid in-scope cursor[bot] thread on that SHA, confirm the fix is
     present in HEAD diff or addressed in a remediation commit since that thread.
   - FAIL or WAITING when no review on HEAD, or valid in-scope items remain unfixed.

3. **Gate 3 — CI.**
   ```bash
   gh pr checks <PR> --repo CorewareHub/coreware-app-backend
   ```
   - PASS when all required checks are pass/success on HEAD.
   - WAITING when checks are pending.
   - FAIL when any required check failed.

4. **Gate 4 — Mergeability.**
   ```bash
   gh pr view <PR> --json mergeable,mergeStateStatus,headRefOid \
     --jq '{mergeable, mergeStateStatus, headRefOid}'
   ```
   - PASS when mergeable is true and mergeStateStatus is CLEAN or BEHIND (not
     CONFLICTING, not BLOCKED).
   - FAIL when conflicts exist — post BLOCKED, ask Angelo to resolve.

5. **Record reviewed SHA.** Save HEAD SHA as `reviewed-sha` for pr-plan-doc-retire
   and the MERGE-READY verdict (reviewed-sha vs post-retire HEAD).

## VERDICT MAPPING

| Gates 1–4 | Verdict for pr-merge-verdict-comment |
|-----------|--------------------------------------|
| All pass | Proceed to pr-plan-doc-retire, then MERGE-READY |
| Gate 2 pending review | WAITING |
| Gate 3 pending | WAITING |
| Gate 1, 2, or 4 fail | BLOCKED |
| Gate 3 fail (CI red) | BLOCKED |

## HOW TO VALIDATE

- Each gate has explicit pass/fail/waiting with quoted gh output.
- HEAD SHA recorded at audit time.
- Gate 5 not claimed pass until pr-plan-doc-retire runs.

## WHAT TO RETURN

Gate table (1–4 pass/fail/waiting), HEAD SHA (reviewed-sha), cursor[bot] review
SHA, CI summary, mergeStateStatus, blocker text if any.

## WHAT REQUIRES APPROVAL

Read-only gh queries need no approval. Never merge. Escalate merge conflicts to Angelo.
