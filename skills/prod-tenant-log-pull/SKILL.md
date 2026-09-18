---
name: prod-tenant-log-pull
description: >-
  Use when Angelo asks Seymour (or devops) to pull Coreware tenant application
  logs (S3) and/or request logs (CloudWatch) for diagnosis — especially
  production — only after an explicit in-chat permission grant, and only via
  commands on Angelo’s local machine.
---
# Pull tenant production logs

## When to use

Use when Angelo asks to pull **application logs** and/or **request logs** for one or more Coreware tenants (by name, domain, or Control Tower tenant id) for given date(s), usually to diagnose a production issue. Typical hosts look like `{tenant}.coreware.app` or Control Tower URLs under `controltower.coreware.app/tenants/{id}/…-logs`.

## Hard permission gate (PROD)

Production log pulls are **forbidden** until Angelo has given an **explicit permission sentence in the current chat** for this pull. A vague “check the issue” or an old memory of a prior grant is **not** enough.

The permission must name at least:

1. That this is a **read-only / peek** production access (no writes, no schema changes, no deploys)
2. **Which tenants** (name, domain, and/or tenant id)
3. **Which dates** (or “today”)
4. **Which log types** (application, request, or both)

If any of those are missing, stop and ask. If the grant is only for peeking landlord metadata, do not widen into tenant DB writes or other accounts.

This work runs **only on Angelo’s registered local computer** (his AWS CLI / SSO / GitHub CLI). Never pull PROD logs from the shared Grok Bot computer.

## PROD web hosts — observe / peek only (Angelo 2026-09-18)

Hard rule for **both**:

- `https://controltower.coreware.app` — landlord Control Tower PRODUCTION
- `https://coreware.coreware.app` — backend / tenant app PRODUCTION

Bots may **only observe or peek** on these hosts when Angelo explicitly asks. **No modifying** — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Log pulls use AWS + read-only landlord DB metadata, not mutations in the live web UI.

## What the two log types actually are

They come from different places:

### Application logs (S3)

Not in a database. Each tenant’s Laravel daily log lives in **that tenant’s own S3 bucket**:

- Prefer: `logs/tenant-{id}/laravel-YYYY-MM-DD.log`
- Fallback: `logs/laravel-YYYY-MM-DD.log`

Bucket name is in the landlord DB `tenants.config` JSON at `filesystems.disks.s3.bucket`. Reads use the provision/AWS pair Control Tower uses for tenant buckets (or Angelo’s SSO role if that is what he authorized).

### Request logs (CloudWatch)

Not S3 and not the DB. Filter CloudWatch Logs for the tenant’s log group (`tenants.cloudwatch_log_group`, often defaulting to something like `/ecs/coreware-app`) with pattern `"HTTP_REQUEST:TENANT_{id}"` over an explicit start/end epoch-ms window. Paginate `nextToken` to completion unless Angelo caps volume.

### Landlord DB (read-only metadata only)

One `SELECT` against the Control Tower landlord DB for the tenant row(s): `id`, `name`, `domain`, `cloudwatch_log_group`, `config`. **SELECT only.** Do not touch tenant `corestore` DBs unless Angelo explicitly expands the grant and the logs truly require it (usually they do not).

## Before any AWS or DB call

1. Restate the grant back to Angelo in one short paragraph (tenants, dates, log types, read-only, local machine only). Wait for confirmation if anything is ambiguous.
2. Confirm timezone for “day” boundaries. Application files rotate on the app’s log date (often UTC). CloudWatch windows are explicit epochs — Eastern business day ≠ UTC file date. If unclear, ask. When spanning an Eastern day across UTC, download the UTC-dated application files that cover the full window.
3. Confirm output folder (default under a gitignored `database/data-dumps/…` on the machine Angelo named). Never commit dumps. Treat contents as PII-heavy.
4. Pass credentials via environment / SSO — never echo secrets into argv, chat, or inventory files.

## Execution order

1. Read-only landlord `SELECT` for the target tenant id(s) → confirm name/domain, resolve S3 bucket + CloudWatch group.
2. `aws s3 cp` / get-object for each tenant × each needed application log date (try tenant-scoped key, then legacy flat key).
3. `aws logs filter-log-events` per tenant per agreed window with `"HTTP_REQUEST:TENANT_{id}"`, follow `nextToken` to the end (or Angelo’s cap). Save raw JSON; optionally flatten to timestamp / method / status / duration / url text.
4. Write an `inventory.txt` listing what was found vs missing, windows used, and sizes. Do not commit.

## Report back

- Folder path on Angelo’s machine
- Per tenant: files pulled, event counts, any missing objects / empty windows
- Retention cautions (some CloudWatch groups keep only ~7 days)
- Whether tenant DB credentials were left unused (preferred)
- Short anomalies only if obvious while verifying the pull — deep product diagnosis is a separate assign

## Never

- Pull PROD logs without a fresh explicit Angelo grant in-chat
- Modify `controltower.coreware.app` or `coreware.coreware.app` (observe/peek only)
- Write to landlord or tenant databases
- Run this from the shared bot computer
- Merge, deploy, or tip-push as part of a log pull
- Paste secrets, customer payloads, or full dump contents into chat
