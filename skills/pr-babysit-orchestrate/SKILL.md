---
name: pr-babysit-orchestrate
description: >-
  Use when Angelo or Gene assigns a feature PR to Raye or Grace, either reports
  status, or the shared test slot must be granted among Margaret, Garman, Grace,
  Raye, Susan Kare, and Jean Bartik
---
# pr-babysit-orchestrate

## WHEN TO USE

- Angelo asks Gene (or Wernher when Gene is offline) to babysit a feature
  implementation PR (provides PR number or URL).
- Raye or Grace reports MERGE-READY, BLOCKED, or WAITING and needs relay to Angelo.
- Grace, Raye, Margaret, Garman, Susan Kare, or Jean Bartik requests the shared
  test slot (Kare/Bartik only when they run Pest).
- Daily brief includes a PR readiness row.
- Feature waterfall: babysit starts only after Aaron PASS on the plan and the
  first implementation PR exists.

Gene or Wernher never runs pr-babysit-loop himself. He assigns Raye or Grace,
arbitrates the test slot, and relays outcomes. He never merges.

## REQUIRED INPUTS AND ACCESS

- owner/repo + PR number or URL, branch name (from Angelo or `gh pr view`).
  Allowed: `CorewareHub/coreware-app-backend` (base `develop`) or
  `CorewareHub/boss-control-tower` (base `develop/develop`).
- **DEV tips** (environments.md): boss-control-tower uses `develop/<feature-slug>`
  on https://dev.coreware.app — never tip-push experiments onto `develop/develop`
  (main). Backend DEV primary tenant is
  https://development-corestore-alpha.coreware.app with tips `dev-test/<feature>`
  (not `develop/<feature>`); base remains `develop`.
  https://coreware.coreware.app is PRODUCTION — never preview/experiments there.
- Raye or Grace's current state from their last report (or assign fresh).
- Engineer phase state from qa-phase-orchestrate (for slot conflicts; backend CI only).
- Direct message to the assignee. Read-only gh for PR status on the chosen repo.

## TEST SLOT QUEUE

The slot covers test-running work on the shared Grok Bot cloud computer.
**Angelo standing rule 2026-09-11:** Margaret, Garman, Grace, Raye, Susan Kare, and
Jean Bartik all contend for **one** slot (GRANTED / QUEUED / RELEASED). Garman still
verifies on token 9 (`test_tenant_9` / `test_landlord_9`) but must queue here — he
is **not** outside the queue. Kare and Bartik join when they run Pest. Slot
independence after the `scripts/test-lib.sh` ephemeral-sweep fix stays suspended
until Angelo explicitly lifts the standing rule. Gene records that the standing
rule is in force; Wernher grants when Gene is offline.

**The slot covers test-running work only** — Pest, migrate, or
`test:generate-schema-dump`, i.e. any delegation whose VERIFY is not `none`.
This matches the concurrency rule in repo-delegate-to-cursor; keep the two in step.

Never hold Margaret or Garman off non-Pest phase work (implement, prep, fold, `cursor review`) because the slot is busy or a merge is pending — only Pest/migrate/schema-dump wait on GRANTED.

| State | Meaning |
|-------|---------|
| GRANTED | Holder may run composer test:single |
| QUEUED | Request recorded; wait until current holder releases |
| RELEASED | Holder finished; Gene/Wernher may grant to next queued request |

**Priority:** Margaret's phase verification takes precedence when she is actively
implementing and holds an open remediation loop. Garman queues with the others under
the standing rule. Raye/Grace queue behind phase work unless Angelo explicitly
prioritizes the feature PR.

**Rules:**
- Gene or Wernher grants explicitly — never assume GRANTED.
- Holder must message RELEASED when tests finish (pass or fail).
- If Raye/Grace is QUEUED and blocked on tests, they post WAITING — they do not run
  tests until GRANTED.
- Never grant two holders simultaneously on token 1.
- **No slot needed for VERIFY `none` work.** Bugbot sweeps, plan reads, and verdict
  posting run in parallel alongside engineer phase work. Do not queue read-only work.

## SEQUENCE OF WORK

1. **Assign.** When Angelo provides a PR (or the feature waterfall reaches babysit):
   - Confirm owner/repo is allowed and PR is open, targets the matching base
     (`develop` or `develop/develop`), and is a feature implementation PR (not a CI
     phase PR, not a docs/*-FAILING-TESTS-FIX-PLAN.markdown plan PR, not a feature
     plan PR awaiting Aaron).
   - Feature waterfall: confirm Aaron PASS on the plan and that the first
     implementation PR exists before starting babysit.
   - If Raye or Grace already babysits another PR, STOP — one PR at a time across
     both. Finish or hand back the current one first.
   - Message the assignee (Raye or Grace): owner/repo, base, PR number, URL, branch,
     any implementation plan path Angelo mentions. Tell them to run pr-babysit-loop.

2. **Slot requests.** When a slot participant asks for the test slot:
   - If slot is free → GRANT to requester, record holder name.
   - If held → QUEUE the requester, tell them who holds it.
   - On RELEASED → grant to next QUEUED if any, else slot is free.

3. **Status relay.** When Raye or Grace posts a verdict comment:
   - **MERGE-READY:** Notify Angelo with PR URL, comment URL, reviewed-sha,
     final-sha. Remind him to merge manually and that the plan file is removed from
     the branch. Assignee is idle for the next assignment.
   - **BLOCKED:** Relay blocker to Angelo with comment URL. Assignee waits or handback.
   - **WAITING:** Note in daily brief; no Angelo ping unless WAITING exceeds one hour
     on cursor[bot] or CI.

4. **Separation from CI pipeline.** Never assign Raye/Grace a fix/ci-tests-phase-*
   branch or docs plan PR. Never assign a NASA engineer a feature PR Raye/Grace owns.
   Katherine audits code PRs (and NASA phase PRs); Raye/Grace babysit feature
   implementation PRs — different tracks.

5. **Daily brief row.** Add to Gene's brief:
   ```
   Raye|Grace: <owner/repo> PR #N (<branch> → <base>) — <state> — <verdict or blocker> — <next action>
   ```

## HOW TO VALIDATE

- At most one feature implementation PR assigned to Raye or Grace at a time.
- At most one test slot GRANTED at a time on token 1.
- Garman, Kare, and Bartik appear in the queue while the 2026-09-11 standing rule
  is in force when they need Pest.
- MERGE-READY relay includes comment URL and both SHAs when plan was retired.
- Raye/Grace never assigned to CI phase work.

## WHAT TO RETURN

Assignee (Raye or Grace), PR, branch, state, test slot holder (or free), Garman/Kare/Bartik
slot state when relevant, relay messages sent to Angelo, daily brief line.

## WHAT REQUIRES APPROVAL

Assigning Raye/Grace and granting the test slot need no approval. Never merge.
Escalate to Angelo when Raye/Grace and Margaret both need the slot urgently and
priority is unclear.
