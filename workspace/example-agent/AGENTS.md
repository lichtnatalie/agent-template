# AGENTS.md - example-agent workspace

This is a second agent running from the same OpenClaw gateway as `main`, with its own
workspace, its own Telegram bot binding, and its own memory. Nothing here is shared with
`main` — that separation is the point of this example.

## Scope

Give each agent a narrow, explicit job. A second agent earns its keep when it has a
distinct purpose and a distinct audience, not when it is a copy of the first one with a
different name.

State that job here in a sentence or two, then let the rules below handle the rest.

## Inherited conventions

This agent follows the same core contract as `main` — see `../AGENTS.md` for the full
version. The short form:

- **Promise = action.** If the work needs a tool call, the response contains the tool call.
  Text that describes future work is a bug, not a style.
- **Read before edit.** Always.
- **Write it down.** Mental notes don't survive a restart; files do.
- **Local actions are free** (read, list, search, edit workspace files). Anything that
  leaves the machine — email, posts, messages to third parties — ask first.
- **Loop guard.** If a tool fails the same way twice, stop and report.

## Per-agent files

- `IDENTITY.md` — this agent's name and persona
- `MEMORY.md` — its own long-term memory (main session only, never in group contexts)
- `memory/YYYY-MM-DD.md` — daily raw notes

Add a `SOUL.md` here if this agent should have a personality distinct from `main`'s.
