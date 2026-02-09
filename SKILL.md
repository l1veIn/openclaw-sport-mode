---
name: sport-mode
description: Activate "Sport Mode" for high-frequency monitoring (3m heartbeat) and auto-cleanup. Use when supervising intense tasks (Codex, builds, migrations).
metadata:
  {
    "openclaw": { "emoji": "🏎️" }
  }
---

# Sport Mode

Temporarily boost heartbeat frequency (default 3m) and inject a monitoring task into `HEARTBEAT.md`.
Perfect for supervising background agents (Codex), long-running builds, or interactive games.

## Usage

```bash
# Turn ON: Set heartbeat to 3m and set monitoring task
sport-mode on --task "Check Codex progress. If done, run sport-mode off."

# Custom Interval: Set to 1 minute
sport-mode on --task "Game tick" --every "1m"

# Turn OFF: Reset heartbeat to 30m and clear HEARTBEAT.md
sport-mode off
```

## How it works

1.  **ON**:
    - Patches `~/.openclaw/openclaw.json` (hot-reload) to set `heartbeat.every`.
    - Writes your task to `HEARTBEAT.md` with a "Sport Mode Active" header.
2.  **OFF**:
    - Patches config back to `30m` (default).
    - Clears `HEARTBEAT.md`.

## Best Practices (Critical)

### 1. State Machine Pattern (Stateful File, Stateless Agent)
**Treat `HEARTBEAT.md` as a dynamic progress bar, not a static sticky note.**
Since you lose chat context between heartbeats, you MUST write your current state and next step into the file.

*   **Bad**: "Monitor M1-M4." (Agent forgets which milestone it's on)
*   **Good**: "Current: M1. If done, update this file to 'Current: M2'."

### 2. The "Liveness Report" Rule
**Prevent silent death.** Long-running tasks can look like they're working when they're actually stuck.
To avoid checking in forever with `HEARTBEAT_OK`:
*   **Rule**: If you have sent `HEARTBEAT_OK` for **approx. 30 minutes** (e.g., 30 times @ 1m, or 6 times @ 5m), you MUST break silence and send a status update.
*   **Format**: "Still running [task]... Last log: [snippet]"
*   This reassures the user that the process hasn't hung.

### 3. Use tmux for Visibility
If the monitoring task involves terminal output (e.g., Codex coding, compiling), running the task in a **tmux session** is ideal.
*   The agent can inspect the pane (`tmux capture-pane`) without interfering.
*   The user can attach (`tmux attach`) to watch live.
*   **Tip**: Append `; echo Done; read` to your command. This keeps the tmux window open if the process crashes or exits, allowing you to inspect logs.
    *   ✅ `tmux new-session -d -s task "codex ...; echo Done; read"`
    *   ❌ `tmux new-session -d -s task "codex ..."` (Window closes on exit)

### 4. Set a Finish Line
Unless you want an endless marathon, always define a **termination condition**.
*   ✅ "If success or fail, run sport-mode off."
*   ❌ "Monitor build."

## Template: HEARTBEAT.md

When activating sport mode, write `HEARTBEAT.md` using this structure:

```markdown
# 🏎️ Sport Mode Active
Target: 1m

## Task
Count down from 5.
State: 5

## Instructions
1. Decrement state (5 -> 4).
2. If State > 0, update this file with new state.
3. If State == 0, run: `sport-mode off`.
4. If State == 0, notify user: "Countdown finished!"
```

## Implementation Note
This skill uses `openclaw config set` to safely patch configuration at runtime, triggering a seamless Gateway reload.
