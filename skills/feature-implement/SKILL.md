---
name: feature-implement
description: Use when a feature engineer implements after plan PASS on the same feature PR
---

## BACKEND BRANCHING (Angelo 2026-09-21, clarified)

For `CorewareHub/coreware-app-backend`:

- **Tip names:** `feature/<name>` for features; `fix/<name>` for fixes. Do **not** use tip prefix `dev-test/<name>` as the default.
- **Normal PR base:** `develop`.
- **`dev-test` is situational:** use it only when you need to run tests, or need the change to reflect on coreware-app-backend DEV (primary tenant https://development-corestore-alpha.coreware.app). Do not make every backend PR target `dev-test`.
- Never tip-push experiments onto a protected base. Never commit on `develop`, `dev-test`, `main`, or `master`.

# Feature implement

Susan Kare, Jean Bartik, Adele Goldberg, or another assigned feature engineer. After plan PASS only.

## CONTROL TOWER BRANCHING (Angelo 2026-09-22)

For `CorewareHub/boss-control-tower`:

- **Tip names:** `feature/<name>` for features; `fix/<name>` for fixes. Do **not** default tip prefix to `develop/<feature-name>`.
- **Normal PR base:** `develop/develop` (Control Tower main).
- **`develop/<feature-name>` is situational:** use it only when you need **visual confirmation** on DEV (https://dev.coreware.app) — same idea as backend using `dev-test` only for tests/DEV reflection.
- Never tip-push experiments onto `develop/develop`.

## ONE PR rule (Angelo 2026-09-18)

Implement on the **existing** feature PR/branch that already holds the plan. Do **not** open a new implement PR in the same repo. Related work in the other repo is a separate pair PR only.

## Sequence

1. Confirm plan PASS on the feature PR and Gene assigned this implement.
2. Implement on that **same** branch/PR. Base remains the repo main (`develop` or `develop/develop`).
3. DEV checks: boss-control-tower tip `feature/<name>` or `fix/<name>` by default on https://dev.coreware.app; use `develop/<feature-name>` only for visual confirmation (never tip-push onto `develop/develop`). coreware-app-backend `feature/<name>` or `fix/<name>` on https://development-corestore-alpha.coreware.app. Never use https://coreware.coreware.app to preview features.
4. Full-repo local Pint before handoff if PHP changed or Check Code Style is red (`./vendor/bin/pint` then `./vendor/bin/pint --test`; never `--dirty`-only). Never `composer format`.
5. Request the shared test slot from Gene before any Pest / migrate / schema dump.
6. Push, comment `cursor review` as Angelo on the handoff commit, ping Gene.
7. Gene assigns Raye or Grace to babysit **this same PR**. You stop unless Gene assigns a follow-up on that PR.

## Constraints

- Never merge. Never commit on `develop`, `develop/develop`, `dev-test`, `main`, or `master`.
- Scope is the committed plan. Product-behavior changes beyond it escalate to Angelo.
- Follow repo-delegate-to-cursor. Do not pin a Composer model version.

## PROD peek vs DEV writes (Angelo 2026-09-20)

- `https://controltower.coreware.app` and `https://coreware.coreware.app` are PRODUCTION. Observe/peek only when Angelo explicitly asks. NO modifying.
- Implement and verify on DEV tips (`https://dev.coreware.app`, `https://development-corestore-alpha.coreware.app`). If a recheck needs data changes, use DEV only — never mutate PROD.
