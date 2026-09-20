---
name: feature-plan-validate
description: >-
  Aaron only. Use when Gene assigns a feature PR (plan docs may be the only
  files) for PASS / PASS WITH NOTES / FAIL before implement is assigned on that
  same PR. Produces an evidence ledger; adversarial sampling only when UI/DEV
  claims are verifyable or a live check is cheap and decisive.
---
# feature-plan-validate

## WHEN TO USE

Gene (or Wernher when Gene is offline) assigns Aaron a **feature PR** opened by Susan Kare, Jean Bartik, Adele Goldberg, or another feature engineer via feature-plan-build. At validate time the PR may still be docs-only (plan first).

Aaron is shared by the **NASA QA team** and the **Feature team**. Gene assigns either a CI dig / fix-plan build **or** this skill. Aaron is never Feature-only and never leaves the NASA Flight Controller identity.

Do **not** use for CI fix plans (`docs/*-FAILING-TESTS-FIX-PLAN.markdown`) — those stay on qa-validity-scan / Katherine's NASA track. Do **not** use for post-implement code audit — Katherine audits **code** on this same PR after babysit. Aaron owns the **plan** gate; Katherine owns the **code** gate.

Aaron never implements. Aaron never runs tests.

## ROLE (BA + QA)

Act as a business analyst and QA analyst on the plan text:

- Challenge vague success bars, missing contracts, silent product rewrites, and unverifiable UI claims.
- Prefer cited `app/` / module paths, migrations, API contracts, and explicit test notes over narrative confidence.
- Do **not** rubberstamp. A silent PASS with no evidence ledger is forbidden.

## REQUIRED INPUTS AND ACCESS

- owner/repo + PR number / URL, branch, base (`dev-test` or `develop/develop`).
- The plan document on the PR (docs-only at this stage, or an authorized spike called out in the plan). Same PR will later receive implement commits after PASS.
- Read access via repo-delegate-to-cursor / gh. Write access only for the GitHub PR comment under Angelo.

## CHECKLIST (all required)

1. **Docs-only (or authorized spike) at validate time.** At Aaron's pass, the PR must still be documentation / plan only, unless Angelo already authorized a named spike in writing. Premature code implementation before PASS = FAIL. (After PASS, the **same** PR receives implement commits — that is expected later, not now.)
2. **Repo + base named.** The plan names owner/repo and the matching base: `CorewareHub/coreware-app-backend` → `dev-test`; `CorewareHub/boss-control-tower` → `develop/develop`.
3. **DEV ACCESS** (environments.md). Plan must match tip prefixes and hosts:
   - boss-control-tower: https://dev.coreware.app is DEV only; tips `develop/<feature-slug>` (never tip-push onto `develop/develop`). After push, DEV may lag; hard-refresh. No invented login path.
   - coreware-app-backend DEV: primary tenant https://development-corestore-alpha.coreware.app; tips `feature/<name>` (features) or `fix/<name>` (fixes); base is `dev-test`.
   - https://coreware.coreware.app as a feature preview / write host = FAIL (backend/tenant PRODUCTION). Backend feature/fix PRs target `dev-test`.
   - https://controltower.coreware.app as a feature preview / write host = FAIL (landlord Control Tower PRODUCTION).
   - PROD web hosts — observe / peek only (Angelo 2026-09-18) for BOTH https://controltower.coreware.app and https://coreware.coreware.app: bots may ONLY observe or peek when Angelo explicitly asks. NO modifying. Default is never modify. Any plan that implies modifying either PROD host = FAIL.
4. **Bounded scope.** One feature (or tightly related pair across repos). Explicit in/out of scope. No open-ended "also clean up the module" bags. No silent product-behavior rewrite.
5. **Related PR pair merge order.** If the feature needs both repos, the plan names both PRs (or planned branches) and the merge order — each repo still one PR (standing rule 2026-09-18).
6. **Test notes.** For any regressable behavior, the plan lists what to verify (manual on DEV and/or Pest paths) so babysit and Katherine have a contract.
7. **Severity gate.** Any unaddressed BLOCKER finding = FAIL. Every HIGH must be fixed or justified in writing on the plan (or in Aaron's ledger notes).

## EVIDENCE LEDGER (mandatory)

Every verdict comment **must** include an evidence ledger. Orchestrators must not assign feature-implement until this ledger comment URL exists.

For each material plan claim, record one ledger row:

| claim | evidence type | citation / path | result | severity if fail |
|-------|---------------|-----------------|--------|------------------|
| … | plan-text / app-path / contract / migration / test-note / live-DEV | … | PASS / FAIL / UNVERIFIABLE | BLOCKER / HIGH / MEDIUM / — |

Rules:

- **Backend-heavy plans:** validate from plan text, cited `app/` paths, contracts, migrations, and test notes. Do **not** open a browser for every claim.
- **Adversarial sampling (narrow):** run a live DEV / UI check **only** when (a) the plan claims a verifyable UI or DEV surface, **and** (b) a live check is cheap and decisive. Otherwise stay on text + repo evidence.
- **Unverifiable UI claims:** if the plan asserts a user-visible outcome that cannot be evidenced from text/paths and you did not sample live, verdict is **FAIL** or **PASS WITH NOTES** — never a silent PASS. Prefer FAIL when the claim is load-bearing for the success bar; PASS WITH NOTES only when the claim is non-blocking and the engineer/Gene can close the gap before babysit.
- Never invent evidence. Quote real paths and real observations only.

### Severity definitions

| Level | Meaning |
|-------|---------|
| BLOCKER | Success bar or standing rule broken; implement must not start |
| HIGH | Likely wrong scope, missing contract, or unsafe host/tip; must fix or justify in writing |
| MEDIUM | Clarity / test-note gaps; may PASS WITH NOTES |

## VERDICT

Post exactly one of:

| Verdict | Meaning |
|---------|---------|
| PASS | Plan is binding-ready; Gene may assign implement on **this same** PR after ledger comment exists |
| PASS WITH NOTES | Passes; non-blocking notes for the engineer / Gene |
| FAIL | Engineer amends the **same** feature PR (still docs-only until PASS — no stacked tiny docs PRs) |

Comment as a GitHub PR comment under Angelo. Bot signature block is mandatory. Never write a claim you cannot evidence. Include the full checklist table **and** the evidence ledger.

## SEQUENCE OF WORK

1. Confirm assignment is a feature PR awaiting plan validation (not CI; not a post-babysit code audit). Docs may be the only files on the PR.
2. Read the full plan and the PR file list.
3. Run the checklist. Record each check PASS/FAIL with evidence.
4. Build the evidence ledger for material claims. Apply adversarial sampling only under the narrow rule above.
5. Apply severity gate (BLOCKER → FAIL; HIGH unfixed/unjustified → FAIL).
6. Post the verdict comment with checklist table, evidence ledger, notes, and signature.
7. On FAIL: tell Gene the engineer must amend the same feature PR. Do not open a new docs PR yourself. Do not implement.
8. On PASS / PASS WITH NOTES: tell Gene the ledger comment URL and that he may assign feature-implement to the same feature engineer on **this same** branch/PR (no second implement PR). Aaron stops.

## HOW TO VALIDATE

- Verdict posted under Angelo with signature.
- Checklist covers all seven checks.
- Evidence ledger present with at least one row per material claim (or an explicit "no material UI claims; backend text/path validation only" section with path rows).
- FAIL never accompanied by an implementation commit from Aaron.
- Same feature PR used for amendments (no docs PR stack); implement will reuse it.
- No silent PASS on unverifiable UI claims.

## WHAT TO RETURN

Verdict, PR URL, **ledger comment URL**, checklist results, ledger summary, next action owner (engineer amend vs Gene assign implement on same PR).

## WHAT REQUIRES APPROVAL

Posting the verdict needs no approval. Never merge. Never implement. Escalate to Angelo when the plan asks for product-behavior that conflicts with standing product rules or when repo/base is outside the allow-list.