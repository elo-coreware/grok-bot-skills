---
name: feature-implement
description: Use when a feature engineer implements after plan PASS on the same feature PR
---
# Feature implement

Susan Kare, Jean Bartik, Adele Goldberg, or another assigned feature engineer. After plan PASS only.

## ONE PR rule (Angelo 2026-09-18)

Implement on the **existing** feature PR/branch that already holds the plan. Do **not** open a new implement PR in the same repo. Related work in the other repo is a separate pair PR only.

## Sequence

1. Confirm plan PASS on the feature PR and Gene assigned this implement.
2. Implement on that **same** branch/PR. Base remains the repo main (`develop` or `develop/develop`).
3. DEV checks: boss-control-tower `develop/<feature-slug>` on https://dev.coreware.app (never tip-push onto `develop/develop`). coreware-app-backend `dev-test/<feature>` on https://development-corestore-alpha.coreware.app. Never use https://coreware.coreware.app to preview features.
4. Local Pint before handoff if PHP changed. Never `composer format`.
5. Request the shared test slot from Gene before any Pest / migrate / schema dump.
6. Push, comment `cursor review` as Angelo on the handoff commit, ping Gene.
7. Gene assigns Raye or Grace to babysit **this same PR**. You stop unless Gene assigns a follow-up on that PR.

## Constraints

- Never merge. Never commit on `develop`, `develop/develop`, `main`, or `master`.
- Scope is the committed plan. Product-behavior changes beyond it escalate to Angelo.
- Follow repo-delegate-to-cursor. Do not pin a Composer model version.

## PROD peek vs DEV writes (Angelo 2026-09-20)

- `https://controltower.coreware.app` and `https://coreware.coreware.app` are PRODUCTION. Observe/peek only when Angelo explicitly asks. NO modifying.
- Implement and verify on DEV tips (`https://dev.coreware.app`, `https://development-corestore-alpha.coreware.app`). If a recheck needs data changes, use DEV only — never mutate PROD.
