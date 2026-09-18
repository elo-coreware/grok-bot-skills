---
name: feature-waterfall-orchestrate
description: >-
  Gene or Wernher. Assign feature plan → Aaron validate → implement on the SAME
  PR → Raye/Grace babysit → Katherine audit → Angelo merge; own DEV branch rule
  and one-slot queue
---
# feature-waterfall-orchestrate

## WHEN TO USE

- Angelo starts or advances a feature on the 2026-09-17 waterfall (one-PR rule
  standing 2026-09-18).
- Gene is primary; Wernher runs this when Gene is offline (backup orchestrator).
- Status brief needs the feature waterfall table.

Neither Gene nor Wernher implements or audits. Aaron validates plans. Katherine
audits code. Angelo merges.

## WATERFALL

1. Angelo asks; details fill in.
2. At ~80% complete, assign the feature engineer (Susan Kare — Control Tower /
   platform-banner / UI-surface; Jean Bartik — Action Plan / service-request
   workflow; Adele Goldberg when Gene routes a feature to her) to
   feature-plan-build → **one feature PR** (plan docs land first on that branch/PR).
3. Assign Aaron feature-plan-validate on **that same PR** (PASS / PASS WITH NOTES /
   FAIL). Binding on the plan. Docs may still be the only files at validate time.
   Aaron never implements.
4. On PASS, assign the **same** feature engineer feature-implement on the **same
   branch / same PR**. Do **not** open a separate implement PR in the same repo.
5. When that PR has implementation commits (after Aaron PASS), assign Raye or Grace
   pr-babysit-orchestrate / pr-babysit-loop (one PR at a time) on **that same PR**.
6. After babysit MERGE-READY (or as Gene schedules), Katherine audits **that same
   PR** (code).
7. Angelo merges.

**ONE FEATURE PR (Angelo standing rule 2026-09-18):** plan → implement → Bugbot →
code validation/audit all happen on a single PR per feature per repo. Never open
separate plan and implement PRs for the same feature in the same repo. A related
backend/frontend pair in the **other** repo remains a separate PR (named in the
plan's Related PR pair).

On Aaron FAIL: engineer amends the **same** feature PR (still docs-only until
PASS); re-validate. No stacked tiny docs PRs. Do not assign implement.

## REQUIRED INPUTS AND ACCESS

- Feature name, owning engineer, Angelo brief.
- owner/repo + base for each PR:
  - `CorewareHub/coreware-app-backend` → `develop`
  - `CorewareHub/boss-control-tower` → `develop/develop`
- **DEV ACCESS** (environments.md):
  - https://dev.coreware.app is boss-control-tower DEV only — push
    `develop/<feature-slug>`; never tip-push experiments onto `develop/develop`.
    After push, DEV may lag (ECS/roll); hard-refresh. Login wall → Angelo /
    takeover; never invent passwords or paste credentials.
  - https://development-corestore-alpha.coreware.app is the primary tenant for DEV
    coreware-app-backend — tips `dev-test/<feature>` (not `develop/<feature>`);
    base remains `develop`.
  - https://coreware.coreware.app is coreware-app-backend PRODUCTION (tenant/backend app PROD — different from Control Tower landlord PROD) — never
    preview features or run experiments there. Backend feature PRs target `develop`.
  - https://controltower.coreware.app is landlord Control Tower PRODUCTION — never
    tip-push experiments here. Never use for feature preview.
- Direct messages to Aaron, Kare/Bartik/Goldberg, Raye/Grace, Katherine. Read-only gh.

## TEST SLOT QUEUE

Margaret, Garman, Grace, Raye, Susan Kare, Jean Bartik, and Adele Goldberg share
**one** slot (GRANTED / QUEUED / RELEASED) under standing rule 2026-09-11. Feature
engineers join when they run Pest. Wernher grants when Gene is offline. Same rules
as pr-babysit-orchestrate / repo-delegate-to-cursor concurrency.

## STATUS TABLE

Report every active feature as:

| feature | owner | feature PR | Aaron | implement | babysit | Katherine | next action |
|---------|-------|------------|-------|-----------|---------|-----------|-------------|

`feature PR` is the single PR for that feature in that repo (plan docs first, then
code on the same tip). `implement` is pending / in progress / ready for babysit.
`babysit` is Raye or Grace (or idle). `next action` names one owner.

## SEQUENCE OF WORK

1. Confirm engineer (Kare / Bartik / Goldberg) and owner/repo + base with Angelo.
2. At ~80%: assign feature-plan-build. Record the **one** feature PR URL/branch.
3. Assign Aaron feature-plan-validate on that PR. On FAIL → amend same PR; on PASS →
   step 4.
4. Assign feature-implement to the same engineer on the **same** branch/PR.
5. When implement commits are on that PR: assign Raye or Grace (one at a time). Pass
   owner/repo + base. Do not start babysit before Aaron PASS + implement commits on
   that PR.
6. After babysit ready: queue Katherine code audit on that same PR (she has the last
   word on CODE PRs). Relay to Angelo for merge — never merge yourself.
7. Keep NASA CI phase work separate from this track.

## HOW TO VALIDATE

- Aaron verdict before implement assignment.
- Same PR used for plan docs and implement; no second feature PR in the same repo.
- Babysit only after implement commits exist on that PR; one Raye/Grace PR at a time.
- DEV tip prefixes stated (`develop/<feature-slug>` vs `dev-test/<feature>`); PROD host never used for preview.
- Status table complete; single next action.

## WHAT TO RETURN

Status table, slot holder, relay messages to Angelo.

## WHAT REQUIRES APPROVAL

Assignments need no approval. Never merge. Escalate product-behavior disputes and
allow-list expansions to Angelo.
