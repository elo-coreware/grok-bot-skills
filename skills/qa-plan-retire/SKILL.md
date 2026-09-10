---
name: qa-plan-retire
description: >-
  Use when Gene assigns Aaron or Bill to open a docs-only PR that deletes (or
  moves) a fully completed CI fix, coverage, or hygiene plan after all phases
  shipped.
---
# QA plan retire

## WHEN TO USE

Gene assigns Aaron (fix plans) or Bill (coverage / hygiene plans) to retire a plan document that is **fully implemented** on `develop`. Retire means a dedicated docs-only PR that removes the finished plan from the tree (or relocates it under `docs/automated-tests/archive/` only when Gene/Angelo explicitly ask to archive instead of delete).

Do **not** use for Grace's feature PR `.cursor/plans/*.plan.md` — that is `pr-plan-doc-retire`.

## REQUIRED INPUTS AND ACCESS

- Plan path (e.g. `docs/20260909-forms-TEST-HYGIENE-PLAN.markdown` or under `docs/automated-tests/`).
- Evidence that every phase is COMPLETE on `develop` (merged phase PRs, plan phase table).
- `repo-delegate-to-cursor` (Composer 2.5 Fast). VERIFY `none`.
- Authority: Gene assignment. Never self-start.

## PLAN LOCATION

Canonical home for new and active QA plans:

`docs/automated-tests/`

Legacy files may still live at `docs/*-FAILING-TESTS-FIX-PLAN.markdown`, `docs/*-TEST-COVERAGE-PLAN.markdown`, `docs/*-TEST-HYGIENE-PLAN.markdown` until migrated. Retire PRs may delete from either location. New plan-build skills write under `docs/automated-tests/`.

## SEQUENCE OF WORK

1. Fetch `origin/develop`. Confirm the plan file exists and quote the phase summary table.
2. **Completeness gate (author pre-check):**
   - Every phase row is COMPLETE (or WONTFIX / ESCALATED with Angelo written disposition).
   - No OPEN implementable phases remain.
   - Last phase PR is merged (or Gene waives with written reason).
   - If any phase is OPEN / PARTIAL without disposition → STOP and return to Gene. Do not open a delete PR.
3. Open branch `docs/retire-<slug>` from `origin/develop`.
4. Docs-only change set:
   - `git rm` the finished plan file(s) listed in Gene's assign.
   - Optionally add a one-line note in `docs/automated-tests/README.md` (create if missing) that the plan was retired with date + last phase PR numbers.
   - Do **not** edit `tests/` or `app/`.
5. Draft PR targeting `develop`:
   - Title: `:memo: retire <plan filename>`
   - Body: plan path, phase completeness table (phase → COMPLETE + PR), delete list, verify `none`.
6. Hand to Gene for Katherine `qa-plan-retire-audit`. Stay draft. Never mark ready. Never merge.

## HOW TO VALIDATE

- Diff is docs-only (plan delete ± README note).
- Completeness gate passed with cited merged PR numbers.
- PR is draft.

## WHAT TO RETURN

Draft PR URL, files deleted, completeness table, HEAD SHA.

## WHAT REQUIRES APPROVAL

Push/draft PR need no approval. Escalate to Gene when completeness is unclear or Angelo disposition is missing on ESCALATED/WONTFIX rows.
