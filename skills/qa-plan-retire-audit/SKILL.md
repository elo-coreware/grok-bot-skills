---
name: qa-plan-retire-audit
description: >-
  Use when Gene assigns Katherine to audit a docs-only plan-retire PR — confirm
  the plan is fully done on develop and safe to delete before ready.
---
# QA plan-retire audit

## WHEN TO USE

Gene assigns a plan-retire PR (Aaron or Bill authored) for Katherine's audit. The PR deletes one or more finished QA plan documents under `docs/` or `docs/automated-tests/`. This is not phase audit (`qa-fix-audit`) and not new-plan validity (`qa-validity-scan` mode b/d).

## REQUIRED INPUTS AND ACCESS

- PR number, branch, plan path(s) being deleted.
- Read access at PR head and on `origin/develop`.
- Never run test commands. Never implement.
- Post verdict via `qa-pr-verdict-comment` (plan-retire template) under Angelo's account.

## SEQUENCE OF WORK

1. Confirm PR is docs-only: `git diff origin/develop...HEAD --name-only` lists only plan markdown under `docs/` / `docs/automated-tests/` (± `docs/automated-tests/README.md`). Any `tests/` or `app/` → **FAIL**.
2. For each deleted plan file, recover content from the parent commit / develop tip before delete and read the phase summary table.
3. **Done-ness checks (binding):**
   - Every phase is COMPLETE, or WONTFIX/ESCALATED with explicit Angelo disposition recorded in the plan or linked PR comment.
   - No OPEN / PARTIAL implementable phases remain without disposition.
   - Cite merged phase PR numbers (or Gene waiver) covering the work the plan authorized.
   - Deleting the plan would not orphan in-flight engineer work (no open phase PR still referencing it as the active plan unless Gene says the retire is after that stack).
4. Confirm the retire PR body lists the same files `git rm`'d and a completeness table. Missing evidence → **FAIL** (return to author).
5. Bugbot: plan-retire PRs are docs-only — not blocked on Bugbot unless `cursor[bot]` already commented; then triage those threads.
6. Verdict: PASS, PASS WITH NOTES, or FAIL. Post via `qa-pr-verdict-comment`. Mark ready only on PASS / PASS WITH NOTES. Leave draft on FAIL. Never merge.

## VERDICT CRITERIA

| Verdict | When |
|---------|------|
| PASS | Docs-only delete; all phases COMPLETE (or disposed); merged PR citations present |
| PASS WITH NOTES | Done, but minor doc hygiene (README wording, leftover Supersedes links elsewhere) |
| FAIL | Open phases remain; missing Angelo disposition; non-docs files in diff; wrong file deleted |

## HOW TO VALIDATE

- Diff checked against develop.
- Phase table checked against develop history / merged PRs, not author claims alone.
- Verdict posted as `elo-coreware`.

## WHAT TO RETURN

Verdict, comment URL, ready-vs-draft, per-plan completeness table, any FAIL reasons for the author.

## WHAT REQUIRES APPROVAL

Posting and marking ready need no approval. Never merge. Escalate to Gene/Angelo when disposition of ESCALATED rows is ambiguous.
