---
name: sync
description: Peeks one or more other AI agents' tmux panes (e.g. Antigravity/Gemini CLI, or other Claude Code instances) to pull their conversations, compiles them with this agent's own conversation into one chronological, speaker-labeled conversation.md, and pushes it to a confirmed repo. Use when the user says "/sync", "peek the other pane", "compile our conversations", or asks to merge session history across multiple agent panes before pushing to GitHub.
---

# Sync

Merges conversation history across two or more agents (this one + N peers, each running in its own tmux pane) into a single `conversation.md`, instead of every agent pushing its own separate file.

Requires `tmux-bridge-mcp` (or manual tmux) so this agent can read another pane's content on request. This is a **pull, not a push** — nothing merges automatically. The user must trigger it explicitly (e.g. `/sync`, "peek panes 1 and 3 and compile").

Works the same whether there are 2 panes or 6 — the only thing that scales with agent count is how precisely each pane must be identified.

## Steps

1. **Identify every peer pane involved.** With only one other agent running, "the other pane" is unambiguous. With multiple instances of the same agent (e.g. two Antigravity sessions), pane numbers or names are required — don't guess which one the user means.
   - Enumerate candidates: `tmux list-panes -a` (or `tmux list-windows` if panes are split across named windows).
   - If the user's request doesn't name the pane(s) explicitly (e.g. `/sync` with nothing else), ask which pane(s) to pull from before reading anything.

2. **Read each peer's session.**
   - Via `tmux-bridge-mcp`: use its pane-read tool against each target pane.
   - Manual fallback: `tmux capture-pane -p -t <target-pane> -S -` (the `-S -` grabs full scrollback, not just the visible screen).

3. **Compile into one file, chronological order.** Interleave every agent's turns by when they happened (not grouped by agent/pane) — the merged log should read as one continuous story, since that's how the work actually happened. Label each entry with a name that disambiguates instances (e.g. `Antigravity-1`, `Antigravity-2`, `Claude-1`, `Claude-2` — not just `Antigravity`/`Claude` if more than one of each is present).

4. **Format each entry:**
   - **Small talk / back-and-forth** (clarifying questions, acknowledgments, "let me check") → compress to one line summarizing what was discussed.
   - **Decisions, file paths, links, code snippets, commands run** → keep verbatim, never summarize these away.

   ```
   [HH:MM] Antigravity-1: discussed relaxation approach
   [HH:MM] Antigravity-1: built webpage — see /webpage/index.html
   [HH:MM] Claude-1: pulled webpage, pushed to Figma — file: <link>
   ```

5. **Confirm the destination repo before writing anything.** Never assume the current working directory is the intended repo. Ask the user which repo (and path within it — root, or an existing `conversation.md` location if one already exists) unless they already named it in this same request.

6. **Write `conversation.md`** at the confirmed path.

7. **Push.** Stage and commit `conversation.md` with a short commit message (e.g. `chore: sync conversation log`). Confirm with the user before pushing if this repo has uncommitted work already in progress that isn't part of this sync.

## Notes

- Never fabricate what a peer agent said — if a pane can't be read (dead session, wrong pane number), say so and ask the user to confirm the pane instead of guessing content.
- If the conversations are already partially compiled from a previous sync, only append the new turns since the last sync — don't re-summarize the whole history every time.
