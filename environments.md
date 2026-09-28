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
| DEV | https://dev.coreware.app | Tips: `feature/<name>` or `fix/<name>` by default (same spirit as backend). Normal PR base: `develop/develop`. Use tip `develop/<feature-name>` **only** for visual confirmation on this DEV host — same idea as backend `dev-test` only for tests/DEV. Never tip-push experiments onto `develop/develop` (main). (Angelo 2026-09-22) |
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

## Seymour / devops local access (Angelo 2026-09-21; Seymour ACK'd; Pest exception 2026-09-28 afternoon)

Seymour remains the sole bot for **AWS** on Angelo's local computer (ListMachines / `machineId` — CLI, SSO/login helpers, observe/peek). Non-AWS non-Pest work (file dumps, analysis, attachments, workspace) stays on the shared Grok Bot computer. Full AWS rule: `agents/seymour-cray.md` and skill `prod-tenant-log-pull`.

**NEW ListMachines exception (Pest / schema-dump):** Pest-runner bots (Margaret Hamilton, Jack Garman, Grace Hopper, Raye Montague, Susan Kare, Jean Bartik, Adele Goldberg) may use ListMachines/`machineId` **only** for Pest/test commands inside the two clone paths below. Schema-dump agents may use `machineId` **only** on the main checkout for dump regen. Seymour remains sole bot for AWS on that Mac.

## Pest host — two Mac clone slots (Angelo 2026-09-28 afternoon)

Supersedes morning 2026-09-28 "box-only Pest / never Mac" and 2026-09-11 one shared slot.

| Slot | Path | Role |
|------|------|------|
| Slot A / CLONE_A | `/Users/angelo/code/coreware-app-backend-clone` | Pest / `composer test:*` / migrate for tests |
| Slot B / CLONE_B | `/Users/angelo/code/coreware-app-backend-clone-ii` | Pest / `composer test:*` / migrate for tests |
| Main (OFF LIMITS for Pest) | `/Users/angelo/code/coreware-app-backend` | Data dump, log read, codebase analysis/read, **schema dump regen only** |

- Mac: `machineId` `ae407d63-7055-4ee5-87b3-df3ee1734ca3` / Angelos-MacBook-Air.local.
- Gene or Wernher arbitrates GRANTED / QUEUED / RELEASED **per slot**. Two Pest runs may be live at once — one per clone. If A is busy, grant B (and vice versa).
- Working directory for Pest = the granted clone path. Checkout/pull the PR tip into that clone before Pest (do not dirty the other clone).
- Garman still uses `TEST_TOKEN=9` when the clone's env expects it; token choice does not create a third slot.
- **Main FORBIDDEN for Pest / test runners.** Clones must **NOT** run schema dump — main is the only valid schema-dump tree. Serialize dump regen (one at a time) under Gene Pest GRANT or SCHEMA-DUMP GRANT.
- **No routine cloud-agent Pest.** Prefer Mac clone slots. Cloud-agent Pest is emergency-only if Angelo explicitly allows for that run. Grok Bot box is for bot chat/orchestration — never fall back to cloud Pest just because the box lacks MySQL.
