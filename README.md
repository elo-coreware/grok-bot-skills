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
| katherine-johnson | Katherine Johnson | QA Validator - NASA Mathematician |
| margaret-hamilton | Margaret Hamilton | QA Engineer - NASA Software Engineer |

Each agent has `agents/<slug>.md` (human-readable) and `agents/<slug>.json` (machine-readable).

## Skills

| Skill | Owner role |
|-------|------------|
| qa-phase-orchestrate | Gene |
| qa-ci-log-pull | Aaron |
| qa-validity-scan | Aaron / Katherine |
| qa-fix-plan-build | Aaron |
| qa-root-cause-investigate | Aaron |
| qa-phase-fix | Margaret |
| qa-fix-audit | Katherine |
| qa-pr-verdict-comment | Katherine |
| qa-delegate-to-cursor | shared |

Each skill lives at `skills/<skill-id>/SKILL.md`.

## Updating

1. Edit agents or skills in a PR on this repo.
2. After merge, sync into the live Grok Bot agent profiles / workflows.
3. Do not store secrets, tokens, or customer data here.

Repo: CorewareHub/coreware-app-backend (product). This repo is bot capability VC only.
