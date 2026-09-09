---
name: qa-suite-hygiene-plan-build
description: >-
  Use after qa-module-test-inventory when duplication clusters or placement
  violations exist. Writes docs/YYYYMMDD-<module>-TEST-HYGIENE-PLAN.markdown.
---
# QA suite hygiene plan build

## When to use

After qa-module-test-inventory, when duplication clusters, placement violations,
weak assertions, or validity issues exist in the module's existing tests. Separate
from qa-coverage-plan-build because this plan authorizes deletions, merges, renames,
and relocations — Katherine treats net assertion drops as FAIL unless this plan
documents the expected delta.

## Required inputs and access

- Full output from qa-module-test-inventory (duplication clusters, placement violations).
- Previous hygiene plan for the same module in `docs/` (Supersedes line), if any.
- Repo read access via `qa-delegate-to-cursor` (Composer 2.5 Fast).
- Authority: `.cursor/commands/automated-tests-validity-detection.md`.
- Authority: `.cursor/commands/automated-tests.md` (directory conventions).
- Katherine audit rule: net assertion drop or removed test = FAIL unless plan authorized
  (see qa-fix-audit).

## Sequence of work

1. Name the file `docs/YYYYMMDD-<module>-TEST-HYGIENE-PLAN.markdown`. Open with
   Supersedes link, module key, and inventory date.
2. **Summary table** — duplication clusters found, placement violations, weak assertion
   files, proposed merges, proposed relocations, proposed renames, expected net
   assertion delta (single number for the whole plan).
3. **Merge before delete** — default remedy for duplicate pairs is merge unique
   assertions into the survivor. Deletion of a file is proposed only when two files
   are assertion-for-assertion identical. Never delete to "reduce noise" without
   equivalence proof.
4. For each **merge or deletion**, record:
   - File A path and File B path
   - Shared route(s) and overlapping assertion summary (equivalence proof)
   - Survivor path and why it matches placement convention
   - Assertions moving from loser to survivor (list by `it()` name)
   - **Expected assertion-count delta** (integer; merge often ≥ 0; identical delete = 0)
5. For each **relocation**, record:
   - Source path → destination path
   - Convention violated and convention satisfied
   - Note: use `git mv` to preserve history
   - Note: crossing `tests/Unit` ↔ `tests/Feature` changes tenant provisioning via `Pest.php`
   - Updated `composer test:single` path after move
6. For each **rename** (`it()` or `describe()`), record:
   - Old name → new name
   - Evidence old name misdescribed behavior (`app/` file:line citation)
   - Without this record, Katherine FAILs under qa-fix-audit step 6 (PR 5785 / e6e8e26f99 class)
7. **Enhancement categories** — tag each phase with one type and cite precedent:
   - Tighten weak assertions — status-only POST/PUT; add DB/model checks (`053e7ed0bc`)
   - Consolidate cleanup — shared helper, slug-scoped teardown (`bec1a1a35a`, PublicBlogTestHelper)
   - Restore strict assertions — isolation fix, not weakening (`e6e8e26f99`)
   - Remove runtime Schema DDL — forbidden in tests (`ddd996edf5`)
   - Fix tenant context — missing `$tenant->execute()` (validity BLOCKER)
   - Fix flaky patterns — sleep, non-uniqid emails (`automated-tests-validity-detection.md`)
8. **Phased plan** — one phase per logical hygiene deliverable (may touch 1–3 files):
   - Phase table: files, work type **Reconcile**, enhancement category, expected
     assertion delta for this phase, verify command
   - Implementer instructions: merge steps, git mv paths, rename list, do not delete
     unless authorized
9. **Plan-level assertion arithmetic** — sum of per-phase expected deltas must equal
   the summary net delta. State the arithmetic explicitly.
10. Ship via qa-delegate-to-cursor: branch `docs/YYYYMMDD-<module>-test-hygiene-plan`,
    commit `:memo: add <month day> <module> test hygiene plan vN`, draft PR targeting
    develop.

## How to validate

- Every duplication cluster from inventory appears in exactly one phase or Deferred.
- Every merge/deletion has equivalence proof and expected assertion delta.
- Every rename has old/new names and app/ citation.
- Every relocation specifies git mv and updated verify path.
- Plan-level assertion delta equals sum of phase deltas.
- No phase proposes deleting tests without merge-first justification.
- Deferred items explain why (e.g. needs product sign-off, cross-module dependency).

## What to return

The plan file path, draft PR URL, phase table, plan-level expected assertion delta,
and merge/rename/relocation counts for Gene.

## What requires approval

Push and draft PR need no approval. Katherine audits the plan PR with qa-validity-scan
on existing files listed in the plan (mode b) plus authorization records. Angelo merges
after Katherine PASS and Gene approval. Escalate when hygiene requires `app/` fixes or
when equivalence between duplicate files is unclear after reading both in full.
