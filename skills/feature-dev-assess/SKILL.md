---
name: feature-dev-assess
description: >-
  Use when a feature engineer must assess a feature on DEV (Control Tower and/or
  tenant) after merge, before handoff, or when Angelo asks for another
  verification round — mobile, layout, blank fields, and UI interference checks
  with a PASS/FAIL report.
---
# Feature DEV assess

## When to use

Run this after a feature ships to DEV, when Angelo asks for another verification round, or before calling a feature “done on DEV.” Use it for visual/behavioral assessment on real DEV hosts. Do not use it for CI Pest, plan writing, or production checks.

## Environments (hard rules)

- Control Tower DEV: `https://dev.coreware.app` on branch tip `develop/<feature-slug>`. Never tip-push experiments onto `develop/develop`.
- Backend / tenant DEV: primary tenant `https://development-corestore-alpha.coreware.app`. Backend feature tips use `dev-test/<feature>`, not `develop/<feature>`.
- Never use `https://coreware.coreware.app` to preview or experiment (that host is PRODUCTION).
- PROD web hosts — observe / peek only (Angelo 2026-09-18; clarified 2026-09-20): `https://controltower.coreware.app` and `https://coreware.coreware.app`. When Angelo asks for a PROD recheck: peek/observe ONLY. NO modifying (no edits, creates, deletes, status changes, write form submits, deploys, tip-pushes, or write APIs). Default is never modify.
- If you must change data to verify a feature, use DEV only (`https://dev.coreware.app` and/or `https://development-corestore-alpha.coreware.app`). Never mutate PROD to test.
- If login, Google Allow, or 2FA blocks you, stop and report — that is Angelo-only.

## Inputs (from Gene / Angelo)

Require before starting:

1. Which surfaces (Control Tower, tenant, or both)
2. Entry URLs or banner/feature IDs to open
3. Focus areas (or use the default matrix below)
4. Whether a PR is allowed yet (default: **no PR** — report only)

## Default assessment matrix

For each surface in scope, walk these and mark PASS / FAIL / N/A:

| ID | Check | What “PASS” means |
|----|--------|-------------------|
| A | Mobile responsiveness | Usable at ~375px and ~768px width; no clipped CTA, title, or image; no horizontal scroll unless intentional |
| B | Spacing / padding | No obvious excess padding (especially below the feature), collapsed gaps, or uneven margins vs sibling UI |
| C | Blank / optional fields | Fields that should allow empty can clear and save; required fields say so clearly; no silent refuse |
| D | Scroll / float / overlay | Sticky or sandbox chrome does not cover controls, steal clicks, or float badly while scrolling |
| E | Desktop baseline | Same primary happy path looks correct at ~1280px+ |
| F | Cross-surface parity | If both CT and tenant are in scope, same content/settings do not diverge in surprising ways (note expected product differences) |

Angelo may add one-off checks (named URLs, specific fields). Append those as G, H, …

## How to run

1. Confirm hosts. Prefer DEV. If Angelo assigned a PROD peek: observe only — never write on PROD.
2. Open each entry URL. Capture desktop and mobile (or narrowed viewport) screenshots of the primary state.
3. Walk the matrix. For every FAIL: one-line repro, which surface, screenshot, and whether it looks like a one-line fix vs a new waterfall item.
4. If CloudAgent / transient failures hit, retry with backoff yourself. Only wait on Angelo for auth, merges, product calls, or production access.
5. Do **not** open or push a PR unless Angelo (or Gene after Angelo) explicitly says to start the waterfall for named failures.

## Report format (to Gene)

Short prose or a compact table is fine. Include:

- Surfaces tested + entry URLs
- Matrix results (A–F + extras)
- FAIL details with screenshots
- Suggested next step: none / one-line fix on existing lane / new feature-waterfall plan (same-PR rule)
- Blockers that need Angelo (auth, product decision)

Gene relays to Angelo. Feature engineers do not merge.
