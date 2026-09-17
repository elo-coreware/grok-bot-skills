# Environments

Site URLs for Coreware bot work. No secrets.

## boss-control-tower

| Env | URL | Notes |
|-----|-----|-------|
| DEV | https://dev.coreware.app | Feature tips use `develop/<feature-slug>`. Never tip-push experiments onto `develop/develop` (that is main). |

## coreware-app-backend

| Env | URL | Notes |
|-----|-----|-------|
| DEV (primary tenant) | https://development-corestore-alpha.coreware.app | Primary tenant for DEV backend work. |
| PRODUCTION | https://coreware.coreware.app | Never use for feature preview or experiments. Backend feature PRs target `develop`. |
