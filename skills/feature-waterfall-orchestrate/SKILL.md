---
name: feature-waterfall-orchestrate
description: >-
  Gene or Wernher. Assign feature plan → Aaron validate → implement → Raye/Grace
  babysit → Katherine audit → Angelo merge; own DEV branch rule and one-slot queue
---
# feature-waterfall-orchestrate

## WHEN TO USE

- Angelo starts or advances a feature on the 2026-09-17 waterfall.
- Gene is primary; Wernher runs this when Gene is offline (backup orchestrator).
- Status brief needs the feature waterfall table.

Neither Gene nor Wernher implements or audits. Aaron validates plans. Katherine
audits code. Angelo merges.

## WATERFALL

1. Angelo asks; details fill in.
2. At ~80% complete, assign the feature engineer (Susan Kare — Control Tower /
   platform-banner / UI-surface; Jean Bartik — Action Plan / service-request
   workflow) to feature-plan-build → one plan PR.
3. Assign Aaron feature-plan-validate (PASS / PASS WITH NOTES / FAIL). Binding on
   the plan. Aaron never implements.
4. On PASS, assign the **same** feature engineer feature-implement.
5. When the first implementation PR exists, assign Raye or Grace
   pr-babysit-orchestrate / pr-babysit-loop (one PR at a time).
6. After babysit MERGE-READY (or as Gene schedules), Katherine audits the code PR.
7. Angelo merges.

On Aaron FAIL: engineer amends the same plan PR; re-validate. No stacked tiny docs
PRs. Do not assign implement.

## REQUIRED INPUTS AND ACCESS

- Feature name, owning engineer, Angelo brief.
- owner/repo + base for each PR:
  - `CorewareHub/coreware-app-backend` → `develop`
  - `CorewareHub/boss-control-tower` → `develop/develop`
- **DEV ACCESS:** https://dev.coreware.app is boss-control-tower DEV only — push
  `develop/<feature-slug>` (that branch is what DEV serves); never tip-push
  experiments onto `develop/develop`. After push, DEV may lag (ECS/roll); hard-refresh.
  Login wall → Angelo / takeover; never invent passwords or paste credentials.
  https://coreware.coreware.app is coreware-app-backend PRODUCTION — not a feature
  preview host. Backend feature PRs target `develop`. No backend DEV host named here.
- Direct messages to Aaron, Kare/Bartik, Raye/Grace, Katherine. Read-only gh.

## TEST SLOT QUEUE

Margaret, Garman, Grace, Raye, Susan Kare, and Jean Bartik share **one** slot
(GRANTED / QUEUED / RELEASED) under standing rule 2026-09-11. Kare and Bartik join
when they run Pest. Wernher grants when Gene is offline. Same rules as
pr-babysit-orchestrate / repo-delegate-to-cursor concurrency.

## STATUS TABLE

Report every active feature as:

| feature | owner | plan PR | Aaron | implement PR | babysit | Katherine | next action |
|---------|-------|---------|-------|--------------|---------|-----------|-------------|

`babysit` is Raye or Grace (or idle). `next action` names one owner.

## SEQUENCE OF WORK

1. Confirm engineer (Kare vs Bartik) and owner/repo + base with Angelo.
2. At ~80%: assign feature-plan-build. Record plan PR.
3. Assign Aaron feature-plan-validate. On FAIL → amend same PR; on PASS → step 4.
4. Assign feature-implement to the same engineer.
5. When first implement PR is up: assign Raye or Grace (one at a time). Pass
   owner/repo + base. Do not start babysit before Aaron PASS + implement PR.
6. After babysit ready: queue Katherine code audit (she has the last word on CODE
   PRs). Relay to Angelo for merge — never merge yourself.
7. Keep NASA CI phase work separate from this track.

## HOW TO VALIDATE

- Aaron verdict before implement assignment.
- Babysit only after implement PR exists; one Raye/Grace PR at a time.
- DEV rule stated on boss-control-tower features.
- Status table complete; single next action.

## WHAT TO RETURN

Status table, slot holder, relay messages to Angelo.

## WHAT REQUIRES APPROVAL

Assignments need no approval. Never merge. Escalate product-behavior disputes and
allow-list expansions to Angelo.
