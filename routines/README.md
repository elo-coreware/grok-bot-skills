# routines/

Grok Bot **liveness** routines for the Coreware team. These are short status pings so Angelo can see who is responsive — not work dumps, not task assignments, not test runs.

## Schedule (Asia/Manila / Philippine Standard Time)

| Routine | When |
|---------|------|
| [weekday-liveness-20min](weekday-liveness-20min.md) | Weekdays only, every 20 minutes, 8:00 AM–8:40 PM PST |
| [weekday-overnight-liveness-hourly](weekday-overnight-liveness-hourly.md) | Weekdays only, hourly at :47, 9:00 PM–7:47 AM PST (hours 21–23 and 0–7) |

**Weekends are off** (cron `1-5` only). Do not enable Saturday/Sunday fires.

## Ownership

- **Gene Kranz** owns the orchestrator copy of these routines (day + overnight).
- **Wernher von Braun** may keep a **separate** Grok Bot copy when he is covering as backup. Do not fold Wernher's overnight job into Gene's daytime prompt — keep the two bots' live configs distinct.
- Other bots may copy the same intent into their own Grok Bot routines if Angelo asks; this directory is the source of truth for wording and cron.

## Shape

Each routine has:

- `name` — display name in Grok Bot
- `schedule` — `CRON_TZ=Asia/Manila …` expression
- `prompt` / intent — what the bot posts on each fire

Markdown is human-readable; matching `.json` is machine-readable. Keep both in sync when editing.
