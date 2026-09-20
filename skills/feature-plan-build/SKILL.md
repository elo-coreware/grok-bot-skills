---
name: feature-plan-build
description: >-
  Feature engineer opens or updates ONE feature branch/PR; plan docs land first;
  write plans Aaron can ledger (cited paths, success bar, test notes); stop until
  Aaron PASS — no later separate implement PR
---
# feature-plan-build

## WHEN TO USE

Susan Kare, Jean Bartik, Adele Goldberg, or another assigned feature engineer
reaches ~80% detail completeness on an Angelo-requested feature. Write the plan and
open or update **one** feature PR with plan docs first. Stop. Do not implement until
Aaron PASSes feature-plan-validate and Gene (or Wernher) assigns feature-implement
on **this same** branch/PR.

## REQUIRED INPUTS AND ACCESS

- Feature brief from Angelo / Gene (goal, repos, constraints, open questions
  resolved enough to plan).
- owner/repo + base:
  - `CorewareHub/coreware-app-backend` → `develop`
  - `CorewareHub/boss-control-tower` → `develop/develop`
- repo-delegate-to-cursor for docs writes and PR open.
- Never commit on develop / develop/develop / main / master.

## ONE FEATURE PR RULE (Angelo standing rule 2026-09-18)

Exactly **one** feature PR per feature per repo for the whole waterfall (plan →
implement → Bugbot → code validation/audit). Plan docs land first on that branch/PR.
After Aaron PASS, the same engineer commits implement on the **same** branch/PR —
do **not** expect or open a later separate implement PR in this repo. Amend the same
PR tip if Aaron FAILs or details change. Do not open stacked tiny docs PRs.

A related pair in the **other** repo (backend vs Control Tower) is a separate PR
there — name it under Related PR pair.

## PLAN CONTENTS (minimum)

1. **Goal** — one paragraph outcome, plus a crisp **success bar** (what must be true
   on DEV or in code for the feature to count as done).
2. **owner/repo + base** — explicit.
3. **DEV ACCESS** (environments.md) —
   - https://dev.coreware.app is **boss-control-tower DEV only**. Feature tips:
     `develop/<feature-slug>`. Never tip-push experiments onto `develop/develop`
     (main). After push, DEV may lag (ECS/roll); hard-refresh and re-check. Do not
     invent a login click-path or passwords; login wall → Angelo / takeover. Never
     paste credentials.
   - https://development-corestore-alpha.coreware.app is the **primary tenant for
     DEV coreware-app-backend**. Feature tips: `dev-test/<feature>` — not
     `develop/<feature>`. Base/main remains `develop`.
   - https://coreware.coreware.app is **coreware-app-backend PRODUCTION** (tenant/backend app PROD — different from Control Tower landlord PROD).
     Backend feature PRs still target `develop`.
   - https://controltower.coreware.app is **landlord Control Tower PRODUCTION**.
   - PROD web hosts — observe / peek only (Angelo 2026-09-18) for BOTH https://controltower.coreware.app (landlord Control Tower PRODUCTION) and https://coreware.coreware.app (backend/tenant PRODUCTION): bots may ONLY observe or peek when Angelo explicitly asks. NO modifying — no edits, creates, deletes, status changes, state-changing comments, form submits, deploys, tip-pushes, or write APIs. Default is never modify.
4. **Scope** — in / out. Bounded. No silent product-behavior rewrite.
5. **Approach** — high-level steps, **cited files or surfaces** (`app/...`, routes,
   migrations, contracts, Blade/Vue paths as applicable), risks.
6. **Related PR pair** — if both repos, name merge order (each repo still one PR).
7. **Test notes** — regressable behavior and how to verify (DEV and/or Pest paths).
8. **Open questions** — only residual items that do not block planning; escalate product-behavior ambiguity to Angelo before opening if critical.

Prefer docs-only at this stage. An authorized spike needs Angelo's written go-ahead
named in the plan.

## AARON LEDGER EXPECTATIONS (write for his gate)

Aaron runs `feature-plan-validate` as a business analyst + QA analyst. Every PASS
needs an **evidence ledger**. Write the plan so he can fill that ledger without
guessing:

| He will look for | Put in the plan |
|------------------|-----------------|
| Load-bearing claims | Explicit success bar; named UI/DEV routes or API behaviors |
| Backend / code claims | Cited `app/` (or module) paths, migrations, contracts — not narrative only |
| UI / DEV claims | Exact DEV host + tip prefix + path (or state the claim is not verifyable yet) |
| Regressable behavior | Test notes (manual DEV steps and/or Pest paths) |
| Scope discipline | Clear in/out; no PROD write hosts; correct tip prefixes |

Rules Aaron applies (do not fight them in the plan):

- **Backend-heavy:** he validates from plan text + cited paths/contracts/migrations/test notes — he will **not** browse for every claim.
- **UI/DEV:** he samples live only when the claim is verifyable **and** a live check is cheap/decisive. Unverifiable user-visible claims → FAIL or PASS WITH NOTES, never a silent PASS.
- **BLOCKER / HIGH** findings block PASS until fixed or justified in writing on the same PR tip.
- Gene will not assign `feature-implement` until Aaron's verdict comment includes the ledger (and a comment URL).

If you cannot cite a path or success bar yet, say so under Open questions and escalate — do not paper over it with confident prose.

## SEQUENCE OF WORK

1. Confirm ~80% details and owner/repo + base with Gene.
2. Draft the plan document on a feature branch (not the base). Prefer reusing an
   existing open feature PR tip for this feature if one already exists. Meet the
   Aaron ledger expectations above.
3. Open or update **one** feature PR targeting the correct base (plan docs first).
4. Notify Gene that the feature PR is ready for Aaron feature-plan-validate.
5. **STOP.** Do not implement. Do not request babysit. Wait for Aaron PASS + Gene
   assign implement on **this same** PR. Do not open a second PR for implement.
   If Aaron FAILs, amend this same tip and re-request validate.

## HOW TO VALIDATE

- Single feature PR for this feature in this repo; docs-only at plan stage (or named
  authorized spike).
- Repo, base, DEV ACCESS (four hosts / tip prefixes), scope, success bar, cited
  paths, and test notes present (ledger-ready).
- No premature implementation commits before Aaron PASS + Gene assign implement.
- No expectation of a later separate implement PR for implement in the same repo.

## WHAT TO RETURN

Plan path, PR URL, branch, owner/repo + base, next action: wait for Aaron (same PR).

## WHAT REQUIRES APPROVAL

Opening or updating the feature PR needs no approval. Product-behavior beyond the brief → Angelo. Never merge. Never start feature-implement without Aaron PASS + Gene
assignment. Never open a second PR for the same feature in the same repo.
