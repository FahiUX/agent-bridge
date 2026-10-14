# Agent Bridge 🌉

> **Cross-agent bridge + conversation sync for Claude Code & Antigravity (Gemini CLI).**
> Supports **tmux** (Mac / Linux) and **WezTerm** (Windows / Mac).
> Stop juggling two separate `conversation.md` files — peek the other agent's pane, compile one merged log, push once.

---

## The Problem
Running Claude Code and Antigravity side by side means two separate sessions, two separate conversation histories, and manually copying context (or files) between them. Each agent ends up pushing its own `conversation.md`, so the repo ends up with duplicate, disconnected logs of the same work.

## The Solution
This skill teaches an agent to:
1. **Peek** another agent's pane on request:
   - In **tmux**: via `tmux capture-pane` or `tmux-bridge-mcp`.
   - In **WezTerm**: via `wezterm cli get-text` (pure native Windows & Mac!).
2. **Compile** both sessions into one chronological, speaker-labeled `conversation.md` — small talk summarized, decisions/files/links kept verbatim.
3. **Push** a single, coherent log instead of two disconnected ones.

Nothing syncs automatically — this is a pull, triggered explicitly ("peek the other pane and compile"), not a shared memory between agents.

---

## Prerequisites

Run both agents in separate panes of the same multiplexer:

### Option A: In tmux (Mac / Linux)
```bash
tmux new-session -s agents
# split pane, run `claude` in one, `agy` in the other
```
Optional: https://github.com/howardpen9/tmux-bridge-mcp

### Option B: In WezTerm (Windows / Mac — No WSL needed!)
Run agents in split panes. The skill uses `wezterm cli` natively to inspect and capture peer panes without requiring tmux or Linux.

---

## How to Install on Your Home PC 🏠

### 1. Install Globally (For All Agents & Workspaces)
```bash
npx skills add FahiUX/agent-bridge -g
```

If you want to auto-confirm without prompts and install to all agents:
```bash
npx skills add FahiUX/agent-bridge -g --all
```

### 2. Install to Current Project Only
```bash
npx skills add FahiUX/agent-bridge
```

### 3. Updating When You Push New Rules
```bash
npx skills update sync -g
```

---

## Alternative: Manual Git Clone
```bash
git clone https://github.com/FahiUX/agent-bridge.git ~/.agents/skills/sync
```

---

## How to Use

Just say:
> `/sync`

Or naturally:
> *"Peek Antigravity's pane and compile our conversations into one file."*

Or after finishing a build in the other agent:
> *"Go peek the active pane, then push a synced conversation.md."*

With more than two agents running (e.g. two Antigravity + two Claude sessions), name the pane(s) explicitly so it's unambiguous which instance to pull from:
> `/sync panes 1 and 3`
