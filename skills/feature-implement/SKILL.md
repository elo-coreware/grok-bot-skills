---
name: feature-implement
description: >-
  Feature engineer implements on the existing feature PR branch after Aaron PASS;
  Local Pint; request test slot before Pest; hand off same PR to Raye or Grace
---
# feature-implement

## WHEN TO USE

Gene (or Wernher) assigns the same feature engineer (Susan Kare / Jean Bartik /
Adele Goldberg) to implement after Aaron's feature-plan-validate is PASS or PASS
WITH NOTES on the **same** feature PR. Do not start on FAIL or without assignment.

## REQUIRED INPUTS AND ACCESS

- Aaron verdict comment URL (PASS / PASS WITH NOTES) on the feature PR.
- Plan document (binding scope) on that PR.
- owner/repo + base (`develop` or `develop/develop`).
- **Same branch / same PR** as the plan (Angelo standing rule 2026-09-18). If no PR
  exists yet, create **one** PR that will hold both plan and implement — never a
  second PR for the same feature in the same repo. Never branch from tip-pushed
  experiments on `develop/develop`.
- **DEV ACCESS** (environments.md):
  - https://dev.coreware.app is boss-control-tower DEV only — push
    `develop/<feature-slug>` (never tip-push experiments onto `develop/develop`).
    After push, DEV may lag (ECS/roll); hard-refresh and report what you actually
    see. Do not invent a login path or passwords; login wall → Angelo / takeover.
    Never paste credentials.
  - https://development-corestore-alpha.coreware.app is the primary tenant for DEV
    coreware-app-backend — push `dev-test/<feature>` (not `develop/<feature>`).
    Base/main remains `develop`.
  - https://coreware.coreware.app is coreware-app-backend **PRODUCTION** (tenant/backend app PROD — different from Control Tower landlord PROD) — never
    preview features or run experiments there. Backend feature PRs target `develop`.
  - https://controltower.coreware.app is landlord Control Tower **PRODUCTION** — never
    tip-push experiments here. Never use for feature preview.
- repo-delegate-to-cursor. Local Pint after PHP edits. Shared test slot before Pest.

## ONE FEATURE PR RULE

Implement **on** the existing plan/feature PR branch. Push updates to that PR. Do
**not** open a separate implement PR. Related work in the other repo stays a
separate PR there only.

## SEQUENCE OF WORK

1. Confirm Aaron PASS and Gene/Wernher implement assignment.
2. Check out the **existing** feature PR branch (plan tip). If somehow no PR
   exists, create exactly one PR that will hold both plan and code — never a second
   PR for this feature in this repo.
3. Implement only what the plan describes. Escalate product-behavior beyond the
   plan to Angelo — do not expand scope quietly.
4. **Local Pint:** after PHP edits, `./vendor/bin/pint --dirty` then
   `./vendor/bin/pint --test`. Commit style fixes on the same branch. Never
   `composer format`. Docs-only / non-PHP may skip.
5. **Test slot:** before any Pest / migrate / schema-dump, request GRANTED from
   Wernher (or Gene). Queue with Margaret / Garman / Grace / Raye / Kare / Bartik /
   Goldberg (standing rule 2026-09-11). Release when done.
6. Push implement commits to the **same** feature PR (update, do not open a second
   PR).
7. **Hand off:** unless Gene says otherwise, notify Gene that implement is on the
   feature PR so Raye or Grace can babysit **that** PR (pr-babysit-orchestrate).
   Do not babysit your own PR end-to-end unless Gene explicitly keeps you on it.
8. Never merge. Katherine audits the same code PR after babysit; Angelo merges.

## HOW TO VALIDATE

- Aaron PASS on record before first implement commit (after assignment).
- Scope matches plan; Pint clean on PHP; slot requested before Pest.
- Implement landed on the same feature PR (no second PR in this repo); Gene notified
  for Raye/Grace babysit.
- Never merged.

## WHAT TO RETURN

PR URL (same as plan PR), branch, owner/repo + base, Pint status, tests run (if any)
with slot evidence, handoff note to Gene.

## WHAT REQUIRES APPROVAL

Pushing implement commits to the existing feature PR needs no approval after
assignment. Opening a second PR for the same feature in the same repo is forbidden.
Scope beyond the plan → Angelo. Never merge.
