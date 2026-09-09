---
name: qa-delegate-to-cursor
description: >-
  Use this when work must read, write, or run commands inside
  CorewareHub/coreware-app-backend
---
# qa-delegate-to-cursor

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

   BRANCH — base branch and the branch name to create.

   SCOPE — the explicit file list or phase table from the plan doc.

   CONSTRAINTS — paste verbatim: never commit on develop/main/master; never merge a PR; never run composer format; one test command at a time; follow .cursor/rules/codebase.mdc, .cursor/rules/test-isolation.mdc, and .cursor/rules/test-failure-triage.mdc; scaffolding failures: fix the test; contract failures: do not invert assertions to match a bug, escalate, write an ESCALATED plan row, leave the test red; do not patch `app/` unless Angelo assigned that specific bug. "No app changes" means escalate, not invert. Framework-semantics corrections (PR 5778) are allowed; product-behavior rewrites (PR 5785 class) are not.

   VERIFY — the exact verification commands to run, or `none` for read-only/planning work.

   REPORT BACK — files changed, tests now passing, commands run with real output, anything unfixed and why, ESCALATED rows.

3. Launch one agent from the stated base branch with model `composer-2.5` and fast mode enabled.

4. Supervise until it finishes. Never launch a second agent against this repo while one is running — they share the test databases and will corrupt each other.

5. Capture the summary, branch name, and diff.

## HOW TO VALIDATE

- The run reports Composer 2.5 Fast as the serving model.
- The branch exists, is not develop/main/master, and came from the stated base.
- Every touched file is inside SCOPE. Out-of-scope edits are a finding, not a bonus.
- No polarity inversion. No new application-behavior change unless Angelo already approved that specific patch.
- VERIFY output is quoted real output, not a claim that it passed.

## WHAT TO RETURN

Branch name, serving model, files changed with line counts, verify output, unresolved items, ESCALATED rows, and any constraint the agent violated.

## WHAT REQUIRES APPROVAL

Pushing and opening a PR need no approval. Escalate to Angelo when the served model is not Composer 2.5 Fast, or when the scope includes `app/` rather than tests/ without a written go-ahead. Never merge a PR.
