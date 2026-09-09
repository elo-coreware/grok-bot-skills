---
name: qa-validity-scan
description: >-
  Use this when scanning failing test files of a new CI run (Aaron), every test
  file listed in a new fix-plan PR (Katherine), changed test files of a phase
  branch (Katherine), or proposed tests on a coverage plan PR (Katherine mode d).
---
# QA validity scan

## When to use

(a) Aaron, on the failing files of a new CI run, before writing a fix plan.

(b) Katherine, on every test file listed in a new fix-plan PR, before that plan merges. She checks whether those tests are still valid to include as work, not whether the phases are well sequenced.

(c) Katherine, on the changed test files of a phase branch, before it merges.

(d) Katherine, on a coverage plan PR (`docs/*-TEST-COVERAGE-PLAN.markdown`). The
    proposed tests do not exist yet — audit each proposal row against validity
    criteria (weak assertions, prohibited unit tests, missing app/ citations, wrong
    paths). Use proposal IDs (phase number + proposed file path) instead of file:line.

## Required inputs and access

- The file list to scan and read access to the repo at the right ref.
- `.cursor/commands/automated-tests-validity-detection.md` is the authority. Where it conflicts with this skill, the command file wins.

## Sequence of work

1. Establish scope.
   - Aaron: the failing files from the CI log.
   - Katherine, plan PR: every test file named in the plan's inventory / phase tables.
   - Katherine, phase branch: `git diff develop...HEAD --name-only --diff-filter=ACMR -- 'tests/**/*Test.php'`
   - Katherine, coverage plan PR: every proposed test row in the plan's phase tables
     (Author and Extend phases). For Extend, also read the existing test file.
2. Read each file in full (modes a–c). For mode (d), read the plan phase row and
   cited app/ sources; read existing files for Extend phases. Never judge from grep
   hits alone.
3. BLOCKER pass — empty or no-op test bodies; `assertTrue(true)` and other tautologies; self-referential expectations; commented-out assertions; tests under `tests/Feature/Tenant/` that never enter `$tenant->execute()`; files with no `Tenant::first()` in `beforeEach`; tests asserting framework behavior instead of application behavior.
4. HIGH pass — `sleep()` for ordering; emails or slugs on unique columns without `uniqid()`; a POST/PUT/PATCH whose assertions stop at `assertOk` or `assertStatus` with no database or model-state check; several unrelated HTTP calls in one test; `beforeEach` with no matching `afterEach` cleanup; missing `uses(FactoryDataHelper::class)`.
5. MEDIUM pass — `->skip()`, `markTestSkipped`, `markTestIncomplete`, TODO/FIXME; mixed `expect()` and `$this->assert*()` styles in one file; standalone `it()` with no `describe()` wrapper; generic test names.
6. Mode (d) additional BLOCKER pass on proposals:
   - Proposed service unit test under `tests/Unit/` without `// CRITICAL:` header
   - Missing `app/` file:line citation on the proposal row
   - Proposed POST/PUT/PATCH with only status assertions and no DB/model check described
   - Proposed path violates automated-tests.md directory convention
7. Record each finding as file:line (modes a–c) or proposal ID (mode d), severity,
   the offending snippet or proposal text, and the concrete fix.

## How to validate

- Every finding has a line number that resolves in the file (modes a–c) or a proposal
  ID that resolves in the plan (mode d).
- Every BLOCKER states a fix, not just a complaint.
- Clean files are listed explicitly, so silence is never ambiguous.
- On a fix-plan PR, every file the plan treats as work appears in either findings or
  the clean-file list.
- On a coverage plan PR, every proposed phase appears in either findings or the
  clean-proposal list.

## What to return

Counts by severity, findings grouped BLOCKER then HIGH then MEDIUM, then the clean-file list. This skill never edits a file.

## What requires approval

Gene approves the scan result before downstream work proceeds: only when no BLOCKER is unaddressed and every HIGH is fixed or justified in writing. Gene escalates to Angelo instead of approving when a fix needs `app/` business-logic changes, assertion count dropped, a root cause is still unknown, or a file failed audit twice.
