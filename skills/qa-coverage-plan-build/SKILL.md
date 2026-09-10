---
name: qa-coverage-plan-build
description: >-
  Use after qa-module-test-inventory when a module has uncovered or under-tested
  surface. Writes docs/YYYYMMDD-<module>-TEST-COVERAGE-PLAN.markdown.
---
# QA coverage plan build

## When to use

After qa-module-test-inventory, when the coverage matrix has symbols mapped to `None`
or existing tests lack mutation/authorization/edge-case coverage worth a new phase.
Do not use for CI failure triage — that is qa-fix-plan-build (Aaron).

## Required inputs and access

- Full output from qa-module-test-inventory for the module.
- Previous coverage plan for the same module in `docs/` (for Supersedes line), if any.
- Repo read access via `repo-delegate-to-cursor` (Composer 2.5 Fast).
- Authority: `.cursor/commands/automated-tests.md` (gap analysis, test types, paths).
- Authority: `.cursor/rules/codebase.mdc` (testing policy — no new service unit tests).
- Canonical new-test precedent: PR #2191 (`b35add9667`, `:robot: add notebook api http tests`).
- Extension precedent: `03190d4c13` (edge-case-first), `1cd1fb1622` (new HTTP + strengthen existing).

## Sequence of work

1. Name the file `docs/YYYYMMDD-<module>-TEST-COVERAGE-PLAN.markdown`. Open with
   Supersedes link to prior version (if any), module key, inventory date, and link
   to the inventory summary.
2. **Summary table** — module key, boundary globs, surface symbol count, existing
   test file count, uncovered symbol count, proposed new files, proposed extensions.
   No percentages.
3. **Mandatory read pass** — before proposing any assertion for each gap, read:
   - Migration(s) for table/column names (PR #2191 lesson: `user_id` not `reactor_id`)
   - Route file for route name and route key (`ulid` vs `id`)
   - Form Request for validation rules (`min:10`, required fields, enums)
   - Controller method for response shape and status codes
   Record the citations in each phase row.
4. **Per proposed test**, document all four:
   - Behavior contract locked (user-visible outcome, not method name)
   - Assertion that fails if it regresses
   - Specific regression it prevents
   - `app/` file:line citation
5. **Gap categories** (from automated-tests.md):
   - Extension needed — existing file, new `it()` blocks
   - New test needed — new file under correct Feature path
   - Missing edge cases — auth, validation failure, tenant isolation, null handling
6. **Phased plan** — one phase per deliverable test file, manifest format from PR #2191:

   ```
   Phase 1: NotebookTest.php — Core CRUD, favorites, search, global access
   Phase 2: NotebookPageTest.php — Page CRUD, hierarchy, reordering, move, duplicate
   ```

   Each phase includes:
   - Proposed path (`tests/Feature/Tenant/Http/<Module>/<Class>Test.php`)
   - Test type (Feature tenant / Feature landlord / Job test)
   - Work type: **Author** (new file) or **Extend** (add `it()` to existing file)
   - Full proposed `describe()` / `it()` structure
   - Setup requirements (`FactoryDataHelper`, `$tenant->execute()`, fakes, uniqid scoping)
   - Key assertions per `it()` block
   - Verify command: `composer test:single -- <path>`
   - Implementer instructions (do / do not; named files; escalate if app/ change needed)

7. **Priority ordering** — Critical (mutations, auth) > High (new service flows) >
   Medium (edge cases) > Low (read-only accessors).
8. **Sizing** — cap at ~10 new/extended files per plan. If inventory exceeds that,
   split by submodule (`shop.checkout` vs `shop.catalog`) and state which sub-plan
   this file covers. Remaining symbols go in a "Deferred" section with rationale.
9. Ship via repo-delegate-to-cursor: branch `docs/YYYYMMDD-<module>-test-coverage-plan`,
   commit `:memo: add <month day> <module> test coverage plan vN`, draft PR targeting
   develop.

## Hard prohibitions

- **No new service unit tests.** Policy allows HTTP/feature, E2E, and job tests only.
  Unit tests only for payment/auth/security with `// CRITICAL:` header. The ~265 files
  in `tests/Unit/Tenant/Services` are grandfathered — read as existing coverage, never
  extend that tree for new work.
- **No coverage percentages** — structural matrix only.
- **No assertions without app/ file:line citation.**
- **No guessing column names, route keys, or validation bounds** — mandatory read pass
  must complete first.

## How to validate

- Every uncovered Critical/High symbol from inventory appears in exactly one phase
  or in Deferred with reason.
- Every proposed test has all four fields (contract, assertion, regression, citation).
- Every phase has a runnable `composer test:single` command.
- New file count ≤ 10 (or plan explicitly declares submodule split).
- No proposed paths under `tests/Unit/Tenant/Services` or `tests/Unit/Services` for
  new service logic (unless CRITICAL security with header — escalate to Angelo first).
- Plan references automated-tests.md path conventions.

## What to return

The plan file path, draft PR URL, phase table summary for Gene, and counts:
proposed new files, extensions, deferred symbols. Summarize Critical/High phases first.

## What requires approval

Push and draft PR need no approval. Gene assigns Katherine qa-validity-scan mode (d)
on the plan PR. Angelo merges after Katherine PASS and Gene approval. Escalate to
Gene/Angelo when gaps require `app/` changes to test (missing route, broken contract)
rather than new tests alone.
