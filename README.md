# grok-bot-skills

Version control for Coreware QA Grok Bot **agents**, **skills**, and **routines**.

Public source of truth when updating bot capabilities. Live bot configs may lag this repo until synced.

## Layout

```
agents/    # persona name, label (title), description
skills/    # reusable skill definitions (SKILL.md per skill)
routines/  # Grok Bot liveness schedules (weekday day + overnight; weekends off)
```

## Agents

| Slug | Name | Label |
|------|------|-------|
| gene-kranz | Gene Kranz | QA Chief of Staff - NASA Flight Director |
| wernher-von-braun | Wernher von Braun | Engineering Chief of Staff |
| john-aaron | John Aaron | QA Analyst - NASA Flight Controller |
| bill-tindall | Bill Tindall | QA Coverage Architect - NASA Mission Techniques |
| katherine-johnson | Katherine Johnson | QA Validator - NASA Mathematician |
| margaret-hamilton | Margaret Hamilton | QA Engineer - NASA Software Engineer |
| jack-garman | Jack Garman | QA Engineer II - NASA Software Engineer |
| grace-hopper | Grace Hopper | PR Readiness Engineer - US Navy Computing Pioneer |
| raye-montague | Raye Montague | PR Readiness Engineer |
| susan-kare | Susan Kare | Feature Engineer - Interface |
| jean-bartik | Jean Bartik | Feature Engineer - Workflow |

Each agent has `agents/<slug>.md` (human-readable) and `agents/<slug>.json` (machine-readable).

**Tracks:** The NASA six (Gene, Aaron, Tindall, Katherine, Margaret, Garman) own CI test health. Garman is the standby engineer — idle unless Angelo explicitly activates DUAL mode via Gene. Wernher (Engineering Chief of Staff) is Gene's backup orchestrator when Gene is offline — alive, not QA-labeled. Grace and Raye are the non-NASA PR readiness track — they babysit feature implementation PRs until MERGE-READY. Susan Kare and Jean Bartik are feature engineers (not QA): plan → Aaron validate → implement on the Angelo 2026-09-17 feature waterfall.

**DEV (boss-control-tower):** main is `develop/develop`. DEV testing uses `develop/<feature-slug>` on https://dev.coreware.app. Never tip-push experiments onto `develop/develop`. coreware-app-backend base remains `develop`.

## Skills

| Skill | Owner role |
|-------|------------|
| qa-phase-orchestrate | Gene |
| pr-babysit-orchestrate | Gene / Wernher |
| feature-waterfall-orchestrate | Gene / Wernher |
| qa-ci-log-pull | Aaron |
| qa-validity-scan | Aaron / Katherine |
| qa-fix-plan-build | Aaron |
| feature-plan-validate | Aaron |
| qa-module-test-inventory | Tindall |
| qa-coverage-plan-build | Tindall |
| qa-suite-hygiene-plan-build | Tindall |
| qa-root-cause-investigate | Aaron / Tindall |
| qa-phase-fix | Margaret / Garman |
| qa-fix-audit | Katherine |
| qa-pr-verdict-comment | Katherine |
| feature-plan-build | Susan Kare / Jean Bartik |
| feature-implement | Susan Kare / Jean Bartik |
| pr-babysit-loop | Grace / Raye |
| pr-bugbot-sweep | Grace / Raye |
| pr-finding-remediate | Grace / Raye |
| pr-merge-readiness-audit | Grace / Raye |
| pr-plan-doc-retire | Grace / Raye |
| pr-merge-verdict-comment | Grace / Raye |
| repo-delegate-to-cursor | shared |

Each skill lives at `skills/<skill-id>/SKILL.md`.

## Routines

Team liveness pings (not work dumps). See [`routines/`](routines/README.md).

| File | Schedule (Asia/Manila) |
|------|------------------------|
| [weekday-liveness-20min](routines/weekday-liveness-20min.md) | Weekdays `*/20` 8:00 AM–8:40 PM PST |
| [weekday-overnight-liveness-hourly](routines/weekday-overnight-liveness-hourly.md) | Weekdays hourly at `:47`, 9:00 PM–7:47 AM PST |

Weekends off. Canonical copies live here. Only the orchestrator Angelo last talked to runs them (Gene live now; Wernher paused) — never both, or Angelo gets a double ping.

## Updating

1. Edit agents, skills, or routines in a PR on this repo.
2. After merge, sync into the live Grok Bot agent profiles / workflows / routines.
3. Do not store secrets, tokens, or customer data here.

Repo: CorewareHub/coreware-app-backend (product). This repo is bot capability VC only.
