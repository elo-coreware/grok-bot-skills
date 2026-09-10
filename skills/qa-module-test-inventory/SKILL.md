---
name: qa-module-test-inventory
description: >-
  Use when Gene assigns a module for coverage or hygiene planning. Produces the
  surface inventory, test inventory, coverage matrix, duplication clusters, and
  placement violations that feed qa-coverage-plan-build and qa-suite-hygiene-plan-build.
---
# QA module test inventory

## When to use

Gene assigns a module key from the backlog. Tindall runs this before writing any
coverage or hygiene plan. This skill never edits files.

## Required inputs and access

- Module key from Gene (must resolve to a registry entry — never invented).
- Repo read access via `repo-delegate-to-cursor` (Composer 2.5 Fast).
- Authority for test placement: `.cursor/commands/automated-tests.md`.
- Authority for validity patterns: `.cursor/commands/automated-tests-validity-detection.md`.

## Module key resolution

The module key must come from an existing registry:

1. **Primary:** route `module-permission` keys referenced in
   `app/Services/Navigator/Search/NavigatorEntitySearchService.php` (`ENTITY_MODULES`).
2. **Secondary:** keys in `app/Services/ModuleMigrationService.php` (`FEATURE_MIGRATIONS`).

If the key does not resolve, STOP and ask Gene for a valid key. Do not invent
submodules unless splitting an oversized plan (e.g. `shop.checkout`) and document
the parent key in the inventory output.

Emit the concrete path globs that define the module boundary, for example:

- `app/Http/Controllers/<Module>/`
- `app/Services/<Module>/`
- `app/Jobs/` (filtered by namespace or filename prefix)
- `routes/` entries whose `module-permission` matches the key
- `tests/Feature/Tenant/Http/<Module>/`
- `tests/Feature/Landlord/Http/<Module>/` (when applicable)

## Sequence of work

1. Resolve the module key and write the boundary globs in the output header.
2. **Surface inventory** — enumerate testable symbols:
   - Routes (name + URI + controller@method)
   - Controller public methods (index, store, update, destroy, and module-specific actions)
   - Job classes with `handle()` in the module boundary
   - Console commands in the module boundary
   - Middleware registered for module routes
   - Landlord equivalents when the module has landlord logic under `app/Landlord/`
3. **Test inventory** — every `tests/**/*Test.php` file under the boundary globs:
   - File path
   - `describe()` / `it()` block names
   - Approximate assertion count (grep `expect(`, `assert`, `assertDatabase` per file)
   - Test type: Feature tenant, Feature landlord, Unit (grandfathered only)
4. **Coverage matrix** — for each surface symbol, map to:
   - Test file(s) + `it()` block(s) that exercise it, or `None`
   - Do not use percentages. Structural coverage only.
5. **Duplication clusters** — group files that exercise the same route + assertion
   overlap. Filename matches alone are candidates only, not proof. Known candidates
   to verify when they appear in the module boundary:
   - `ActionPlanActivationValidationTest.php`
   - `AiCreditGateMiddlewareTest.php`
   - `EmailChannelPollingServiceTest.php`
   - `SessionManagementTest.php`
   - `TenantAdminProvisioningServiceTest.php`
   For each cluster: list files, shared route(s), overlapping assertion summary,
   and recommended survivor path per `.cursor/commands/automated-tests.md` conventions.
6. **Placement violations** — files whose path does not match the source layout:
   - Controllers under `app/Http/Controllers/X/` should map to
     `tests/Feature/Tenant/Http/X/` or `tests/Feature/Landlord/Http/X/`
   - Services with HTTP tests belong in Feature, not new Unit/Services
   - Flag orphan trees: e.g. `tests/Feature/Service` outside `Tenant/` and `Landlord/`
   - Flag parallel unit trees: `tests/Unit/Tenant/Services` vs `tests/Unit/Services`
   - Read grandfathered unit files as existing coverage; do not propose adding to them
7. **Backlog signals** (optional, for Gene's ranking):
   - Count of `git log --since="6 months ago"` commits touching `app/` under the
     module globs with `:bug:` or `fix(` in subject
   - Surface symbol count vs test file count ratio

## How to validate

- Module key cites the registry source (ENTITY_MODULES or FEATURE_MIGRATIONS).
- Boundary globs are explicit and reproducible.
- Every surface symbol appears once in the coverage matrix.
- Duplication clusters cite route + assertion overlap, not filename alone.
- Placement violations cite the convention from automated-tests.md.
- No coverage percentages anywhere in the output.

## What to return

Structured inventory for Tindall and Gene:

| Section | Content |
|---------|---------|
| Module key | Registry source, boundary globs |
| Surface inventory | Symbol table (route, controller action, job, command) |
| Test inventory | File paths with describe/it summary and assertion counts |
| Coverage matrix | Symbol → test mapping or None |
| Duplication clusters | Files, shared routes, overlap summary, survivor recommendation |
| Placement violations | File path, expected path, reason |
| Uncovered symbols | Symbols with None in matrix, prioritized Critical > High > Medium > Low |

Hand this to qa-coverage-plan-build (gaps) and/or qa-suite-hygiene-plan-build
(duplication and placement).

## What requires approval

Read-only. Escalate to Gene when the module key does not resolve, the boundary is
ambiguous (two modules share the same controllers), or surface inventory exceeds
~200 symbols without a submodule split recommendation.
