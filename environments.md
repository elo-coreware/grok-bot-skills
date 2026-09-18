# Environments

Site URLs for Coreware bot work. No secrets.

## PRODUCTION HARD RULE (Angelo 2026-09-18)

Applies to **BOTH** production hosts:

- https://controltower.coreware.app — landlord Control Tower PROD
- https://coreware.coreware.app — backend/tenant app PROD

**HARD RULE:** bots may ONLY observe / peek when Angelo explicitly asks. **NO modifying** — no edits, creates, deletes, status changes, comments that change state, deploys, tip-pushes, form submits, or any write API. Read-only / peek only.

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
| PRODUCTION | https://controltower.coreware.app | Landlord Control Tower PRODUCTION. HARD RULE (Angelo 2026-09-18): bots may ONLY observe / peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, comments that change state, deploys, tip-pushes, form submits, or any write API. Read-only / peek only. Never tip-push experiments. Never use for feature preview. |

## coreware-app-backend

| Env | URL | Notes |
|-----|-----|-------|
| DEV (primary tenant) | https://development-corestore-alpha.coreware.app | Primary DEV tenant. Feature tips use `dev-test/<feature>` — not `develop/<feature>`. Base/main remains `develop`. |
| PRODUCTION | https://coreware.coreware.app | Tenant/backend app PRODUCTION — different from Control Tower landlord PROD. HARD RULE (Angelo 2026-09-18): bots may ONLY observe / peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, comments that change state, deploys, tip-pushes, form submits, or any write API. Read-only / peek only. Never use for feature preview or experiments. Backend feature PRs target `develop`. |
