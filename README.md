# grok-bot-skills

Version control for Coreware QA Grok Bot **agents** and **skills**.

Public source of truth when updating bot capabilities. Live bot configs may lag this repo until synced.

## Layout

```
agents/   # persona name, label (title), description
skills/   # reusable skill definitions (SKILL.md per skill)
```

## Agents

| Slug | Name | Label |
|------|------|-------|
| gene-kranz | Gene Kranz | QA Chief of Staff - NASA Flight Director |
| john-aaron | John Aaron | QA Analyst - NASA Flight Controller |
| bill-tindall | Bill Tindall | QA Coverage Architect - NASA Mission Techniques |
| katherine-johnson | Katherine Johnson | QA Validator - NASA Mathematician |
| margaret-hamilton | Margaret Hamilton | QA Engineer - NASA Software Engineer |
| grace-hopper | Grace Hopper | PR Readiness Engineer - US Navy Computing Pioneer |

Each agent has `agents/<slug>.md` (human-readable) and `agents/<slug>.json` (machine-readable).

**Tracks:** The NASA five (Gene through Katherine) own CI test health. Grace Hopper is the non-NASA, non-CI track — she babysits feature PRs until MERGE-READY.

## Skills

| Skill | Owner role |
|-------|------------|
| qa-phase-orchestrate | Gene |
| pr-babysit-orchestrate | Gene |
| qa-ci-log-pull | Aaron |
| qa-validity-scan | Aaron / Katherine |
| qa-fix-plan-build | Aaron |
| qa-module-test-inventory | Tindall |
| qa-coverage-plan-build | Tindall |
| qa-suite-hygiene-plan-build | Tindall |
| qa-root-cause-investigate | Aaron / Tindall |
| qa-phase-fix | Margaret |
| qa-fix-audit | Katherine |
| qa-pr-verdict-comment | Katherine |
| pr-babysit-loop | Grace |
| pr-bugbot-sweep | Grace |
| pr-finding-remediate | Grace |
| pr-merge-readiness-audit | Grace |
| pr-plan-doc-retire | Grace |
| pr-merge-verdict-comment | Grace |
| qa-delegate-to-cursor | shared |

Each skill lives at `skills/<skill-id>/SKILL.md`.

## Updating

1. Edit agents or skills in a PR on this repo.
2. After merge, sync into the live Grok Bot agent profiles / workflows.
3. Do not store secrets, tokens, or customer data here.

Repo: CorewareHub/coreware-app-backend (product). This repo is bot capability VC only.
