---
name: qa-ci-log-pull
description: >-
  Use this when a CI run on develop finished with failures and needs a fix plan
  or a re-baseline.
---
# QA CI log pull

## When to use

A CI run on develop finished with failures and needs a fix plan or a re-baseline.

## Required inputs and access

- GitHub Actions run URL or run ID for CorewareHub/coreware-app-backend.
- `gh` authenticated with read access to Actions.

## Sequence of work

1. `gh run view <id> --repo CorewareHub/coreware-app-backend` for branch, conclusion, and job list.
2. Download the failing job's log for the test-execution step.
3. Save under `database/data-dumps/YYYYMMDD-test-logs/` using today's date and the repo's dump naming convention. Same-day re-pulls append `-ii`, `-iii`, `-iv`. Multi-file pulls go in a directory. Never commit these files.
4. Extract the metrics table: total tests, passed, failed, skipped, duration, worker count, branch, and the random-order seed.
5. Extract every failure as: test file path, test name, assertion or exception message, first meaningful stack frame.
6. Aggregate per file, sorted descending by failure count.

## How to validate

- The failure count in the summary line equals the number of extracted failures. A mismatch means the log was truncated — re-pull before continuing.
- The seed is present. Without it the run is not reproducible.
- Spot-check three failures against the raw log text.

## What to return

Dump path, metrics table, per-file inventory sorted by count, and the reproduce command: `PEST_SEED=<seed> composer test`

## What requires approval

None — read-only. Never commit a dump. Never paste credentials or customer data found in a log into chat.
