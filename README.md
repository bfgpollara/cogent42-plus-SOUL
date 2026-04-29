<p align="center">
  <img src="logo.png" alt="cogent42" width="120">
</p>

<h1 align="center">cogent42-plus-SOUL</h1>

<p align="center"><strong>Full Claude Code access on your server, controlled from Telegram — with a durable persona harness.</strong></p>

<p align="center"><em>1 file &nbsp;|&nbsp; 3 dependencies &nbsp;|&nbsp; 2-minute setup &nbsp;|&nbsp; The anti-platform.</em></p>

---

> **About this fork.** `cogent42-plus-SOUL` is a fork of [`cogent42/cogent42`](https://github.com/cogent42/cogent42) maintained at [`bfgpollara/cogent42-plus-soul`](https://github.com/bfgpollara/cogent42-plus-soul). All upstream functionality is preserved; this fork adds the **SOUL.md persona harness** — an optional markdown file that injects a durable persona, tone, and working-style guidance into Claude's runtime context on every query and scheduled task. See the [Persona (SOUL.md)](#persona-soulmd) section for details, and credit to the upstream project for everything else.

cogent42 is a single-file Telegram bot that gives you complete Claude Code capabilities on any server -- bash, files, search, scheduled tasks, persistent memory -- all from your phone. No plugins, no YAML, no Docker. Clone it, run the setup, message your bot.

## Features

### Added in this fork

- **SOUL.md persona harness** -- optional `SOUL.md` file in the repo root defines a durable persona, tone, and working style that is injected into Claude's runtime context on every query and scheduled task. See [Persona (SOUL.md)](#persona-soulmd).

### Inherited from upstream cogent42

- **Full Claude Code access** -- bash, file read/write/edit, and search via Telegram
- **Photo & document support** -- send images and files, Claude processes them on your server
- **Live progress updates** -- see what Claude is doing in real-time with turn counter (`[3/25] Using tool: bash`) via editable messages
- **Acknowledgment reactions** -- instant visual feedback (👀 on receive, 👍 on success, 👎 on error)
- **Scheduled tasks** -- schedule recurring tasks in plain English (e.g. "check disk space every morning at 9am")
- **Interactive confirmations** -- inline keyboard buttons for destructive actions like /reset and /opus
- **Persistent memory** -- conversations survive restarts through session resume
- **Automatic knowledge extraction** -- facts, decisions, and server config are pulled from conversations into a knowledge base (capped at 1,000 entries, auto-pruned)
- **Cross-session context** -- knowledge is injected into new sessions so Claude remembers what matters
- **Smart model routing** -- defaults to Claude Sonnet 4.6, auto-escalates to Opus 4.7 on failure, auto-reverts after success
- **Manual model switching** -- `/opus` and `/sonnet` commands
- **Mid-task context injection** -- send follow-up messages while the bot is working. Short messages ("also check logs", "use port 8080") are auto-injected with a ⚡ reaction. Longer messages get inject/queue buttons so you choose
- **Smart message queue** -- messages that aren't injected are queued and auto-processed in order after the current task finishes
- **Cancel in-flight queries** -- `/cancel` aborts the current query
- **Bot personality** -- optional personality config that also evolves through knowledge extraction
- **Graceful shutdown** -- in-flight queries and scheduled jobs are cleanly aborted on SIGINT/SIGTERM
- **Session resume fallback** -- automatically starts a fresh session if resume fails
- **One-command updates** -- `/update` pulls the latest version from GitHub, restarts the bot, and confirms it's back online
- **Automatic session cleanup** -- archived sessions older than 180 days are automatically deleted on startup

## Prerequisites

- Node.js 18+
- A [Claude Code](https://claude.ai/claude-code) subscription (the setup script installs the CLI if needed)
- A Telegram bot token (from [@BotFather](https://t.me/BotFather))

## Quick Start

**Automated setup (recommended):**

```bash
git clone https://github.com/bfgpollara/cogent42-plus-soul.git
cd cogent42-plus-soul
node setup.js
```

The interactive setup walks you through everything.

**Manual setup:**

```bash
git clone https://github.com/bfgpollara/cogent42-plus-soul.git
cd cogent42-plus-soul
cp .env.example .env
# Edit .env with your tokens
npm install
node bot.js
```

> Prefer the upstream version without SOUL.md? Clone [`cogent42/cogent42`](https://github.com/cogent42/cogent42) instead.

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Yes | -- | Bot token from [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_USER_ID` | Yes | -- | Your user ID from [@userinfobot](https://t.me/userinfobot) |
| `MAX_TURNS` | No | `25` | Max agentic turns per query |
| `WORKING_DIRECTORY` | No | `~` | Directory where Claude operates |
| `BOT_NAME` | No | `cogent42` | Display name for your bot |
| `BOT_PERSONALITY` | No | -- | Optional personality (e.g. "concise and direct") |
| `DISABLE_KNOWLEDGE_FALLBACK` | No | -- | Set to `true` to disable cross-bot knowledge fallback (registration and sibling lookup) |

## Commands

| Command | Description |
|---|---|
| `/start` | Welcome message |
| `/reset` | Clear conversation, extract knowledge, start fresh |
| `/cancel` | Cancel the current in-flight query |
| `/schedule` | Schedule a recurring task in plain English |
| `/schedules` | View and manage scheduled tasks |
| `/unschedule` | Remove a scheduled task by ID |
| `/status` | System info (version, uptime, disk, memory, model, session, schedules) |
| `/history` | Session stats |
| `/opus` | Switch to Opus 4.7 |
| `/sonnet` | Switch to Sonnet 4.6 |
| `/knowledge` | View stored knowledge entries |
| `/update` | Update bot to the latest version and restart |

## Scheduling

Schedule recurring tasks in plain English -- no cron syntax needed:

```
/schedule check disk space every morning at 9am
/schedule remind me about deploy every friday at 5pm
/schedule run backup every sunday at 2am
/schedule check server health every 30 minutes
```

Claude parses your intent into a schedule. Tasks persist across restarts. Manage them with `/schedules` (interactive buttons) or `/unschedule <id>`.

## Persona (SOUL.md)

> This feature is the distinguishing addition of the `cogent42-plus-SOUL` fork; it is not present in upstream `cogent42`.

`SOUL.md` is an optional markdown file placed in the repo root (alongside `bot.js`). If present, its contents are appended to Claude's runtime system context on every query and scheduled task. Use it for durable persona, tone, and working-style guidance.

- **Location:** `SOUL.md` in the repo root
- **Missing file:** silently skipped, no error
- **Unreadable file:** a warning is logged to stderr and the bot continues without it
- **Size cap:** content beyond 12,000 characters is truncated with a warning
- **Precedence:** hardcoded safety rules > `SOUL.md` > `BOT_PERSONALITY` env var > persistent knowledge and context

`BOT_PERSONALITY` still works and is applied in addition to `SOUL.md`. Use `BOT_PERSONALITY` for a quick one-liner personality, and `SOUL.md` for richer persona definitions.

A starter template lives at `SOUL.example.md` — copy it to `SOUL.md` to activate:

```bash
cp SOUL.example.md SOUL.md
```

## How Knowledge Works

cogent42 maintains a persistent knowledge base across conversations:

1. After every **10 conversation turns**, cogent42 extracts facts and decisions from the conversation.
2. On `/reset`, knowledge is **always extracted** before the session is archived.
3. On **shutdown or session expiry**, knowledge is extracted before the session is discarded.
4. When a new session starts, cogent42 **scores every knowledge entry against your first message** and injects only the most relevant entries (up to 30) plus all rules -- not the entire knowledge base.
5. If your first message is generic (e.g. "hi"), it falls back to the **most recent** entries.
6. Knowledge is capped at **5,000 entries** -- oldest normal entries are pruned when the limit is reached; permanent entries are never dropped.

This means the context window stays small (~2K tokens for knowledge) regardless of how large the knowledge base grows, while still surfacing the right context for every conversation.

### Fallback knowledge from sibling bots

When you run multiple cogent42 bots on the same host, each one auto-registers itself in `~/.cogent42/instances/` on startup. During context injection, each bot reads its siblings' knowledge files as a **read-only fallback pool** — so bot B can surface something bot A learned. Factual categories only (`server`, `project`, `config`, `bug`, `workflow`, `mistake`, `decision`); `preference` and `rule` stay private so each bot's personality and corrections are its own.

Entries borrowed from siblings are tagged on the way into the prompt, e.g. `[project from bot-a]`, so the bot knows they're not its own memory. Stale markers (lastSeen older than 30 days) and markers pointing at missing files are skipped automatically — no cleanup needed.

No configuration. To opt out: `DISABLE_KNOWLEDGE_FALLBACK=true`.

## Architecture

```
cogent42-plus-soul/
  bot.js                  # Single-file entry point (ESM)
  ecosystem.config.cjs    # PM2 config (.cjs because project is ESM)
  setup.js                # Interactive setup script
  SOUL.md                 # Optional persona harness (fork addition; see Persona section)
  SOUL.example.md         # Starter template for SOUL.md
  memory/                 # Session files + current.txt for persistence
  knowledge/              # knowledge.json, schedules.json
```

## Running with PM2

For production use, run cogent42 with PM2 for process management and automatic restarts.

```bash
# Start
pm2 start ecosystem.config.cjs

# View logs
pm2 logs cogent42

# Check status
pm2 status

# Restart / Stop
pm2 restart cogent42
pm2 stop cogent42

# Survive reboots
pm2 save && pm2 startup
```

## Security

- **Single user** -- the bot is locked to one Telegram user ID. Unauthorized users are silently rejected.
- **Never commit secrets** -- `.env`, `memory/`, and `knowledge/` are in `.gitignore`.
- **File downloads** -- photos and documents sent via Telegram are saved to the working directory for Claude to process.

> **Warning:** cogent42 uses `bypassPermissions` mode, which gives Claude **unrestricted access** to your server -- bash, file system, everything. Only run this on machines you are comfortable giving full access to.

## Development

This fork tracks upstream `cogent42/cogent42` and layers fork-specific changes (notably the SOUL.md persona harness) on top. Work happens on feature branches and is merged into the fork's default branch via PR.

The upstream release scripts in `package.json` (`release:patch` / `release:minor` / `release:major`) push tags to whichever remote `main` you have configured, so adapt them to this fork's release process before invoking.

## Credits

- Original project: [cogent42/cogent42](https://github.com/cogent42/cogent42) — all the upstream features listed above were built there.
- This fork: [bfgpollara/cogent42-plus-soul](https://github.com/bfgpollara/cogent42-plus-soul) — adds the SOUL.md persona harness.

## License

MIT (inherited from upstream)
