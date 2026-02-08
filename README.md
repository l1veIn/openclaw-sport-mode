# 🏎️ OpenClaw Sport Mode

> **Shift your agent into high gear.** 
> Temporarily boost OpenClaw's heartbeat frequency for intense monitoring tasks, then auto-cool-down when done.

![License](https://img.shields.io/badge/license-MIT-blue.svg) ![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-orange.svg)

## Why?

By default, OpenClaw checks in every 30 minutes. That's great for battery life, but terrible when you're:
- 🍿 Watching **Codex** write code (you want to see errors *now*).
- 🏗️ Waiting for a **long build** or deployment.
- 🎮 Playing a turn-based game with your agent.

**Sport Mode** lets you toggle a high-frequency heartbeat (e.g., 3m, 1m) and inject a "Mission" that the agent will obsessively check until completion.

## 🚀 Installation

### Manual
```bash
git clone https://github.com/l1veIn/openclaw-sport-mode.git ~/.openclaw/skills/sport-mode
```

## 🎮 Usage

### 1. Shift into Sport Mode (On)

```bash
# Default: 3-minute interval
sport-mode on --task "Monitor the build. If failed, notify me immediately."

# Turbo Mode: 1-minute interval
sport-mode on --task "Play idiom solitaire" --every "1m"
```

This will:
1. Hot-patch your `openclaw.json` to the new interval (Gateway reloads automatically).
2. Create/Overwrite `HEARTBEAT.md` with your task and a state machine template.

### 2. Cool Down (Off)

```bash
sport-mode off
```

This will:
1. Reset heartbeat to **30m** (default).
2. Clear `HEARTBEAT.md`.

## 🧠 Best Practices

### The "Auto-Pilot" Pattern
Sport Mode encourages a **Stateless Agent, Stateful File** pattern. 
Instead of relying on a massive conversation history context, let the agent read `HEARTBEAT.md`, perform one step, update `HEARTBEAT.md`, and sleep.

**Example HEARTBEAT.md during a game:**
```markdown
# 🏎️ Sport Mode Active
Target: 1m

## Task
Idiom Solitaire
State:
- Last: 天长地久
- Remaining: 1

## Auto-Off
If Remaining == 0, run `sport-mode off`.
```

### Silence is Golden
When running every 1 minute, don't let your agent spam you.
- **No change?** Reply `HEARTBEAT_OK` (silence).
- **Status changed?** Send a notification.

## License

MIT
