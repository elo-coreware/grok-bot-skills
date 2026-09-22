---
name: pr-merge-readiness-audit
description: >-
  Use when pr-bugbot-sweep shows zero unfixed valid in-scope findings and Raye or
  Grace must verify all merge gates before plan retirement
---
# pr-merge-readiness-audit

## WHEN TO USE

pr-babysit-loop has zero unfixed valid in-scope items on current HEAD and Raye or
Grace needs to verify merge readiness before pr-plan-doc-retire and MERGE-READY
verdict.

Gate 5 (implementation plan retired) is evaluated **after** gates 1–4 pass, by
pr-plan-doc-retire — not in this audit pass.

## REQUIRED INPUTS AND ACCESS

- owner/repo + base (`develop` for coreware-app-backend, or `develop/develop` for
  boss-control-tower), PR number, branch name, current HEAD SHA.
- **DEV branch rule (boss-control-tower):** Tips default to `feature/<name>` or `fix/<name>`; use `develop/<feature-name>` only for visual confirmation on https://dev.coreware.app. Never tip-push experiments onto `develop/develop`
  (main). PROD web hosts — observe / peek only (Angelo 2026-09-18) for BOTH https://controltower.coreware.app (landlord Control Tower PRODUCTION) and https://coreware.coreware.app (backend/tenant PRODUCTION): bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Backend base remains `develop`.
- Latest combined findings ledger from pr-bugbot-sweep.
- gh authenticated for the assigned owner/repo (coreware-app-backend or
  boss-control-tower).

## THE FIVE GATES

| Gate | Check | Pass criterion |
|------|-------|----------------|
| 1 | Local sweep | Zero unfixed valid in-scope findings on HEAD |
| 2 | cursor[bot] | Review exists with commit_id == HEAD; all valid in-scope threads addressed in code |
| 3 | Verify (touched tests + full Pint) | Bugbot CLEAN==HEAD already gated in 1–2; **full-repo** Pint/lint green (`pint --test` or Check Code Style `lint (8.3)` green on HEAD); touched tests pass under Gene Pest GRANT on shared machine. Full self-hosted Tests suite green is **not** required. Ambient full-suite **Tests** red OK unless tip-caused; full-repo **Pint** red is always a Gate 3 fail |
| 4 | Mergeability | mergeable == "MERGEABLE", mergeStateStatus != "DIRTY", not a draft |
| 5 | Plan retired | pr-plan-doc-retire completed (run after 1–4) |

This skill evaluates gates **1–4 only**. Gate 5 is confirmed after pr-plan-doc-retire.

## SEQUENCE OF WORK

1. **Gate 1 — Local sweep.**
   - Re-run pr-bugbot-sweep or confirm the latest ledger on current HEAD.
   - PASS when valid + in-scope + Fixed?=no count is zero.

2. **Gate 2 — cursor[bot].**
   ```bash
   gh api repos/<owner>/<repo>/pulls/<PR>/reviews \
     --jq '.[] | select(.user.login=="cursor[bot]") | {commit_id, submitted_at, state}'
   ```
   - PASS when a review exists whose commit_id equals HEAD SHA.
   - For each valid in-scope cursor[bot] thread on that SHA, confirm the fix is
     present in HEAD diff or addressed in a remediation commit since that thread.
   - FAIL or WAITING when no review on HEAD, or valid in-scope items remain unfixed.

3. **Gate 3 — Verify (touched tests + full-repo Pint/lint; Angelo 2026-09-23).**
   Do **not** require the full self-hosted `Tests` / Run Tests suite green.
   Do **not** treat ambient full-suite red or pending as FAIL/WAITING.

   Verify on the shared machine under Gene Pest GRANT:
   ```bash
   # after GRANTED — touched paths only
   composer test:single -- <touched test paths>
   ./vendor/bin/pint && ./vendor/bin/pint --test   # FULL tree — never --dirty / path-scoped-only
   # or confirm Actions Check Code Style / lint (8.3) green on HEAD
   ```
   Optional read of `gh pr checks` is informational only (note ambient debt);
   it is **not** a pass/fail gate for MERGE-READY.
   - PASS when touched tests pass under Pest GRANT and **full-repo** Pint/lint
     is green (`pint --test` exit 0 or Check Code Style green on HEAD; N/A only
     if already green and no PHP under Pint paths). Bugbot CLEAN==HEAD is gates 1–2.
   - WAITING when Pest GRANT not yet available (or Bugbot still pending — gate 2).
   - FAIL when touched tests fail tip-caused, **or** full-repo Pint/`lint (8.3)` is red on HEAD (always tip-blocking for style).
   - Ambient full-suite self-hosted Tests red/pending → PASS WITH NOTE (debt OK).

4. **Gate 4 — Mergeability.**
   ```bash
   gh pr view <PR> --repo <owner>/<repo> --json mergeable,mergeStateStatus,isDraft,headRefOid \
     --jq '{mergeable, mergeStateStatus, isDraft, headRefOid}'
   ```

   These are two different GraphQL enums. Do not conflate them, and never treat
   `mergeable` as a boolean — it is a string.

   | Field | Enum | Values |
   |-------|------|--------|
   | `mergeable` | MergeableState | `MERGEABLE`, `CONFLICTING`, `UNKNOWN` |
   | `mergeStateStatus` | MergeStateStatus | `CLEAN`, `BEHIND`, `HAS_HOOKS`, `UNSTABLE`, `BLOCKED`, `DIRTY`, `UNKNOWN` |

   - **PASS** when `mergeable == "MERGEABLE"` and `mergeStateStatus` is not `DIRTY`.
     `CLEAN`, `BEHIND`, and `HAS_HOOKS` are all genuinely mergeable.
   - **FAIL (BLOCKED verdict)** when `mergeable == "CONFLICTING"` or
     `mergeStateStatus == "DIRTY"` — real conflicts with `<base>`. Ask Angelo to
     resolve; never resolve conflicts yourself.
   - **WAITING** when `mergeable == "UNKNOWN"` — GitHub is still computing
     mergeability. Re-poll; do not report a verdict off an UNKNOWN.
   - **PASS WITH NOTE, do not fail**, on these — they are not Raye/Grace defects:
     - `mergeStateStatus == "UNSTABLE"` — mergeable but commit status not passing.
       Gate 3 owns touched-test + full-repo Pint verify (not full Tests suite); ambient Tests red is OK; ambient/full Pint red is NOT OK.
     - `mergeStateStatus == "BLOCKED"` — branch protection (e.g. a required
       approving review). Surface it in the verdict so Angelo knows he must
       approve before merging.
   - **`isDraft == true`** — a draft cannot be merged. `mergeStateStatus` still
     reports `CLEAN` for drafts, so this must be checked separately. Never post
     MERGE-READY on a draft: surface it and tell Angelo to mark the PR ready
     (Raye/Grace does not run `gh pr ready` unless Angelo asked them to).

5. **Record reviewed SHA.** Save HEAD SHA as `reviewed-sha` for pr-plan-doc-retire
   and the MERGE-READY verdict (reviewed-sha vs post-retire HEAD).

## VERDICT MAPPING

| Gates 1–4 | Verdict for pr-merge-verdict-comment |
|-----------|--------------------------------------|
| All pass | Proceed to pr-plan-doc-retire, then MERGE-READY |
| Gate 2 pending review | WAITING |
| Gate 3 waiting Pest GRANT | WAITING |
| Gate 4 mergeable == UNKNOWN | WAITING |
| Gate 1, 2, or 4 fail | BLOCKED |
| Gate 3 fail (tip-caused touched-test / full-repo Pint red) | BLOCKED |
| Ambient full-suite CI red/pending | Not a blocker — PASS WITH NOTE |
| Gate 4 pass but isDraft, BLOCKED, or UNSTABLE | Proceed, but state it in the verdict |

## HOW TO VALIDATE

- Each gate has explicit pass/fail/waiting with quoted gh output.
- HEAD SHA recorded at audit time.
- Gate 5 not claimed pass until pr-plan-doc-retire runs.
- `mergeable` compared against the string `"MERGEABLE"`, never treated as boolean.
- `isDraft` checked separately — never inferred from `mergeStateStatus`.

## WHAT TO RETURN

Gate table (1–4 pass/fail/waiting), HEAD SHA (reviewed-sha), cursor[bot] review
SHA, touched-test/Pint summary (and optional ambient CI note), mergeable, mergeStateStatus, isDraft, blocker text if any.

## WHAT REQUIRES APPROVAL

Read-only gh queries need no approval. Never merge. Escalate merge conflicts to Angelo.
