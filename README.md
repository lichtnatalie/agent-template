# My OpenClaw Agent Setup

A personal [OpenClaw](https://github.com/openclaw/openclaw) gateway configuration for running always-on AI agents
that I use for real, ongoing projects — not a one-off script, a system I actively maintain.

## What this is

OpenClaw is an open-source agent gateway that connects LLM-backed agents to chat channels
(Telegram, in my case) so they can run continuously and hold state across conversations,
rather than living in a single chat session.

This repo is a **sanitized template** of my actual config: same structure, same setup pattern,
with every secret and personal identifier replaced by a placeholder. It's here to show how the
system is put together, not to be a drop-in working config.

## Agents

- **main** — a general-purpose agent (originally built and refined using Claude, currently
  running on GPT-5.4 via OpenAI Codex, with model fallback configured).
- **example-agent** ("Scout") — a planning and research agent bound to its own Telegram bot
  and its own isolated workspace. It owns one long-running project at a time: gathering
  material, tracking what's decided and what's still open, and surfacing next steps. Its
  workspace lives at `workspace/example-agent/`, matching the path in the config.

The split is by time horizon, not by topic: `main` handles the day, `example-agent` handles
a project that runs for months. Each has its own memory, so neither crowds out the other.
`workspace/example-agent/AGENTS.md` sets out when a second agent is worth having versus when
the same thing should just be a skill on the first one.

Each agent has its own workspace directory and agent directory, so context and files stay
scoped per-agent rather than bleeding into each other.

## What I actually do with this

The interesting part isn't the config, it's what running these agents day to day taught me:
getting an agent to respond well once is easy, getting it to hold up under repeated, real use
is not. That's meant constantly refining prompts, catching subtly wrong output, deciding when
to trust an agent to run unattended versus when to step in, and treating that process as an
informal evals loop rather than a one-time setup.

## Structure

```
openclaw.config.template.json   # gateway config: agents, models, channel bindings
workspace/                      # main agent's instructions, personality, memory
  AGENTS.md                     # the action contract — how the agent behaves, tool-call discipline
  SOUL.md                       # personality definition
  IDENTITY.md                   # this agent's name/persona ("Buddy")
  USER.md                       # who the agent is helping (example/placeholder here)
  MEMORY.md                     # long-term curated memory (empty template)
  CHAT_RULES.md                 # group chat etiquette
  HEARTBEAT.md / HEARTBEAT_RULES.md  # periodic proactive check-ins
  TOOLS.md                      # local environment notes (cameras, SSH, TTS, etc.)
  example-agent/                # a second, independent agent sharing the same gateway
    AGENTS.md, IDENTITY.md      # its own instructions and persona ("Scout", planning/research)
```

Each agent has its own workspace, so two agents can run from one gateway with completely
separate context, memory, and personality.

## Requirements

- **OpenClaw** — [openclaw/openclaw](https://github.com/openclaw/openclaw). Install with the
  official script:
  ```sh
  curl -fsSL https://openclaw.ai/install.sh | bash
  ```
  or, if you manage Node yourself (Node 22.22.3+, 24.15+ or 25.9+):
  ```sh
  npm install -g openclaw@latest --allow-scripts=openclaw
  ```
- A **Telegram bot token** per agent, from [@BotFather](https://t.me/BotFather).
- An API key or OAuth login for whichever model provider you point the agents at.

> **Version note:** this config was written against OpenClaw **2026.4.5** (see `meta.lastTouchedVersion`
> in the config). The config schema does change between releases — if a key is rejected, check it
> against the version you installed rather than assuming this template is wrong.

## Setup (if you want to run your own)

1. Install OpenClaw and run onboarding once, so the standard directories exist:
   ```sh
   openclaw onboard --install-daemon
   ```

2. Create the directories this config expects. OpenClaw does **not** create the per-agent
   directories for you, and it will fail to start if `agentDir` doesn't exist:
   ```sh
   mkdir -p ~/.openclaw/workspace/example-agent
   mkdir -p ~/.openclaw/agents/main/agent
   mkdir -p ~/.openclaw/agents/example-agent/agent
   ```

3. Copy `openclaw.config.template.json` to `~/.openclaw/openclaw.json` and replace every
   placeholder (`<...>`):
   - `<TELEGRAM_BOT_TOKEN_DEFAULT>` / `<TELEGRAM_BOT_TOKEN_EXAMPLE>` — one bot token per agent
   - `<GATEWAY_AUTH_TOKEN>` — generate a long random string, e.g. `openssl rand -hex 32`.
     Use token auth; do not set a plaintext password.
   - `<you>` — your macOS username, in all five paths
   - `<your-email>` — the account for your model provider's OAuth profile, or delete that
     profile block if you don't use it

4. Copy the contents of `workspace/` into `~/.openclaw/workspace/`, then fill in `USER.md`
   with your own context. The agents read it every session, so vague input here means vague
   output for weeks.

5. Create the memory directories the agent instructions reference:
   ```sh
   mkdir -p ~/.openclaw/workspace/memory
   mkdir -p ~/.openclaw/workspace/example-agent/memory
   ```
   `AGENTS.md` tells each agent to read `memory/YYYY-MM-DD.md` at the start of every session
   and `HEARTBEAT_RULES.md` tracks state in `memory/heartbeat-state.json`. The agent creates
   the files itself, but the directory needs to exist. These are gitignored here — they fill
   up with personal context fast.

6. Start it and confirm it's alive:
   ```sh
   openclaw gateway status
   ```

7. Never commit your real `openclaw.json` or a filled-in `USER.md`. See `.gitignore`.

## Security notes

- The real config file contains live API tokens and a gateway auth token. It is **never**
  committed here.
- Use token-based gateway auth only. Avoid setting a plaintext password in config.
- `credentials/`, `logs/`, and `telegram/` runtime state are all gitignored — they contain
  session data and pairing secrets that shouldn't leave the machine.
