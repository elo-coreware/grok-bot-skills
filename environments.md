# Environments

Site URLs for Coreware bot work. No secrets.

## Host map (do not confuse)

| Host | Role |
|------|------|
| https://controltower.coreware.app | Landlord Control Tower **PRODUCTION** |
| https://dev.coreware.app | Control Tower **DEV** |
| https://coreware.coreware.app | Backend / tenant app **PRODUCTION** |
| https://development-corestore-alpha.coreware.app | Backend **DEV** primary tenant |

`controltower.coreware.app` ≠ `coreware.coreware.app`. Landlord Control Tower PROD is not the tenant/backend app PROD.

## boss-control-tower

| Env | URL | Notes |
|-----|-----|-------|
| DEV | https://dev.coreware.app | Feature tips use `develop/<feature-slug>`. Never tip-push experiments onto `develop/develop` (that is main). |
| PRODUCTION | https://controltower.coreware.app | Landlord Control Tower PRODUCTION. Never tip-push experiments here. Never use for feature preview. |

## coreware-app-backend

| Env | URL | Notes |
|-----|-----|-------|
| DEV (primary tenant) | https://development-corestore-alpha.coreware.app | Primary DEV tenant. Feature tips use `dev-test/<feature>` — not `develop/<feature>`. Base/main remains `develop`. |
| PRODUCTION | https://coreware.coreware.app | Tenant/backend app PRODUCTION — different from Control Tower landlord PROD. Never use for feature preview or experiments. Backend feature PRs target `develop`. |
