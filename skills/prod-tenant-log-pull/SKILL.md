---
name: prod-tenant-log-pull
description: >-
  Use when Angelo asks Seymour (or devops) to pull Coreware tenant application
  logs (S3) and/or request logs (CloudWatch), observe metrics, or peek AWS infra
  for diagnosis — especially production — only after an explicit in-chat
  permission grant, and only via AWS read-only commands on Angelo’s local
  machine. File dumps and analysis go on the shared Grok Bot computer. Never
  remediate, provision, or tune; show Angelo fix commands first.
---
# Pull tenant production logs (observe-only)

## When to use

Use when Angelo asks to pull **application logs** and/or **request logs** for one or more Coreware tenants (by name, domain, or Control Tower tenant id) for given date(s), or to **observe metrics / peek AWS infrastructure** for diagnosis. Typical hosts look like `{tenant}.coreware.app` or Control Tower URLs under `controltower.coreware.app/tenants/{id}/…-logs`.


## LOCAL vs BOX (Angelo 2026-09-21; Seymour ACK'd)

**Angelo's local computer (machineId / ListMachines):**
- AWS-related commands **ONLY**: AWS CLI, AWS SSO/login helpers, and AWS observe/peek (S3, CloudWatch, Cost Explorer, ECS, SQS, STS, `describe`/`list`/`get` style) unless Angelo explicitly authorizes mutate.
- **NO** non-AWS commands on local (no `ls`/`find` under Code, no `gh` for CT PRs, no staging dumps via local Shell, etc.).

**Shared Grok Bot computer (no machineId):**
- Everything else: file dumps, analysis, plan drafting, non-AWS tooling, reading attachments, writing `data-dumps`/workspace.

**Files Angelo gives Seymour:** save/copy onto the shared remote Grok Bot computer (prefer chat attachments that already land on the box). Not a license to broaden local-shell access.

## Observe-only standing rule (Angelo 2026-09-18)

Seymour (and this skill) may **only**:

- Pull logs (application + request; Control Tower UI pages and/or read-only AWS)
- Observe metrics
- Peek into AWS infrastructure (`describe` / `list` / `get` and equivalent read-only)

Seymour must **not**:

- Execute remediation or “fix” commands
- Provision new services
- Fine-tune server settings or change configuration

If diagnosis finds a problem that needs a fix: **draft the exact command(s) and show Angelo first**. Angelo runs them, or explicitly tells Seymour to. Do not self-serve mutate, provision, or tune.

Read-only log pulls and peeks remain OK under a fresh explicit grant.

## Hard permission gate (PROD)

Production log pulls are **forbidden** until Angelo has given an **explicit permission sentence in the current chat** for this pull. A vague “check the issue” or an old memory of a prior grant is **not** enough.

The permission must name at least:

1. That this is a **read-only / peek** production access (no writes, no schema changes, no deploys)
2. **Which tenants** (name, domain, and/or tenant id)
3. **Which dates** (or “today”)
4. **Which log types** (application, request, or both)

If any of those are missing, stop and ask. If the grant is only for peeking landlord metadata, do not widen into tenant DB writes or other accounts.

AWS/DB read commands for this skill run **only on Angelo’s registered local computer** (his AWS CLI / SSO). Never pull PROD logs from the shared Grok Bot computer. After the AWS pull, put dumps and analysis on the **box** (see LOCAL vs BOX).

## PROD web hosts — observe / peek only (Angelo 2026-09-18)

Hard rule for **both**:

- `https://controltower.coreware.app` — landlord Control Tower PRODUCTION
- `https://coreware.coreware.app` — backend / tenant app PRODUCTION

Bots may **only observe or peek** on these hosts when Angelo explicitly asks. **No modifying** — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify. Log pulls use AWS + read-only landlord DB metadata, not mutations in the live web UI. Control Tower application-logs / request-logs pages are an allowed read path when granted.

## What the two log types actually are

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

1. Restate the grant back to Angelo in one short paragraph (tenants, dates, log types, read-only, AWS-on-local-only). Wait for confirmation if anything is ambiguous.
2. Confirm timezone for “day” boundaries. Application files rotate on the app’s log date (often UTC). CloudWatch windows are explicit epochs — Eastern business day ≠ UTC file date. If unclear, ask. When spanning an Eastern day across UTC, download the UTC-dated application files that cover the full window.
3. Confirm dump destination on the **shared Grok Bot computer** (e.g. gitignored `data-dumps/…` / workspace under `/home/box` or `/workspace`). AWS commands run on local; do not stage dumps via local Shell beyond what the AWS CLI itself writes. Prefer chat attachments that already land on the box. Never commit dumps. Treat contents as PII-heavy.
4. Pass credentials via environment / SSO — never echo secrets into argv, chat, or inventory files.

## Execution order

1. Read-only landlord `SELECT` for the target tenant id(s) → confirm name/domain, resolve S3 bucket + CloudWatch group.
2. `aws s3 cp` / get-object for each tenant × each needed application log date (try tenant-scoped key, then legacy flat key) — **on Angelo’s local machine only**.
3. `aws logs filter-log-events` per tenant per agreed window with `"HTTP_REQUEST:TENANT_{id}"`, follow `nextToken` to the end (or Angelo’s cap). Save raw JSON; optionally flatten to timestamp / method / status / duration / url text.
4. Copy/save dumps + `inventory.txt` onto the **shared Grok Bot computer** for analysis. Do not commit.
5. If the pull points at a fix (bad config, restart, IAM change, scale, etc.): **stop and show Angelo the exact proposed command(s)** — do not run them.

## Report back

- Dump / analysis path on the **shared Grok Bot computer** (and note that AWS ran on local)
- Per tenant: files pulled, event counts, any missing objects / empty windows
- Retention cautions (some CloudWatch groups keep only ~7 days)
- Whether tenant DB credentials were left unused (preferred)
- Short anomalies only if obvious while verifying the pull — deep product diagnosis is a separate assign
- Any proposed remediation commands (for Angelo to run), clearly labeled as **not executed**

## Never

- Pull PROD logs without a fresh explicit Angelo grant in-chat
- Modify `controltower.coreware.app` or `coreware.coreware.app` (observe/peek only)
- Write to landlord or tenant databases
- Run remediation, provision services, or tune server settings yourself
- Run AWS/PROD pulls from the shared bot computer
- Run non-AWS commands on Angelo’s local machine (see LOCAL vs BOX)
- Merge, deploy, or tip-push as part of a log pull
- Paste secrets, customer payloads, or full dump contents into chat
