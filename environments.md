# Environments

Site URLs for Coreware bot work. No secrets.

## PROD web hosts — observe / peek only (Angelo 2026-09-18)

For BOTH https://controltower.coreware.app (landlord Control Tower PRODUCTION) and https://coreware.coreware.app (backend/tenant PRODUCTION):
Bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify.

## Host map (do not confuse)

| Host | Role |
|------|------|
| https://controltower.coreware.app | Landlord Control Tower **PRODUCTION** (observe/peek only when Angelo asks — NO modifying) |
| https://dev.coreware.app | Control Tower **DEV** |
| https://coreware.coreware.app | Backend / tenant app **PRODUCTION** (observe/peek only when Angelo asks — NO modifying) |
| https://development-corestore-alpha.coreware.app | Backend **DEV** primary tenant |

`controltower.coreware.app` ≠ `coreware.coreware.app`. Landlord Control Tower PROD is not the tenant/backend app PROD.

## boss-control-tower

| Env | URL | Notes |
|-----|-----|-------|
| DEV | https://dev.coreware.app | Feature tips use `develop/<feature-slug>`. Never tip-push experiments onto `develop/develop` (that is main). |
| PRODUCTION | https://controltower.coreware.app | Landlord Control Tower PRODUCTION. Observe/peek only when Angelo asks. Never tip-push. Never feature preview. |

## coreware-app-backend

| Env | URL | Notes |
|-----|-----|-------|
| DEV (primary tenant) | https://development-corestore-alpha.coreware.app | Primary DEV tenant. Tips: `feature/<name>` or `fix/<name>` (not tip prefix `dev-test/<name>` by default). Normal PR base: `develop`. Use `dev-test` only for tests or DEV tenant reflection (Angelo 2026-09-21 clarified). |
| PRODUCTION | https://coreware.coreware.app | Tenant/backend PRODUCTION — different from Control Tower landlord PROD. Observe/peek only when Angelo asks. Never feature preview. Backend feature PRs normally target `develop`. |

## PROD recheck vs DEV writes (Angelo 2026-09-20)

When Angelo asks a feature engineer (or any bot) to recheck PROD: peek/observe ONLY on
https://controltower.coreware.app and https://coreware.coreware.app. NO modifying.

If you need to change data to test a feature, use DEV only:
- https://dev.coreware.app (Control Tower DEV)
- https://development-corestore-alpha.coreware.app (backend DEV primary tenant)

Never mutate PROD to verify.
