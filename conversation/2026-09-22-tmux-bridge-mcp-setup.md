# tmux-bridge-mcp — Setup Conversation

**Date:** 2026-09-22
**Repo studied:** https://github.com/howardpen9/tmux-bridge-mcp
**Goal:** Get side-by-side multi-agent CLI setup on Windows using WSL + tmux + tmux-bridge-mcp

---

## 1. Studying the repo

**User:** https://github.com/howardpen9/tmux-bridge-mcp let study this.

**Claude:**
`tmux-bridge-mcp` is a standalone MCP server that lets multiple AI CLIs (Claude Code, Codex, Gemini, Kimi, Copilot CLI, etc.) talk to each other by reading/writing into each other's **tmux panes**. tmux itself is the transport — no daemon, no websockets, no framework.

### Core idea
Each AI agent runs in its own tmux pane. The MCP server exposes 9 tools that wrap `tmux capture-pane` / `send-keys` / `list-panes`. Any MCP-capable agent gets cross-pane superpowers by just adding a config entry.

### The 9 tools
`tmux_list`, `tmux_read`, `tmux_type`, `tmux_message`, `tmux_keys`, `tmux_name`, `tmux_resolve`, `tmux_id`, `tmux_doctor`.

### Key design: read-act-read guard
Write tools (`tmux_type`/`message`/`keys`) fail unless you called `tmux_read` on the target first. Prevents blind typing into the wrong pane. Sequencing aid, **not** a security boundary.

### Install
```
npx tmux-bridge-mcp setup   # auto-configures every detected CLI
npx tmux-bridge-mcp demo    # 3-pane live demo
```

### When it wins vs alternatives
- **vs LangChain/CrewAI**: no framework rewrite — keeps your existing CLIs as-is.
- **vs Claude Code subagents**: subagents are fire-and-forget; tmux panes carry **persistent conversation context** across turns.
- **vs smux**: MCP tool calls instead of bash CLI + skill injection; no tmux.conf changes.

### Limitations
- Local single-machine only, no auth, any agent can read/write any pane.
- Buffer truncation possible; native subagents more reliable for structured returns.
- Extra operational layer (labels, pane IDs).

---

## 2. Learning tmux basics

**User:** yes i understand well. so how put site by site panel? is it hard to learn? or just simple

**Claude:**
Super simple. tmux has ~4 keys you'll ever need.

Start a session:
```
tmux
```

Everything below starts with **prefix** = `Ctrl+b` (press, release, then hit the next key).

| Do this | Keys |
|---|---|
| Split **vertically** (side-by-side) | `Ctrl+b` then `%` |
| Split **horizontally** (top/bottom) | `Ctrl+b` then `"` |
| Jump between panes | `Ctrl+b` then arrow key |
| Close a pane | just type `exit` in it |
| Detach (leave running) | `Ctrl+b` then `d` |
| Come back later | `tmux attach` |

### Typical AI-agent layout
```
tmux                      # start
Ctrl+b %                  # split vertical → 2 panes side by side
claude                    # run Claude Code in left pane
Ctrl+b →                  # move to right pane
codex                     # run Codex in right pane
Ctrl+b %                  # split right pane again → 3 panes
gemini                    # run Gemini in the new one
```

### Windows note
tmux is Unix. On Windows you run it inside **WSL** (Ubuntu):
```
wsl
sudo apt install tmux
tmux
```

---

## 3. Downloading tmux (WSL required)

**User:** ok let download this tmux i want to try.

**Claude:** WSL isn't installed yet. tmux is Linux-only, so on Windows you need WSL first.

### Steps
1. Open PowerShell as **Administrator**
2. Run: `wsl --install`
3. Reboot
4. Ubuntu opens, set username/password
5. In Ubuntu:
   ```bash
   sudo apt update && sudo apt install -y tmux nodejs npm
   tmux
   ```

---

## 4. Setup ran on Windows first

**User:** npx tmux-bridge-mcp setup

**Result:**
```
Detecting agents...
  ✓ Claude Code (claude) — config written to ~\.claude.json
  ✗ Gemini CLI (gemini) — not found
  ✗ Kimi CLI (kimi) — not found
  ✗ Codex CLI (codex) — not found
```

**Note:** Setup wrote MCP config to `~/.claude.json` but tmux itself was not installed. Tools would fail with "tmux not found" until WSL + tmux installed.

---

## 5. Installing WSL

**User:** my bad2. let download tmux

**Claude:** Attempted `wsl --install -d Ubuntu` — failed because needs Administrator privileges. Instructed user to run in Administrator PowerShell manually.

User ran it, rebooted, entered Ubuntu.

---

## 6. First time inside tmux

**User:** ok bro i cannot ctrl b %[Image #1] im corrent or not?

**Screenshot shows:** User already inside tmux (green bar visible), typed `tmux` again (got "sessions should be nested" warning), then typed `sudo TMUX` and `$TMUX` by mistake.

**Claude clarified:**
- User was ALREADY inside tmux (green bar = tmux status bar)
- `Ctrl+b %` is two-step: hold Ctrl+b, release, THEN press Shift+5
- Don't type `sudo TMUX` or `$TMUX` — those aren't commands

---

## 7. The Antigravity confusion

**User:** i cannot use agy? command? antigravity?

**Claude (initial wrong answer):** Antigravity is a GUI IDE, won't work with tmux-bridge.

**User:** i use claude code and antigravity?

**Claude:** Explained that Antigravity's GUI can't be bridged via tmux, only CLIs can.

**User:** anti gravity is cli right?

**Claude:** Insisted it was a GUI-only IDE.

**User:** waht are the different between antigirty and gemini? is the same right?

**Claude:** Explained Gemini (the model) vs Antigravity (IDE product using Gemini) vs Gemini CLI (terminal tool).

**User:** https://antigravity.google/download bro dont lie se cli

**Claude fetched the page — corrected mistake:**
Antigravity DOES have a CLI (v1.2.6):
| Product | Type | Works in tmux? |
|---|---|---|
| **Antigravity CLI v1.2.6** | Terminal command | ✅ YES |
| Antigravity 2.0 (desktop) | GUI app | ❌ |
| Antigravity IDE (v2.5.5) | GUI app | ❌ |

Apology issued. 🙏

---

## 8. The two-worlds problem

**User:** but i cannot run agy in tmux why?

**Claude:** Antigravity CLI installed on **Windows**, but tmux runs inside **WSL Ubuntu** — two separate worlds. `agy` doesn't exist in the Linux side yet. Solution: install Linux version of the CLI inside WSL.

**User:** hmm i dont get it. help me.

**Claude explained with diagram:**
```
┌─────────────────────────────────────────┐
│  WINDOWS (your normal PC)                │
│  ─ Antigravity IDE installed here        │
│  ─ agy command installed here            │
└─────────────────────────────────────────┘
        │
        │  (they don't share programs)
        │
┌─────────────────────────────────────────┐
│  UBUNTU (WSL, the new Linux one)         │
│  ─ tmux runs HERE                        │
│  ─ Empty — no agy, no claude yet         │
└─────────────────────────────────────────┘
```

Steps given:
1. `sudo apt install -y curl nodejs npm`
2. `npm install -g @anthropic-ai/claude-code`
3. Install Antigravity CLI Linux script inside Ubuntu
4. `which claude` / `which agy` to verify

---

## 9. Package errors

**User screenshot:** `Unable to locate package nodejs / npm`

**Claude fix:**
```bash
sudo apt update && sudo apt install -y curl git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g @anthropic-ai/claude-code
```

---

## 10. Claude Code installed in Ubuntu

**Screenshot:** `2.1.197 (Claude Code)` ✅

**Claude next step:**
```bash
npx -y tmux-bridge-mcp setup
```

---

## 11. tmux-bridge wired in Ubuntu

**Screenshot output:**
```
Detecting agents...
  ✓ Claude Code (claude) — config written to ~/.claude.json
  ✗ Gemini CLI (gemini) — not found
  ✗ Kimi CLI (kimi) — not found
  ✗ Codex CLI (codex) — not found
```

---

## 12. Split panes success

**Screenshot:** Claude Code running in left pane, empty bash in right pane, one tmux session, green status bar at bottom. ✅

Warnings (harmless):
- `SessionStart:startup hook error /bin/sh: 1: cmd: not found` — Windows-specific hook leaking into Linux
- `Auto-update failed: no write permission to npm prefix` — because `sudo npm install`

**Test prompt suggested:**
```
List all tmux panes using the tmux_list tool
```

---

## 13. tmux-bridge FULLY WORKING 🚀

**Screenshot:** Claude used `tmux_list` and returned a table of **6 panes across 3 windows**, including itself (`%5` = claude) and the bash pane next to it (`%6`).

```
Target ID | Window | Process | Label | Working Directory
%0        | 0:0    | sudo    | none  | /mnt/c/WINDOWS/system32
%1        | 0:0    | bash    | none  | /mnt/c/WINDOWS/system32
%2        | 0:0    | bash    | none  | /mnt/c/WINDOWS/system32
%4        | 2:0    | bash    | none  | /mnt/c/Users/fahi
%5        | 3:0    | claude  | none  | /mnt/c/Users/fahi
%6        | 3:0    | bash    | none  | /mnt/c/Users/fahi
```

---

## Final state

- ✅ WSL2 + Ubuntu installed
- ✅ Node.js 20 + npm installed
- ✅ Claude Code 2.1.197 installed in Ubuntu
- ✅ tmux-bridge-mcp configured
- ✅ tmux running with split panes
- ✅ Claude can see and control other panes via MCP tools
- 🟡 Next: install Antigravity CLI (Linux version) or Codex/Gemini CLI in second pane

## Lessons learned

1. **Windows programs ≠ Linux programs** — anything you want in tmux must be installed inside WSL/Ubuntu.
2. **Antigravity has a CLI** — I was wrong initially. Always check the source of truth.
3. **tmux prefix is two-step** — `Ctrl+b` release → then the next key.
4. **`sudo npm install -g`** works but breaks auto-update. Fix later with nvm or `chown -R $USER ~/.npm`.
5. **tmux-bridge just needs one setup command** — auto-detects installed CLIs and writes MCP config for each.
