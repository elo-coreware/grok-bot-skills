---
name: pr-babysit-orchestrate
description: >-
  Use when Angelo assigns a feature PR to Grace, Grace reports status, or the
  shared test slot must be granted between Margaret, Garman, and Grace
---
# pr-babysit-orchestrate

## WHEN TO USE

- Angelo asks Gene to babysit a feature PR (provides PR number or URL).
- Grace reports MERGE-READY, BLOCKED, or WAITING and needs Gene to relay to Angelo.
- Grace, Margaret, or Garman (pre-fix only) requests the shared test slot.
- Daily brief includes Grace's PR readiness row.

Gene never runs pr-babysit-loop himself. He assigns Grace, arbitrates the test
slot, and relays outcomes. He never merges.

**Angelo standing rule 2026-09-21 (via Gene; Pint tightened 2026-09-23):** Bot PR verify / MERGE-READY must **not** wait on full self-hosted CI Tests (~60 min). Gate = Bugbot CLEAN==HEAD + **full-repo** Pint/lint (`pint --test` or Check Code Style `lint (8.3)` green on HEAD — never `--dirty` / path-scoped-only) + touched tests only under Gene Pest GRANT. Ambient full-suite **Tests** red ≠ babysit blocker unless tip-caused. Full-repo **Pint** red **is** a babysit blocker — fix on the tip until green. WAITING = Bugbot or Pest slot only — never full-suite Tests pending.

**Angelo standing rule 2026-09-21 (cursor-review pending gate; example CorewareHub/coreware-app-backend#6487):** Before posting `cursor review` / `bugbot run`, check HEAD SHA, bare invoke comments, and cursor[bot] reviews. If HEAD is already reviewed, or a prior invoke is still PENDING (no later cursor[bot] review covering HEAD), stay WAITING-BUGBOT — do **not** post another invoke. Never a second `cursor review` for the same HEAD SHA. See pr-babysit-loop CURSOR REVIEW INVOKE GATE.

## BACKEND BRANCHING (Angelo 2026-09-21, clarified)

For `CorewareHub/coreware-app-backend`:

- **Tip names:** `feature/<name>` for features; `fix/<name>` for fixes. Do **not** use tip prefix `dev-test/<name>` as the default.
- **Normal PR base:** `develop`.
- **`dev-test` is situational:** use it only when you need to run tests, or need the change to reflect on coreware-app-backend DEV (primary tenant https://development-corestore-alpha.coreware.app). Do not make every backend PR target `dev-test`.
- Never tip-push experiments onto a protected base. Never commit on `develop`, `dev-test`, `main`, or `master`.

## CONTROL TOWER BRANCHING (Angelo 2026-09-22)

For `CorewareHub/boss-control-tower`:

- **Tip names:** `feature/<name>` for features; `fix/<name>` for fixes. Do **not** default tip prefix to `develop/<feature-name>`.
- **Normal PR base:** `develop/develop` (Control Tower main).
- **`develop/<feature-name>` is situational:** use it only when you need **visual confirmation** on DEV (https://dev.coreware.app) — same idea as backend using `dev-test` only for tests/DEV reflection.
- Never tip-push experiments onto `develop/develop`.

## REQUIRED INPUTS AND ACCESS

- owner/repo + PR number or URL, branch name (from Angelo or `gh pr view`).
  Allowed: `CorewareHub/coreware-app-backend` (base `develop`) or
  `CorewareHub/boss-control-tower` (base `develop/develop`).
- Grace's current state from her last report (or assign fresh).
- Engineer phase state from qa-phase-orchestrate (for slot conflicts; backend CI only).
- Direct message to Grace. Read-only gh for PR status on the chosen repo.

## TEST SLOT QUEUE

The slot covers test-running work on the shared Grok Bot cloud computer.
**Angelo standing rule 2026-09-11:** Margaret, Garman, and Grace all contend for
**one** slot (GRANTED / QUEUED / RELEASED). Garman still verifies on token 9
(`test_tenant_9` / `test_landlord_9`) but must queue here — he is **not** outside the
queue. Slot independence after the `scripts/test-lib.sh` ephemeral-sweep fix stays
suspended until Angelo explicitly lifts the standing rule. Gene records that the
standing rule is in force.

**The slot covers test-running work only** — Pest, migrate, or
`test:generate-schema-dump`, i.e. any delegation whose VERIFY is not `none`.
This matches the concurrency rule in repo-delegate-to-cursor; keep the two in step.

Never hold Margaret or Garman off non-Pest phase work (implement, prep, fold, `cursor review`) because the slot is busy or a merge is pending — only Pest/migrate/schema-dump wait on GRANTED.

| State | Meaning |
|-------|---------|
| GRANTED | Holder (Margaret, Grace, or Garman pre-fix) may run composer test:single |
| QUEUED | Request recorded; wait until current holder releases |
| RELEASED | Holder finished; Gene may grant to next queued request |

**Priority:** Margaret's phase verification takes precedence when she is actively
implementing and holds an open remediation loop. Garman queues with Margaret and Grace under the standing rule. Grace queues behind phase work unless Angelo explicitly prioritizes the feature PR.

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
   - Confirm owner/repo is allowed and PR is open, targets the matching base
     (`develop` or `develop/develop` per allow-list), and is a feature PR (not a CI phase PR, not a docs/*-FAILING-TESTS-FIX-PLAN.markdown plan PR).
   - If Grace already babysits another PR, STOP — one PR at a time. Finish or hand back the current one first.
   - Message Grace: owner/repo, base, PR number, URL, branch, any implementation plan path Angelo mentions. Tell her to run pr-babysit-loop.

2. **Slot requests.** When Grace, Margaret, or Garman (pre-fix) asks for the test slot:
   - If slot is free → GRANT to requester, record holder name.
   - If held → QUEUE the requester, tell them who holds it.
   - On RELEASED → grant to next QUEUED if any, else slot is free.

3. **Status relay.** When Grace posts a verdict comment:
   - **MERGE-READY:** Notify Angelo with PR URL, comment URL, reviewed-sha,
     final-sha. Remind him to merge manually and that the plan file is removed from
     the branch. MERGE-READY does **not** require full self-hosted Tests green
     (Angelo 2026-09-21). Grace is idle for the next assignment.
   - **BLOCKED:** Relay blocker to Angelo with comment URL. Grace waits or handback.
   - **WAITING:** Note in daily brief; WAITING = Bugbot-on-HEAD pending (including
     an unprocessed `cursor review` invoke — do not tell Grace to re-comment) or
     Pest GRANT pending only — **not** full self-hosted Tests pending. No Angelo
     ping unless WAITING exceeds one hour on cursor[bot] or Pest slot. Ambient
     full-suite CI red/pending is not a babysit blocker unless tip-caused.

4. **Separation from CI pipeline.** Never assign Grace a fix/ci-tests-phase-* branch or docs plan PR. Never assign an engineer a feature PR Grace owns. Katherine audits phase PRs; Grace babysits feature PRs — different tracks.

5. **Daily brief row.** Add to Gene's five-line brief:
   ```
   Grace: <owner/repo> PR #N (<branch> → <base>) — <state> — <verdict or blocker> — <next action>
   ```


## ELI5 DECISIONS FOR ANGELO (Angelo 2026-09-22)

Whenever you ask Angelo to decide something (widgets, questions, MERGE-READY needs-eyes, product calls, go/no-go):

- No jargon. Plain simple English. Explain like he is five (ELI5).
- Longer / wordier is OK if it makes the choice clearer.
- Before you list options, explain what the choice means in everyday words (what happens if he picks A vs B, in human terms — not just branch names or internal labels).

## HOW TO VALIDATE

- At most one PR assigned to Grace at a time.
- At most one test slot GRANTED at a time on token 1.
- Garman appears in the queue while the 2026-09-11 standing rule is in force.
- MERGE-READY relay includes comment URL and both SHAs when plan was retired.
- Grace never assigned to CI phase work.

## WHAT TO RETURN

Grace assignment (PR, branch, state), test slot holder (or free), Garman slot state
(in queue under standing rule; token 9 DB pair only), relay messages sent to Angelo, daily brief Grace line.

## WHAT REQUIRES APPROVAL

Assigning Grace and granting the test slot need no approval. Never merge. Escalate
to Angelo when Grace and Margaret both need the slot urgently and priority is unclear.
