---
name: feature-implement
description: >-
  Feature engineer implements after Aaron PASS on the plan PR; Local Pint; request
  test slot before Pest; hand off to Raye or Grace when the first implementation
  PR is up
---
# feature-implement

## WHEN TO USE

Gene (or Wernher) assigns the same feature engineer (Susan Kare / Jean Bartik) to
implement after Aaron's feature-plan-validate is PASS or PASS WITH NOTES on the
plan PR. Do not start on FAIL or without assignment.

## REQUIRED INPUTS AND ACCESS

- Aaron verdict comment URL (PASS / PASS WITH NOTES).
- Plan document (binding scope).
- owner/repo + base (`develop` or `develop/develop`).
- Prefer the **same branch** as the plan when possible; otherwise a clear
  implement branch from the correct base — never from tip-pushed experiments on
  `develop/develop`.
- **DEV ACCESS:** https://dev.coreware.app is boss-control-tower DEV only — push
  `develop/<feature-slug>` on CorewareHub/boss-control-tower (that branch is what
  DEV serves; never tip-push experiments onto `develop/develop`). After push, DEV
  may lag (ECS/roll); hard-refresh and report what you actually see. Do not invent
  a login path or passwords; login wall → Angelo / takeover. Never paste credentials.
  https://coreware.coreware.app is coreware-app-backend **PRODUCTION** — not a
  feature preview host; do not experiment against it. Backend feature PRs target
  `develop`. No backend DEV host is named here — do not invent one.
- repo-delegate-to-cursor. Local Pint after PHP edits. Shared test slot before Pest.

## SEQUENCE OF WORK

1. Confirm Aaron PASS and Gene/Wernher implement assignment.
2. Check out the plan branch (or create implement branch from the correct base).
3. Implement only what the plan describes. Escalate product-behavior beyond the
   plan to Angelo — do not expand scope quietly.
4. **Local Pint:** after PHP edits, `./vendor/bin/pint --dirty` then
   `./vendor/bin/pint --test`. Commit style fixes on the same branch. Never
   `composer format`. Docs-only / non-PHP may skip.
5. **Test slot:** before any Pest / migrate / schema-dump, request GRANTED from
   Wernher (or Gene). Queue with Margaret / Garman / Grace / Raye / Kare / Bartik
   (standing rule 2026-09-11). Release when done.
6. Push and open (or update) the first implementation PR targeting the correct base.
7. **Hand off:** unless Gene says otherwise, notify Gene that the first
   implementation PR is up so Raye or Grace can babysit (pr-babysit-orchestrate).
   Do not babysit your own PR end-to-end unless Gene explicitly keeps you on it.
8. Never merge. Katherine audits the code PR after babysit; Angelo merges.

## HOW TO VALIDATE

- Aaron PASS on record before first implement commit (after assignment).
- Scope matches plan; Pint clean on PHP; slot requested before Pest.
- Implementation PR open; Gene notified for Raye/Grace babysit.
- Never merged.

## WHAT TO RETURN

PR URL, branch, owner/repo + base, Pint status, tests run (if any) with slot
evidence, handoff note to Gene.

## WHAT REQUIRES APPROVAL

Pushing and opening the implementation PR need no approval after assignment.
Scope beyond the plan → Angelo. Never merge.
