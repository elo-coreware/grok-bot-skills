---
name: qa-fix-audit
description: Use when Gene assigns Katherine a phase PR (or other PR) to audit
---
---
name: qa-fix-audit
description: Use when Gene assigns Katherine a phase PR (or other PR) to audit
---
# qa-fix-audit

## WHEN TO USE

Gene assigns a phase PR (or another PR) for Katherine's audit. This is not plan-building. Plan-building is qa-fix-plan-build (Aaron). Do not start this audit until `cursor[bot]` has reviewed **this PR's current head SHA**, unless Gene explicitly says the PR is docs-only and Bugbot does not apply.

## REQUIRED INPUTS AND ACCESS

- PR number, branch (`fix/ci-tests-phase-N-<slug>`), phase table from the current plan.
- Read access at the PR head. Never run test commands.
- Authority files (lowercase commands only, not `.cursor/BUGBOT.md`):
  - `.cursor/rules/test-failure-triage.mdc` (alwaysApply; wins on test-vs-app classification)
  - `.cursor/commands/automated-tests-validity-detection.md`
  - `.cursor/commands/bugbot.md`
  - `.cursor/commands/bugbot-triage.md`
- GitHub PR reviews and review threads from `cursor[bot]` (Cursor Bugbot **app**). Fetch them. A review whose `commit_id` is not the current head SHA is stale. They are inputs to triage, not a substitute for lowercase `/bugbot`.

## SEQUENCE OF WORK

1. Confirm head SHA. If `cursor[bot]` has no review whose `commit_id` equals that SHA, STOP. Do not PASS. Do not `gh pr ready`. Return WAITING to Gene: "Bugbot app has not reviewed this SHA; the owning engineer must comment `cursor review` on the handoff commit." Do not invent a PASS with a note that Bugbot was missing.

2. Read `git diff origin/develop...HEAD` in full, every line. Three-dot vs develop. Out-of-scope: files not in that diff, lines not in a hunk, code identical on develop.

3. Count assertions and test blocks before and after. A net drop or any removed test is a FAIL unless the plan authorized it and the justification holds.

4. qa-validity-scan on changed `tests/**/*Test.php`. Authority: `.cursor/commands/automated-tests-validity-detection.md`.

5. Grep the diff for cheating: deleted/skipped tests, weakened assertions, assertOk downgrades, tautologies, commented assertions, driver detection, Cache::flush(), runtime Schema:: DDL, `->first()` fixture selection, loosened tolerances, polarity inversions.

6. Per file, apply `.cursor/rules/test-failure-triage.mdc`. Scaffolding-only test edits (setup, fakes, fixtures) can PASS. **FAIL** polarity inversions, product-behavior assertion rewrites, or `it(...)` renames that match a bug (PR 5785 class), even if labeled tests-only. Contract failures must stay red with an **ESCALATED** plan-doc row — a red test with no ESCALATED row is FAIL. New `app/` in this phase's own commits without Angelo's written go-ahead is FAIL. Framework-semantics corrections (PR 5778 Inertia 409→302) are allowed. Would a genuine product regression still fail this test afterward? If not, FAIL. Yesterday's "tests follow current app behavior" instruction is revoked.

7. Run `.cursor/commands/bugbot.md` (lowercase) on `develop...HEAD`. Plan-mode detection only. Never implement fixes. Never use `.cursor/BUGBOT.md`. Never comment `cursor review` or `bugbot run` yourself — the owning engineer invokes the app.

8. Fetch GitHub review threads on this PR whose author is `cursor[bot]`. Include unresolved, outdated, and newly landed threads. Stale threads (reviewed a previous SHA) still go through triage if the finding may still apply to HEAD.

9. Immediately run `.cursor/commands/bugbot-triage.md` on **the combined finding list** — her own lowercase `/bugbot` findings plus the GitHub `cursor[bot]` threads. Classify each as valid bug, false positive, out of scope, or needs clarification. Only **valid + in-scope** bugs feed the verdict. Do not invent extra findings during triage. Do not implement. Do not mark a finding false-positive just to unblock ready.

10. Verdict is PASS, PASS WITH NOTES, or FAIL. file:line (+ hunk for bugbot items) for every finding. Explicit verdict line for every changed file so nothing is silently skipped. Valid in-scope bugbot Critical/High/Medium that is still unfixed in the HEAD diff → FAIL. Untriaged bugbot output (lowercase or GitHub app) is not evidence.

11. qa-pr-verdict-comment. Convert draft to ready **only** when the verdict is PASS or PASS WITH NOTES **and** `cursor[bot]` reviewed this SHA **and** no valid in-scope Bugbot item remains unfixed. Leave draft on FAIL, WAITING, or a stale Bugbot review. Never merge. Never GitHub review-approve. Never resolve `cursor[bot]` threads yourself.

## HOW TO VALIDATE

- Diff was `develop...HEAD`, not two-dot.
- Assertion and test-block counts are reported.
- `cursor[bot]` review `commit_id` equals HEAD before any PASS.
- Validity scan, cheating grep, triage.mdc classification, lowercase bugbot, GitHub `cursor[bot]` threads, and bugbot-triage all ran.
- Bugbot-triage table exists and covers both sources. False positives and out-of-scope items are not FAIL reasons.
- Polarity inversions and missing ESCALATED rows were treated as FAIL.
- `gh pr ready` ran only after both Katherine's verdict and Bugbot-on-this-SHA were clean.
- Verdict was never upgraded to unblock a schedule.

## WHAT TO RETURN

Verdict (including WAITING), comment URL, ready-vs-draft, head SHA, Bugbot review SHA, assertion counts, validity counts, cheating result, triage.mdc classifications, bugbot-triage summary (valid / false positive / out of scope / needs clarification) split by source (lowercase `/bugbot` vs GitHub `cursor[bot]`), per-file verdict lines.

## WHAT REQUIRES APPROVAL

Posting and marking ready need no approval. Never merge. Never implement bugbot or triage fixes. Escalate to Angelo when the same file fails this audit twice, assertion count dropped without plan authorization, an engineer disagrees with the verdict, or the phase inverted contract assertions or introduced application-behavior changes without a written go-ahead.
