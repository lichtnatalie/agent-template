# My Agent Setup

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

## What actually broke

The rules in `workspace/AGENTS.md` aren't hypothetical — each one exists because something
went wrong first. The useful ones:

### 1. The agent promised instead of acting

The most common and most costly failure. The agent would reply *"I'll patch that"* or
*"Let me update the file"* — and then simply stop. No tool call, no edit, nothing. Asking
*"is it done?"* often produced another confident *"yes, doing it now"*, still with no tool
call. Work I believed was finished silently wasn't.

This is what the **action contract** at the top of `AGENTS.md` exists for: if the work needs
a tool call, the response must *contain* that tool call — never a promise of one in a later
turn. The contract names the banned response shapes explicitly (`"I'll do X." [stop]`) and
defines *done* as "tool call executed + result observed", because a vaguer instruction didn't
hold.

### 2. A tool returned plausible junk instead of failing

Transcribing a voice memo, the Whisper API skill returned the string `(see attached image)`
as the transcript. Not an error — a syntactically fine result that was completely wrong.
It survived a WAV conversion and an HTTP retry, which ruled out a file-format problem and
pointed at the provider.

The lesson that stuck: an agent failing loudly is a good day. The dangerous case is output
that looks right enough to pass, and the only defence is checking results against what you
expected rather than whether the call returned 200.

### 3. Fixing that broke the environment underneath

Installing `openai-whisper` locally as a workaround pulled in `numpy 2.2.6`, which conflicts
with the `tensorflow 2.16.2` my thesis pipeline needs (`numpy < 2.0`). The retry then failed
with `RuntimeError: Numpy is not available`, and the thesis environment was quietly broken
until NumPy was pinned back.

An agent with shell access can leave your machine in a worse state than it found it while
doing exactly what you asked. This is why `AGENTS.md` separates *safe local actions* from
things that need permission, and why `trash > rm`.

### 4. Unattended runs stall on interactive prompts

A sub-agent kept hanging on a first-run theme/login picker. Neither `--output-format text`
nor `--dangerously-skip-permissions` got past it, and the run just sat there. I wrote the
script by hand instead.

Automation assumes non-interactive tooling, and "it works when I run it" is not the same as
"it works when nothing is watching". The **loop guard** — if a tool fails the same way twice,
stop and report — comes from this: burning tokens retrying is worse than surfacing the block.

### 5. Silent model fallback

Sessions sometimes ran on a fallback model rather than the configured default. Behaviour
changes with the model — noticeably in how much it hedges and how readily it calls tools —
and nothing announces the switch. I only caught it because the session greeting reports the
runtime model when it differs from the default.

Worth knowing before you conclude your prompt regressed: check what actually served the turn.

### What this taught me about prompts

Vague instructions produce vague compliance. What measurably worked:

- **Be directive, not suggestive.** "Read before edit. Always." beats "it's usually a good
  idea to read the file first."
- **Bullet multi-step actions.** A paragraph describing four things gets two of them done;
  the same four as a numbered list gets four.
- **Name the failure mode explicitly.** Listing banned response shapes worked far better than
  positively describing the desired behaviour.
- **Define done.** If "done" isn't defined in terms of an observable result, it drifts into
  meaning "I intend to."

None of this was measured formally — no eval suite, no held-out set. It's an informal loop:
notice a failure, write a rule, watch whether it recurs. Worth being clear about that.

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

## Security notes

- The real config file contains live API tokens and a gateway auth token. It is **not**
  committed here.
- Use token-based gateway auth. Avoid setting a plaintext password in config.
- `credentials/`, `logs/`, and `telegram/` runtime state are all gitignored — they contain
  session data and pairing secrets that shouldn't leave the machine.
