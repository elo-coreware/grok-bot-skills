---
name: pr-babysit-loop
description: >-
  Use when Gene or Angelo assigns Grace one feature PR to babysit until MERGE-READY
---
# pr-babysit-loop

## WHEN TO USE

Gene or Angelo assigns Grace exactly one feature PR (not a CI phase PR, not a docs/*-FAILING-TESTS-FIX-PLAN.markdown plan PR). Grace runs this loop until the PR is MERGE-READY or she hands it back with BLOCKED and a written reason.

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

- PR number and URL, branch name, head SHA at assignment.
- Read/write on the PR branch via repo-delegate-to-cursor.
- owner/repo + base for the assigned PR (allowed:
  `CorewareHub/coreware-app-backend` → `develop`, or
  `CorewareHub/boss-control-tower` → `develop/develop`).
- gh authenticated for that owner/repo.
- Committed implementation plan on the branch when one exists (identification rules
  from .cursor/commands/git-commit.md step 5; BugBot-generated plans excluded).

## STATES

| State | Meaning |
|-------|---------|
| SWEEPING | Running pr-bugbot-sweep on current HEAD |
| REMEDIATING | Implementing valid in-scope findings (pr-finding-remediate) |
| WAITING-BUGBOT | Pushed a fix commit or invoke pending; waiting for cursor[bot] review on HEAD — do not re-invoke while PENDING |
| WAITING-PEST | Findings clean; waiting for Gene Pest GRANT to run touched tests |
| BLOCKED | Cannot proceed without Angelo (conflicts, scope, slot denied, escalation) |
| READY | Gates 1–4 pass per standing-rule verify (Bugbot CLEAN==HEAD + Pint/lint + touched tests under Pest GRANT); plan retired; MERGE-READY posted — full self-hosted Tests green is NOT required |

Exactly one PR at a time. Never start a second PR until the first is MERGE-READY or explicitly handed back to Gene/Angelo.


## LOCAL PINT (Angelo 2026-09-23) — full-repo, match CI

CI `.github/workflows/lint.yml` ("Check Code Style") runs bare `pint --test` with **no** path filter — it scans the whole Pint-configured tree. Babysit / phase verify must match that gate.

**Fix (write):** run `./vendor/bin/pint` (or `pint`) with **no** `--dirty` and **no** path args so it reformats the entire tree. Commit `:art: pint` (or fold into the remedi commit) if files change. Never `composer format`. Do **not** use `--dirty` or path-scoped Pint as the only fix when Check Code Style is red or when preparing MERGE-READY.

**Verify:** `./vendor/bin/pint --test` must exit 0 on HEAD (full tree). Equivalently, confirm GitHub Actions job `lint (8.3)` / Check Code Style is **green** on this HEAD (job URL is fine evidence). Style issues anywhere under Pint paths on the tip **are** babysit blockers — fix them on this branch until full `pint --test` is green. Do **not** dismiss full-repo lint red as "ambient / not tip-caused."

Docs-only or non-PHP commits: skip only when `pint --test` (or the Actions lint job) is already green on HEAD.


## VERIFY / MERGE-READY GATE (Angelo standing rule 2026-09-21, via Gene)

Bot PR verify and MERGE-READY must **not** wait on the full self-hosted CI
`Tests` / Run Tests suite (~60 min). Do **not** poll `gh pr checks` for that
suite as a hard babysit gate.

**MERGE-READY gate (all required):**
1. Bugbot CLEAN == HEAD (`cursor[bot]` review `commit_id` equals HEAD; zero
   unfixed valid in-scope findings)
2. Pint / lint **full-repo** clean: local `./vendor/bin/pint` (write) then
   `./vendor/bin/pint --test` exit 0, **or** GitHub Actions Check Code Style /
   `lint (8.3)` green on HEAD (no `--dirty` / path-scoped-only)
3. Touched tests only, run on the shared machine under Gene's Pest GRANT
   (`composer test:single` / equivalent on changed paths)

**Ambient full-suite CI red is OK** and is **not** a babysit blocker unless the
failure is tip-caused (introduced by this PR's HEAD). Do not hold MERGE-READY,
poll for ~1h, or post WAITING solely because ambient self-hosted Tests are red
or still pending.

## CURSOR REVIEW INVOKE GATE (Angelo 2026-09-21; example CorewareHub/coreware-app-backend#6487)

Bots must not spam bare `cursor review` / `bugbot run` while Bugbot is still pending
(e.g. #6487: invoke at 23:38 UTC, another at 00:27 UTC, then cursor[bot] CLEAN at
00:29 on SHA 435e6130). **If a cursor Bugbot review is still pending** (existing
Bugbot run in flight, or a prior `cursor review` / `bugbot run` comment not yet
processed for current HEAD), do **NOT** post another `cursor review` yet.

**Before posting `cursor review` or `bugbot run`:**
1. Read current HEAD SHA.
2. List PR issue comments whose body trims to exactly `cursor review` or `bugbot run` (case-insensitive).
3. List reviews by `cursor[bot]`.
4. If any non-stale cursor[bot] review has `commit_id` == HEAD → already reviewed; do **not** post.
5. If the newest invoke comment has no later cursor[bot] review that covers HEAD (`commit_id` == HEAD with `submitted_at` after that comment, or any review on HEAD after the comment) → treat as **PENDING**; stay WAITING-BUGBOT; do **not** post another invoke.
6. Only post exactly one new invoke when: HEAD is not yet reviewed by cursor[bot] **AND** there is no pending unprocessed invoke for this PR.
7. Never post a second `cursor review` for the same HEAD SHA.
8. After a new commit, you may invoke once only after confirming no pending invoke remains from before that push (or the prior invoke was for an older SHA and Bugbot already reported / timed out — prefer wait for pending to clear or confirm stale before re-invoke).

## SEQUENCE OF WORK

1. **Intake.** Record owner/repo, base branch, PR number, branch, initial HEAD SHA.
   Fetch origin. Confirm PR is open and targets the correct base (`develop` or
   `develop/develop` per allow-list). If merge conflicts with `<base>` exist before
   work starts, post BLOCKED verdict and ask Angelo to resolve — do not merge `<base>`
   yourself unless Gene or Angelo explicitly assigns conflict resolution.

2. **SWEEPING.** Run pr-bugbot-sweep. Produce the combined findings ledger (local
   /bugbot + /bugbot-triage + cursor[bot] HEAD threads).

3. **Decision.** If the ledger has unfixed valid in-scope items → REMEDIATING.
   If zero unfixed valid in-scope items → pr-merge-readiness-audit.

4. **REMEDIATING.** For each valid in-scope item (Critical first, then High, then
   Medium), run pr-finding-remediate. If verification needs tests, message Gene for
   the test slot. Do not run composer test:single until Gene grants GRANTED. If
   QUEUED, wait. Batch related fixes into one commit when sensible. **Pint gate:**
   if PHP changed **or** Check Code Style is red on HEAD, run LOCAL PINT (full-repo write + `--test`) and
   commit style fixes before `cursor review`. After push, run the
   **CURSOR REVIEW INVOKE GATE** above; only then comment exactly one
   `cursor review` as Angelo → WAITING-BUGBOT. Never post a second invoke
   for the same HEAD or while a prior invoke is still PENDING.

5. **WAITING-BUGBOT.** Stay here while Bugbot is pending: either (a) a bare
   `cursor review` / `bugbot run` invoke is still unprocessed for current HEAD,
   or (b) cursor[bot] has not yet posted a review with `commit_id` == HEAD.
   **Do not post another invoke** while PENDING (CURSOR REVIEW INVOKE GATE).
   Poll until cursor[bot] has a review whose commit_id equals HEAD, or timeout
   and report WAITING verdict to Gene. When review lands, return to SWEEPING
   (re-triage GitHub threads + optional fresh local sweep if findings changed
   materially).

6. **Merge readiness.** When sweep shows zero unfixed valid in-scope items, run
   pr-merge-readiness-audit (gates 1–4) under the Angelo 2026-09-21 standing rule.
   Route on the outcome:
   - Gate 1 or 2 fail → back to SWEEPING or REMEDIATING.
   - Gate 3 waiting on Pest GRANT (touched-test verify not yet run) → WAITING-PEST.
   - Gate 3 fail because **tip-caused** touched-test / Pint failure → post BLOCKED.
   - Gate 4 `mergeable == "UNKNOWN"` → brief re-poll mergeability only (not full CI).
   - Gate 4 conflicts → post BLOCKED.
   - Ambient full-suite self-hosted Tests red or pending → **ignore** for routing;
     not a blocker unless tip-caused.
   - Gates 1–4 pass → pr-plan-doc-retire → pr-merge-verdict-comment MERGE-READY →
     notify Gene and Angelo. Do **not** wait for full Tests green first.

7. **WAITING-PEST (not WAITING-CI).** Do **not** poll the full self-hosted Tests
   suite (~60 min) and do **not** treat ambient CI red as a hard gate. WAITING
   means only: (a) Bugbot review not yet on HEAD, or (b) Gene has not yet GRANTED
   the Pest slot for touched-test verify. When Pest is GRANTED, run touched tests
   only on the shared machine, RELEASE the slot, re-run pr-merge-readiness-audit,
   and proceed. Plan-retirement (docs delete) does **not** require re-entering a
   full-suite CI wait before MERGE-READY.

8. **Handback.** On BLOCKED, post verdict, tell Gene the blocker, release the test
   slot if held, and stop. Angelo or Gene must re-assign to resume.

## HOW TO VALIDATE

- Only one PR active in Grace's state at any time.
- Every fix commit followed by at most one `cursor review` on that SHA, and
  only after CURSOR REVIEW INVOKE GATE passes (no pending invoke; HEAD not
  already reviewed).
- Test runs only after Gene grants the slot; verify = touched tests only (never full suite for MERGE-READY).
- Ambient full-suite CI red/pending is not a babysit blocker unless tip-caused.
- MERGE-READY never posted before pr-plan-doc-retire completes (when a plan existed).
- Never merged the PR.

## WHAT TO RETURN

State, PR number, branch, current HEAD SHA, findings ledger summary (valid remaining /
fixed / dismissed), gate table, verdict comment URL, blocker if BLOCKED.

## WHAT REQUIRES APPROVAL

Pushing fix commits and posting comments need no approval when posting as Angelo.
Never merge. Escalate to Angelo for scope beyond the implementation plan, unresolvable
conflicts, or repeated remediation failure on the same finding.
