# claude-md

This is how I work with [Claude Code](https://claude.com/claude-code) as of **2026-11-05**.

[`CLAUDE.md`](CLAUDE.md) holds the rules I keep in my global `~/.claude/CLAUDE.md`. Claude Code loads that file into every session, so these rules apply to everything I do with it.

## What these rules do

- **Orchestrator mode:** big jobs finish faster, and the chat stays free for
  talking to you.

  When you ask for it, Claude coordinates and subagents do the work in parallel,
  each in its own git worktree, routine agents on a cheaper model. Claude merges
  their work, watches memory, CPU and plan usage, and keeps a numbered backlog in `BACKLOG.md` and `DECISIONS.md`.

- **Away mode:** work keeps going while you're away, without nasty surprises when
  you return.

  Before you leave, Claude asks only the questions that would block hours of work.
  It works through the agreed backlog, takes sensible defaults instead of waiting
  on you, and never pushes, deploys, deletes data or spends money without
  approval. The backlog and the questions for later live in `BACKLOG.md` and
  `DECISIONS.md`, with everything from your absence marked as added while away.
  When you're back it reports what it finished and asks the questions it saved
  for later.

- **Rules that always apply:** you stay in control, and old code doesn't pile up.

  - Working on its own never means merging to the default branch or deploying.
  - "Discuss" and "plan" mean no file changes yet.
  - When something is replaced, the old version is removed in the same piece of work.

- **CPU and memory limits:** your machine stays responsive while Claude works.

  Claude's work shouldn't push the machine's CPU or memory usage over 83%. If it
  does, Claude lowers its usage by running fewer workers, using smaller batches
  and throttling the processes it started, and pauses its heavy work if that
  isn't enough. Works on Linux and macOS.

- **Precedence:** each project can set its own rules for pushing, merging and
  deploying.

  A project's own `CLAUDE.md` or memory overrides these rules.

## Use these rules yourself

You can copy all of my rules or only some of them. Paste this prompt into Claude Code:

```
Fetch https://raw.githubusercontent.com/n-dream/claude-md/main/CLAUDE.md and
show me each rule in it with a one-line summary. Ask me which rules I want,
then merge the ones I pick into my ~/.claude/CLAUDE.md:
- Create ~/.claude/CLAUDE.md if it doesn't exist yet.
- Keep everything that is already in the file.
- If one of my existing rules says the same thing as a new one or contradicts
  it, show me both and ask which to keep. Don't overwrite it.
- Show me the final diff before you save it.
```

To copy everything, replace "Ask me which rules I want" with "Take all of them".

### Sessions that are already running

Claude Code reads `~/.claude/CLAUDE.md` when a session starts, so new sessions pick up your changes on their own. A session that is already running keeps the version it started with. To apply your changes there, tell it:

```
Can you reload my ~/.claude/CLAUDE.md? Read it again and follow the
current version from now on.
```

Claude then reads the file again and follows the new rules for the rest of that session.

## License

Copy, adapt and share these rules however you like.
