# routines/

Grok Bot **liveness** routines for the Coreware team. These are short status pings so Angelo can see who is responsive — not work dumps, not task assignments, not test runs.

The two files here are the **canonical** wording and cron for both routines. Sync them into the live Grok Bot that currently owns orchestration. Do **not** run them on Gene and Wernher at the same time.

## Schedule (Asia/Manila / Philippine Standard Time)

| Routine | When |
|---------|------|
| [weekday-liveness-20min](weekday-liveness-20min.md) | Weekdays only, every 20 minutes, 8:00 AM–8:40 PM PST |
| [weekday-overnight-liveness-hourly](weekday-overnight-liveness-hourly.md) | Weekdays only, hourly at :47, 9:00 PM–7:47 AM PST (hours 21–23 and 0–7) |

**Weekends are off** (cron `1-5` only). Do not enable Saturday/Sunday fires.

## Ownership — one orchestrator only (no double ping)

Angelo must not get two liveness pings for the same slot.

- The orchestrator Angelo **last talked to** owns both liveness routines (day + overnight).
- The **other** orchestrator stays **quiet** — those two routines must be **paused** on that bot.
- **Right now:** Gene is last-talked-to → Gene's copies are the live ones; **Wernher's copies must be paused**.
- If Angelo next assigns orchestration to Wernher → Wernher takes liveness and **Gene pauses**.

Gene and Wernher must never both execute these routines while enabled. Canonical files in this repo are shared templates; only one live bot fires them.

Other bots may copy the intent into their own Grok Bot routines only if Angelo asks; this directory remains the source of truth for wording and cron.

## Shape

Each routine has:

- `name` — display name in Grok Bot
- `schedule` — `CRON_TZ=Asia/Manila …` expression
- `prompt` / intent — what the bot posts on each fire

Markdown is human-readable; matching `.json` is machine-readable. Keep both in sync when editing.
