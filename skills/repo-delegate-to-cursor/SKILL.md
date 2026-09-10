---
name: repo-delegate-to-cursor
description: >-
  Use this when work must read, write, or run commands inside
  CorewareHub/coreware-app-backend
---
# repo-delegate-to-cursor

## WHEN TO USE

Any time work must read, write, or run commands inside CorewareHub/coreware-app-backend. Grok Bot never edits the repo directly. It launches a Cursor Cloud Agent on Composer 2.5 with fast mode enabled and supervises it.

## REQUIRED INPUTS AND ACCESS

- A task brief, self-contained: the agent cannot see the Grok Bot conversation.
- Base branch (default: develop) and the exact target branch name.
- Cursor Cloud Agents access.
- gh authenticated for CorewareHub/coreware-app-backend.

## SEQUENCE OF WORK

1. Launch with model `composer-2.5` and `fast: true` (the current launcher representation of Composer 2.5 Fast). Verify the actually served model is Composer 2.5 Fast. If fast mode is disabled, the launcher substitutes another model, or the served model differs, STOP and report to Angelo.

2. Write the brief with all six sections in this order:

   GOAL — one sentence, the outcome.

   BRANCH — base branch and the branch name to create. **Existing-branch mode:**
   when the work is remediation on a branch that already exists (Grace on an
   assigned feature PR), state the existing branch to check out and say
   explicitly "do not create a new branch". Everything else in this skill applies
   unchanged.

   SCOPE — the explicit file list or phase table from the plan doc.

   CONSTRAINTS — paste verbatim: never commit on develop/main/master; never merge a PR; never run composer format; one test command at a time; follow .cursor/rules/codebase.mdc, .cursor/rules/test-isolation.mdc, and .cursor/rules/test-failure-triage.mdc; scaffolding failures: fix the test; contract failures: do not invert assertions to match a bug, escalate, write an ESCALATED plan row, leave the test red; do not patch `app/` unless Angelo assigned that specific bug. "No app changes" means escalate, not invert. Framework-semantics corrections (PR 5778) are allowed; product-behavior rewrites (PR 5785 class) are not.

   **Grace's feature-PR exception:** on a feature PR she was assigned, the `app/`
   restriction above is replaced by: edit only files already in
   `git diff develop...HEAD --name-only` on that PR; new files or product-behavior
   changes beyond the committed implementation plan require Angelo's go-ahead.
   Every other constraint stands verbatim. Do not apply this exception to CI phase
   work — Margaret's `app/` restriction is unchanged.

   VERIFY — the exact verification commands to run, or `none` for read-only/planning work.

   REPORT BACK — files changed, tests now passing, commands run with real output, anything unfixed and why, ESCALATED rows.

3. Launch one agent from the stated base branch with model `composer-2.5` and fast mode enabled.

4. Supervise until it finishes. **Concurrency (Angelo 2026-09-09):** Exactly one agent may run **tests / migrate / schema dump** on this repo at a time (shared `test_tenant_1` / `test_landlord_1` — a second Pest run silently corrupts both). Margaret owns that slot when she is implementing. Grace contends for the same slot when remediating a feature PR with a non-`none` VERIFY; Gene arbitrates between them (GRANTED / QUEUED / RELEASED) and Margaret takes precedence unless Angelo prioritizes the feature PR. Aaron, Bill, and Grace **may** launch additional agents **in parallel** only when VERIFY is `none` (read-only / planning / docs plan PRs / Grace's bugbot sweeps — no Pest, no migrate, no `test:generate-schema-dump`). Never launch a second **test-running** agent while another test-running agent is live.

5. Capture the summary, branch name, and diff.

## HOW TO VALIDATE

- The run reports Composer 2.5 Fast as the serving model.
- The branch exists and is not develop/main/master. In create mode it came from
  the stated base; in existing-branch mode it is the branch named in BRANCH and no
  new branch was created.
- Every touched file is inside SCOPE. Out-of-scope edits are a finding, not a bonus.
- No polarity inversion. No new application-behavior change unless Angelo already approved that specific patch.
- VERIFY output is quoted real output, not a claim that it passed.
- If VERIFY is not `none`, confirm no other test-running agent was live for this repo.

## WHAT TO RETURN

Branch name, serving model, files changed with line counts, verify output, unresolved items, ESCALATED rows, and any constraint the agent violated.

## WHAT REQUIRES APPROVAL

Pushing and opening a PR need no approval. Escalate to Angelo when the served model is not Composer 2.5 Fast, or when the scope includes `app/` rather than tests/ without a written go-ahead. Never merge a PR.
