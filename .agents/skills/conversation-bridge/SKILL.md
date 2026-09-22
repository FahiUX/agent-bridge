---
name: conversation-bridge
description: Peeks another AI agent's tmux pane (e.g. Antigravity/Gemini CLI) to pull its conversation, compiles it with this agent's own conversation into one chronological, speaker-labeled conversation.md, and pushes it to the current repo. Use when the user says "peek the other pane", "compile both conversations", "sync chat", or asks to merge session history across two agent panes before pushing to GitHub.
---

# Conversation Bridge

Merges conversation history across two agents (this one + a peer running in another tmux pane, e.g. Antigravity) into a single `conversation.md`, instead of each agent pushing its own separate file.

Requires `tmux-bridge-mcp` (or manual tmux) so this agent can read another pane's content on request. This is a **pull, not a push** — nothing merges automatically. The user must trigger it explicitly (e.g. "peek Antigravity's pane and compile").

## Steps

1. **Identify the peer pane.** Ask the user which tmux pane the other agent is running in if not already known (e.g. `tmux list-panes -a` to enumerate, or the user just tells you "pane 1").

2. **Read the peer's session.**
   - Via `tmux-bridge-mcp`: use its pane-read tool against the target pane.
   - Manual fallback: `tmux capture-pane -p -t <target-pane> -S -` (the `-S -` grabs full scrollback, not just the visible screen).

3. **Compile into one file, chronological order.** Interleave both agents' turns by when they happened (not grouped by agent) — the merged log should read as one continuous story, since that's how the work actually happened.

4. **Format each entry:**
   - **Small talk / back-and-forth** (clarifying questions, acknowledgments, "let me check") → compress to one line summarizing what was discussed.
   - **Decisions, file paths, links, code snippets, commands run** → keep verbatim, never summarize these away.

   ```
   [HH:MM] Antigravity: discussed relaxation approach
   [HH:MM] Antigravity: built webpage — see /webpage/index.html
   [HH:MM] Claude: pulled webpage, pushed to Figma — file: <link>
   ```

5. **Write `conversation.md`** at the repo root (or wherever the user's existing convention places it — check for an existing file first rather than assuming root).

6. **Push.** Stage and commit `conversation.md` with a short commit message (e.g. `chore: sync conversation log`). Confirm with the user before pushing if this repo has uncommitted work already in progress that isn't part of this sync.

## Notes

- Never fabricate what the peer agent said — if the pane can't be read (dead session, wrong pane number), say so and ask the user to confirm the pane instead of guessing content.
- If both conversations are already partially compiled from a previous sync, only append the new turns since the last sync — don't re-summarize the whole history every time.
