# HEARTBEAT_RULES.md

When you receive a heartbeat poll, don't reflexively reply `HEARTBEAT_OK`. Use it.

Default prompt:
`Read HEARTBEAT.md if it exists. Follow it strictly. Do not infer or repeat old tasks. If nothing needs attention, reply HEARTBEAT_OK.`

You can edit `HEARTBEAT.md` with a short checklist. Keep it small — tokens cost.

## Heartbeat vs Cron

**Heartbeat** — batchable checks (inbox + calendar + notifications), drift-tolerant timing, conversational context.

**Cron** — exact timing, isolated context, one-shot reminders, direct channel delivery.

Batch periodic checks into `HEARTBEAT.md` instead of multiplying cron jobs.

## What to check (rotate, 2-4×/day)

- Emails — anything urgent?
- Calendar — next 24-48h?
- Mentions — Twitter/social?
- Weather — relevant if your human is going out?

Track in `memory/heartbeat-state.json`:
```json
{ "lastChecks": { "email": 1703275200, "calendar": 1703260800 } }
```

## Reach out when

- Important email arrived
- Calendar event <2h away
- You found something interesting
- It's been >8h since you said anything

## Stay quiet (HEARTBEAT_OK) when

- 23:00–08:00 unless urgent
- Your human is clearly busy
- Nothing new since last check
- You checked <30 min ago

## Free background work

- Organize memory files
- `git status` on projects
- Update docs
- Commit/push your own changes
- Review and update `MEMORY.md`

### Memory maintenance

Every few days, use a heartbeat to read recent `memory/YYYY-MM-DD.md` files, distill significant stuff into `MEMORY.md`, drop outdated entries.

Daily files = raw notes. MEMORY.md = curated wisdom.
