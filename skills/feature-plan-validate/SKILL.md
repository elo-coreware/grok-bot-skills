---
name: feature-plan-validate
description: >-
  Aaron only. Use when Gene assigns a feature PR (plan docs may be the only files)
  for PASS / PASS WITH NOTES / FAIL before implement is assigned on that same PR
---
# feature-plan-validate

## WHEN TO USE

Gene (or Wernher when Gene is offline) assigns Aaron a **feature PR** opened by
Susan Kare, Jean Bartik, Adele Goldberg, or another feature engineer via
feature-plan-build. At validate time the PR may still be docs-only (plan first).

Do **not** use for CI fix plans (`docs/*-FAILING-TESTS-FIX-PLAN.markdown`) — those
stay on qa-validity-scan / Katherine's NASA track. Do **not** use for post-implement
code audit — Katherine audits code on this same PR after babysit.

Aaron never implements. Aaron never runs tests.

## REQUIRED INPUTS AND ACCESS

- owner/repo + PR number / URL, branch, base (`develop` or `develop/develop`).
- The plan document on the PR (docs-only at this stage, or an authorized spike called
  out in the plan). Same PR will later receive implement commits after PASS.
- Read access via repo-delegate-to-cursor / gh. Write access only for the GitHub
  PR comment under Angelo.

## CHECKLIST (all required)

1. **Docs-only (or authorized spike) at validate time.** At Aaron's pass, the PR must
   still be documentation / plan only, unless Angelo already authorized a named spike
   in writing. Premature code implementation before PASS = FAIL. (After PASS, the
   **same** PR receives implement commits — that is expected later, not now.)
2. **Repo + base named.** The plan names owner/repo and the matching base:
   `CorewareHub/coreware-app-backend` → `develop`;
   `CorewareHub/boss-control-tower` → `develop/develop`.
3. **DEV ACCESS** (environments.md). Plan must match tip prefixes and hosts:
   - boss-control-tower: https://dev.coreware.app is DEV only; tips
     `develop/<feature-slug>` (never tip-push onto `develop/develop`). After push,
     DEV may lag; hard-refresh. No invented login path.
   - coreware-app-backend DEV: primary tenant
     https://development-corestore-alpha.coreware.app; tips `dev-test/<feature>` —
     not `develop/<feature>`; base remains `develop`.
   - https://coreware.coreware.app as a feature preview host = FAIL (backend/tenant PRODUCTION).
     Never preview features or run experiments there. Backend feature PRs target
     `develop`.
   - https://controltower.coreware.app as a feature preview host = FAIL (landlord Control Tower PRODUCTION).
     Never tip-push experiments or use for feature preview.
4. **Bounded scope.** One feature (or tightly related pair across repos). Explicit
   in/out of scope. No open-ended "also clean up the module" bags.
5. **Related PR pair merge order.** If the feature needs both repos, the plan names
   both PRs (or planned branches) and the merge order — each repo still one PR
   (standing rule 2026-09-18).
6. **Test notes.** For any regressable behavior, the plan lists what to verify
   (manual on DEV and/or Pest paths) so babysit and Katherine have a contract.
7. **Severity gate.** Any unaddressed BLOCKER finding in the plan (or from prior
   Aaron notes on this PR) = FAIL. Every HIGH must be fixed or justified in writing
   on the plan.

## VERDICT

Post exactly one of:

| Verdict | Meaning |
|---------|---------|
| PASS | Plan is binding-ready; Gene may assign implement on **this same** PR |
| PASS WITH NOTES | Passes; non-blocking notes for the engineer / Gene |
| FAIL | Engineer amends the **same** feature PR (still docs-only until PASS — no stacked tiny docs PRs) |

Comment as a GitHub PR comment under Angelo. Bot signature block is mandatory.
Never write a claim you cannot evidence.

## SEQUENCE OF WORK

1. Confirm assignment is a feature PR awaiting plan validation (not CI; not a
   post-babysit code audit). Docs may be the only files on the PR.
2. Read the full plan and the PR file list.
3. Run the checklist. Record each check PASS/FAIL with evidence.
4. Apply severity gate (BLOCKER → FAIL; HIGH unfixed/unjustified → FAIL).
5. Post the verdict comment with checklist table, notes, and signature.
6. On FAIL: tell Gene the engineer must amend the same feature PR. Do not open a new
   docs PR yourself. Do not implement.
7. On PASS / PASS WITH NOTES: tell Gene to assign feature-implement to the same
   feature engineer on **this same** branch/PR (no second implement PR). Aaron stops.

## HOW TO VALIDATE

- Verdict posted under Angelo with signature block.
- Checklist covers all seven checks.
- FAIL never accompanied by an implementation commit from Aaron.
- Same feature PR used for amendments (no docs PR stack); implement will reuse it.

## WHAT TO RETURN

Verdict, PR URL, comment URL, checklist results, next action owner (engineer amend
vs Gene assign implement on same PR).

## WHAT REQUIRES APPROVAL

Posting the verdict needs no approval. Never merge. Never implement. Escalate to
Angelo when the plan asks for product-behavior that conflicts with standing product
rules or when repo/base is outside the allow-list.
