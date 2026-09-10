---
name: pr-merge-verdict-comment
description: >-
  Use after pr-plan-doc-retire (or when gates fail) to post MERGE-READY, BLOCKED,
  or WAITING verdict on the feature PR under Angelo's account
---
# pr-merge-verdict-comment

## WHEN TO USE

- **MERGE-READY:** pr-plan-doc-retire completed (or gate 5 N/A), CI green on final-sha.
- **BLOCKED:** pr-merge-readiness-audit gate 1, 2, or 4 failed; CI failed; scope
  escalation; deletion-only assertion failed.
- **WAITING:** cursor[bot] has not reviewed HEAD; CI checks pending.

Post for every verdict, not only MERGE-READY — a silent BLOCKED is worse than a
noisy one.

## REQUIRED INPUTS AND ACCESS

- PR number, branch name, reviewed-sha, final-sha (same as reviewed-sha if no plan
  retirement commit).
- Gate table from pr-merge-readiness-audit.
- Combined findings ledger from pr-bugbot-sweep.
- Verdict: MERGE-READY, BLOCKED, or WAITING.

## IDENTITY — READ THIS EVERY TIME

Every verdict comment must appear under Angelo's GitHub account (`elo-coreware`),
never under `cursor[bot]` or any integration bot.

Preferred posting path (in order):
1. **GitHub MCP** `add_issue_comment` with `owner=CorewareHub`,
   `repo=coreware-app-backend`, `issue_number=<PR>`, `body=<verdict markdown>`.
2. Else `gh pr comment <number> --repo CorewareHub/coreware-app-backend --body-file
   <path>` only when `gh auth status` shows Angelo / `elo-coreware`.

After posting, verify authorship of the newest issue comment. Accept only when
`user.login` is `elo-coreware` (or Angelo's current login). If `gh`/MCP returns
403, is unauthenticated, or the comment lands as `cursor[bot]`, STOP. Do not treat
a bot-authored comment as success. Never fall back to the Cursor PR-management API
or any integration token that posts as `cursor[bot]`. Tell Gene Angelo must re-auth.

Because the comment posts under his name: the signature block is mandatory, and
never write a claim you cannot evidence.

## SEQUENCE OF WORK

1. Confirm PR head branch matches the audited branch. If it moved unexpectedly,
   re-run pr-merge-readiness-audit first.

2. Fill the template below for the verdict type.

3. Post via IDENTITY path above.

4. Verify comment author is Angelo. If not, STOP per IDENTITY.

5. Notify Gene and Angelo with the comment URL. Never merge. Never `gh pr ready`
   unless Angelo explicitly asked Grace to mark ready (default: leave as-is).

## TEMPLATE — MERGE-READY

```
## PR readiness — <branch>

**Verdict: MERGE-READY**

| Gate | Status |
|------|--------|
| 1 Local sweep (zero unfixed valid in-scope) | pass |
| 2 cursor[bot] on reviewed-sha | pass |
| 3 CI checks (green on final-sha) | pass |
| 4 Mergeable with develop | pass |
| 5 Implementation plan retired | pass / N/A |

**SHAs:** reviewed `<reviewed-sha>` → final `<final-sha>`

Gate 2 is asserted against `reviewed-sha`, not `final-sha`. When a plan retirement
commit moved HEAD, state the deletion-only delta explicitly:
`git diff <reviewed-sha>..<final-sha> --name-only` returned only the plan path, so
no new Bugbot review was required. CI is green on `final-sha`.

**Merge state:** `mergeable` / `mergeStateStatus` / draft — call out `BLOCKED`
(needs your approving review), `UNSTABLE`, or `isDraft: true` (you must mark the
PR ready) when present.

**Findings ledger:** valid fixed (N), dismissed false positive (N), out of scope (N).
Split by source: local /bugbot vs cursor[bot]. List any fix commits with SHAs.

**Plan archive:** comment URL or "no plan on branch"

**Needs your eyes:** the one or two judgment calls a human should confirm before
merge, or "Nothing; safe to merge after your review." Never leave this empty.

---

*Audited by **Grace** (PR readiness bot) · model Composer 2.5 Fast · branch `<branch>`
· posted via Angelo's account. Verdict is advisory; merge decision is Angelo's.*
```

## TEMPLATE — BLOCKED

```
## PR readiness — <branch>

**Verdict: BLOCKED**

**Blocker:** one sentence — which gate failed and why.

| Gate | Status |
|------|--------|
| 1–5 | pass / fail / waiting / N/A |

**Findings still open:** list valid in-scope items or "none — blocked on CI/conflicts"

**Next action:** what Grace or Angelo must do.

**Needs your eyes:** never empty.

---

*Audited by **Grace** (PR readiness bot) · model Composer 2.5 Fast · branch `<branch>`
· posted via Angelo's account. Verdict is advisory; merge decision is Angelo's.*
```

## TEMPLATE — WAITING

```
## PR readiness — <branch>

**Verdict: WAITING**

**Waiting on:** cursor[bot] review on HEAD / CI checks / test slot from Gene

**HEAD SHA:** `<sha>`

**Needs your eyes:** "None while waiting" or specific concern.

---

*Audited by **Grace** (PR readiness bot) · model Composer 2.5 Fast · branch `<branch>`
· posted via Angelo's account. Verdict is advisory; merge decision is Angelo's.*
```

## HOW TO VALIDATE

- Verdict matches audit outcome exactly. Never upgraded to MERGE-READY to unblock.
- Gate table traces to pr-merge-readiness-audit and pr-plan-doc-retire output.
- reviewed-sha and final-sha stated on MERGE-READY.
- Gate 2 labelled against reviewed-sha, never claimed as "on HEAD" once a plan
  retirement commit has moved HEAD.
- Draft status surfaced — MERGE-READY never implies a draft is mergeable as-is.
- "Needs your eyes" populated.
- Signature block present.
- Comment author is Angelo, not cursor[bot].

## WHAT TO RETURN

Comment URL, comment author login, verdict, reviewed-sha, final-sha.

## WHAT REQUIRES APPROVAL

Posting needs no approval when posting as Angelo. Never merge. Never submit GitHub
review approval. Never resolve cursor[bot] threads.
