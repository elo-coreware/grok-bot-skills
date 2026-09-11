---
name: pr-babysit-orchestrate
description: >-
  Use when Angelo assigns a feature PR to Grace, Grace reports status, or the
  shared test slot must be granted between Margaret and Grace
---
# pr-babysit-orchestrate

## WHEN TO USE

- Angelo asks Gene to babysit a feature PR (provides PR number or URL).
- Grace reports MERGE-READY, BLOCKED, or WAITING and needs Gene to relay to Angelo.
- Grace, Margaret, or Garman (pre-fix only) requests the shared test slot.
- Daily brief includes Grace's PR readiness row.

Gene never runs pr-babysit-loop himself. He assigns Grace, arbitrates the test
slot, and relays outcomes. He never merges.

## REQUIRED INPUTS AND ACCESS

- PR number or URL, branch name (from Angelo or `gh pr view`).
- Grace's current state from her last report (or assign fresh).
- Engineer phase state from qa-phase-orchestrate (for slot conflicts).
- Direct message to Grace. Read-only gh for PR status.

## TEST SLOT QUEUE

The slot covers `test_tenant_1` / `test_landlord_1` on the shared Grok Bot cloud
computer. Margaret and Grace contend for it. Garman is isolated on token 9
(`test_tenant_9` / `test_landlord_9`) and is **outside this queue** once Angelo
confirms the ephemeral-sweep scoping fix in `scripts/test-lib.sh` is on develop.
Until then Garman queues here too. Gene records which state is current.

**The slot covers test-running work only** — Pest, migrate, or
`test:generate-schema-dump`, i.e. any delegation whose VERIFY is not `none`.
This matches the concurrency rule in repo-delegate-to-cursor; keep the two in step.

| State | Meaning |
|-------|---------|
| GRANTED | Holder (Margaret, Grace, or Garman pre-fix) may run composer test:single |
| QUEUED | Request recorded; wait until current holder releases |
| RELEASED | Holder finished; Gene may grant to next queued request |

**Priority:** Margaret's phase verification takes precedence when she is actively
implementing and holds an open remediation loop. During the pre-fix period Garman
queues between Margaret and Grace. Grace queues behind both unless Angelo explicitly
prioritizes the feature PR.

**Rules:**
- Gene grants explicitly — never assume GRANTED.
- Holder must message Gene RELEASED when tests finish (pass or fail).
- If Grace is QUEUED and blocked on tests, she posts WAITING — she does not run tests.
- Never grant two holders simultaneously on token 1.
- **No slot needed for VERIFY `none` work.** Grace's pr-bugbot-sweep, plan reads,
  and verdict posting run in parallel alongside engineer phase work, exactly like
  Aaron's and Bill's planning agents. Do not queue read-only work — it stalls the
  loop for no reason.

## SEQUENCE OF WORK

1. **Assign.** When Angelo provides a PR:
   - Confirm PR is open, targets develop, is a feature PR (not a CI phase PR, not
     a docs/*-FAILING-TESTS-FIX-PLAN.markdown plan PR).
   - If Grace already babysits another PR, STOP — one PR at a time. Finish or hand
     back the current one first.
   - Message Grace: PR number, URL, branch, any implementation plan path Angelo
     mentions. Tell her to run pr-babysit-loop.

2. **Slot requests.** When Grace, Margaret, or Garman (pre-fix) asks for the test slot:
   - If slot is free → GRANT to requester, record holder name.
   - If held → QUEUE the requester, tell them who holds it.
   - On RELEASED → grant to next QUEUED if any, else slot is free.

3. **Status relay.** When Grace posts a verdict comment:
   - **MERGE-READY:** Notify Angelo with PR URL, comment URL, reviewed-sha,
     final-sha. Remind him to merge manually and that the plan file is removed from
     the branch. Grace is idle for the next assignment.
   - **BLOCKED:** Relay blocker to Angelo with comment URL. Grace waits or handback.
   - **WAITING:** Note in daily brief; no Angelo ping unless WAITING exceeds one hour
     on cursor[bot] or CI.

4. **Separation from CI pipeline.** Never assign Grace a fix/ci-tests-phase-* branch
   or docs plan PR. Never assign an engineer a feature PR Grace owns. Katherine audits
   phase PRs; Grace babysits feature PRs — different tracks.

5. **Daily brief row.** Add to Gene's five-line brief:
   ```
   Grace: PR #N (<branch>) — <state> — <verdict or blocker> — <next action>
   ```

## HOW TO VALIDATE

- At most one PR assigned to Grace at a time.
- At most one test slot GRANTED at a time on token 1.
- Garman appears in the queue only while the sweep fix is unmerged.
- MERGE-READY relay includes comment URL and both SHAs when plan was retired.
- Grace never assigned to CI phase work.

## WHAT TO RETURN

Grace assignment (PR, branch, state), test slot holder (or free), Garman slot state
(in queue / independent on token 9), relay messages sent to Angelo, daily brief Grace line.

## WHAT REQUIRES APPROVAL

Assigning Grace and granting the test slot need no approval. Never merge. Escalate
to Angelo when Grace and Margaret both need the slot urgently and priority is unclear.
