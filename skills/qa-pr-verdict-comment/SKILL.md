---
name: qa-pr-verdict-comment
description: >-
  Use immediately after qa-fix-audit, a plan-validity qa-validity-scan, or
  qa-plan-retire-audit returns a verdict. Post a signed GitHub PR comment under
  Angelo's account. On phase PRs, mark ready only when PASS and cursor[bot]
  reviewed this SHA with no unfixed valid items.
---
# qa-pr-verdict-comment

## WHEN TO USE

Immediately after qa-fix-audit returns a verdict, after Katherine's qa-validity-scan on a new fix-plan PR, or after qa-plan-retire-audit. Post for every verdict, not only PASS — a silent FAIL is worse than a noisy one. Do not post a PASS that pretends Bugbot ran if it did not.

## REQUIRED INPUTS AND ACCESS

- PR number, branch name, and either phase number (phase PR), "plan" (new plan PR), or "plan-retire" (retire PR).
- The completed qa-fix-audit, plan-validity scan, or plan-retire audit output, including WAITING if Bugbot has not reviewed this SHA.
- Angelo's GitHub identity for posting (see IDENTITY).

## IDENTITY — READ THIS EVERY TIME

Every verdict comment must appear under Angelo's GitHub account (`elo-coreware`), never under `cursor[bot]` or any integration bot.

Preferred posting path (in order):
1. **GitHub MCP** `add_issue_comment` with `owner=CorewareHub`, `repo=coreware-app-backend`, `issue_number=<PR>`, `body=<verdict markdown>` — uses Angelo's connected GitHub account.
2. Else `gh pr comment <number> --repo CorewareHub/coreware-app-backend --body-file <path>` only when `gh auth status` shows Angelo / `elo-coreware`.

After posting, verify authorship of the newest issue comment. Accept only when `user.login` is `elo-coreware` (or Angelo's current login). If `gh`/MCP returns 403, is unauthenticated, or the comment lands as `cursor[bot]`, STOP. Do not treat a bot-authored comment as success. Do not fall back to the Cursor PR-management API or any integration token. Tell Gene Angelo must re-auth, and leave the PR draft if readiness depends on this comment.

Because the comment posts under his name: the signature block is mandatory, and never write a claim you cannot evidence. If unsure of a fact, omit it and state what you did not verify. A wrong claim under his name is worse than no comment.

## SEQUENCE OF WORK

1. Confirm the PR head branch matches the audited branch. If it moved, re-run the audit or scan first.
2. Fill the matching template below. Keep it to what a reviewer needs in 60 seconds.
3. Post via IDENTITY path above.
4. Verify the comment author is Angelo. If not, STOP per IDENTITY.
5. Ready for review:
   - **Phase PR:** `gh pr ready` only when verdict is PASS or PASS WITH NOTES **and** `cursor[bot]` reviewed this head SHA **and** no valid in-scope Bugbot item remains unfixed. WAITING, FAIL, or a stale Bugbot review (wrong `commit_id`) → leave draft.
   - **Plan PR:** `gh pr ready` on PASS or PASS WITH NOTES. If `cursor[bot]` already commented, the scan must have triaged those threads. Leave draft on FAIL.
   - **Plan-retire PR:** `gh pr ready` on PASS or PASS WITH NOTES. Docs-only; Bugbot only if `cursor[bot]` already commented. Leave draft on FAIL.
   - If it is already ready and the new verdict is FAIL or WAITING, do not convert it back unless `gh` supports converting to draft; report that to Gene. Never merge.
6. Report the comment URL, comment author login, and whether the PR is now ready to Gene.

## TEMPLATE — phase PR

```
## QA audit — Phase N: <slug>

**Verdict: PASS** (or PASS WITH NOTES / FAIL / WAITING)

A table with: failures resolved (X of Y listed), assertion count (before → after), test blocks (before → after, none removed), validity scan (BLOCKER/HIGH/MEDIUM counts), scope (only phase files touched), verify output (present and reproduced), Bugbot app review SHA vs HEAD.

**Root cause addressed:** one sentence, the actual cause not the symptom.

**How each fix was classified:** per file — test setup wrong, or app regression.

**Bugbot triage:** valid / false positive / out of scope / needs clarification, split by lowercase `/bugbot` vs GitHub `cursor[bot]`. Unfixed valid items mean FAIL.

**Anti-cheating checks:** state which checks were run and that they are clear — no tests deleted or skipped, no assertions removed or weakened, no assertOk downgrades, no driver detection, no Cache::flush, no runtime Schema:: DDL, no ->first() fixture selection, no tolerance loosened.

**Needs your eyes:** the one or two judgment calls a human should confirm, or "Nothing; the diff is mechanical." Never leave this empty.

---

*Audited by **Katherine** (QA validator bot) · model Composer 2.5 Fast · branch `fix/ci-tests-phase-N-slug` · posted via Angelo's account. Verdict is advisory; merge decision is Angelo's.*
```

## TEMPLATE — plan PR

```
## QA plan-validity — <plan filename>

**Verdict: PASS** (or PASS WITH NOTES / FAIL)

Authority: `.cursor/commands/automated-tests-validity-detection.md`

A table with: test files listed in the plan, files scanned, BLOCKER/HIGH/MEDIUM counts, files still valid to include as work, files that must be dropped or rewritten before they stay in the plan.

**Invalid tests the plan must not treat as work:** file:line, severity, snippet, concrete fix. Or "None."

**Needs your eyes:** the one or two judgment calls a human should confirm, or "Nothing; the listed tests are valid." Never leave this empty.

---

*Audited by **Katherine** (QA validator bot) · model Composer 2.5 Fast · branch `docs/YYYYMMDD-failing-tests-fix-plan` · posted via Angelo's account. Verdict is advisory; merge decision is Angelo's.*
```


## TEMPLATE — plan-retire PR

```
## QA plan-retire — <plan filename(s)>

**Verdict: PASS** (or PASS WITH NOTES / FAIL)

Diff is docs-only (plan delete ± `docs/automated-tests/README.md`).

A table with: plan path deleted, every phase COMPLETE or disposed (WONTFIX/ESCALATED + Angelo disposition), merged phase PR citations, open engineer work orphan risk (none).

**Needs your eyes:** the one or two judgment calls a human should confirm, or "Nothing; the plan is fully shipped and safe to delete." Never leave this empty.

---

*Audited by **Katherine** (QA validator bot) · model Composer 2.5 Fast · branch `docs/retire-…` · posted via Angelo's account. Verdict is advisory; merge decision is Angelo's.*
```

## HOW TO VALIDATE

- The verdict matches the audit or scan exactly. Never upgraded.
- Every number traces to the audit or scan output.
- "Needs your eyes" is populated.
- Signature block present, naming the bot and the model.
- Comment author is Angelo, not `cursor[bot]`.
- On a phase PR, ready only if both Katherine PASS and Bugbot-on-this-SHA are clean. On a plan or plan-retire PR, ready on PASS or PASS WITH NOTES.

## WHAT REQUIRES APPROVAL

Posting and marking ready need no approval **when the comment posts as Angelo**. Never merge the PR, never submit a GitHub review approval, never resolve someone else's review thread. Escalate to Gene/Angelo when a verdict cannot be posted under Angelo's account.
