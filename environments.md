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
| PRODUCTION | https://controltower.coreware.app | Landlord Control Tower PRODUCTION. PROD web hosts — observe / peek only (Angelo 2026-09-18): bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Never tip-push experiments. Never use for feature preview. |

## coreware-app-backend

| Env | URL | Notes |
|-----|-----|-------|
| DEV (primary tenant) | https://development-corestore-alpha.coreware.app | Primary DEV tenant. Feature tips use `dev-test/<feature>` — not `develop/<feature>`. Base/main remains `develop`. |
| PRODUCTION | https://coreware.coreware.app | Tenant/backend app PRODUCTION — different from Control Tower landlord PROD. PROD web hosts — observe / peek only (Angelo 2026-09-18): bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Never use for feature preview or experiments. Backend feature PRs target `develop`. |
