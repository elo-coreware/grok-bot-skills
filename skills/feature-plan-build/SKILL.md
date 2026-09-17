---
name: feature-plan-build
description: >-
  Feature engineer writes one feature plan PR after ~80% of details are filled,
  then stops until Aaron PASS and Gene assigns implement
---
# feature-plan-build

## WHEN TO USE

Susan Kare, Jean Bartik, or another assigned feature engineer reaches ~80% detail
completeness on an Angelo-requested feature. Write the plan and open **one** plan
PR. Stop. Do not implement until Aaron PASSes feature-plan-validate and Gene (or
Wernher) assigns feature-implement.

## REQUIRED INPUTS AND ACCESS

- Feature brief from Angelo / Gene (goal, repos, constraints, open questions
  resolved enough to plan).
- owner/repo + base:
  - `CorewareHub/coreware-app-backend` → `develop`
  - `CorewareHub/boss-control-tower` → `develop/develop`
- repo-delegate-to-cursor for docs writes and PR open.
- Never commit on develop / develop/develop / main / master.

## ONE PLAN PR RULE

Exactly one plan PR per feature. Amend the same PR tip if Aaron FAILs or details
change. Do not open stacked tiny docs PRs.

## PLAN CONTENTS (minimum)

1. **Goal** — one paragraph outcome.
2. **owner/repo + base** — explicit.
3. **DEV ACCESS** — https://dev.coreware.app is **boss-control-tower DEV only**.
   Push `develop/<feature-slug>` on CorewareHub/boss-control-tower (that branch is
   what DEV serves). Never tip-push experiments onto `develop/develop` (main). After
   push, DEV may lag (ECS/roll); hard-refresh and re-check. Do not invent a login
   click-path or passwords; login wall → Angelo / takeover. Never paste credentials.
   https://coreware.coreware.app is **coreware-app-backend PRODUCTION** — do not use
   it to preview feature work; do not run experiments / schema dumps / tip-pushes
   against it. Backend feature PRs still target `develop`. No backend DEV host is
   named here — do not invent one.
4. **Scope** — in / out. Bounded.
5. **Approach** — high-level steps, files or surfaces touched, risks.
6. **Related PR pair** — if both repos, name merge order.
7. **Test notes** — regressable behavior and how to verify (DEV and/or Pest).
8. **Open questions** — only residual items that do not block planning; escalate
   product-behavior ambiguity to Angelo before opening if critical.

Prefer docs-only. An authorized spike needs Angelo's written go-ahead named in the
plan.

## SEQUENCE OF WORK

1. Confirm ~80% details and owner/repo + base with Gene.
2. Draft the plan document on a feature branch (not the base).
3. Open one plan PR targeting the correct base.
4. Notify Gene that the plan PR is ready for Aaron feature-plan-validate.
5. **STOP.** Do not implement. Do not request babysit. Wait for Aaron PASS + Gene
   assign implement.

## HOW TO VALIDATE

- Single plan PR; docs-only (or named authorized spike).
- Repo, base, DEV ACCESS (dev.coreware.app vs PROD), scope, test notes present.
- No implementation commits on the plan PR.

## WHAT TO RETURN

Plan path, PR URL, branch, owner/repo + base, next action: wait for Aaron.

## WHAT REQUIRES APPROVAL

Opening the plan PR needs no approval. Product-behavior beyond the brief → Angelo.
Never merge. Never start feature-implement without Aaron PASS + Gene assignment.
