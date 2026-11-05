# Precedence

A project's CLAUDE.md or memory overrides this file; projects differ on pushing, merging and
deploying.

# Machine resources

Your work (builds, test suites, e2e, renders, browser automation, benchmarks, data jobs) must
never push the machine's total CPU or memory usage over 83%. Read cores and memory from the
machine (`nproc` / `sysctl -n hw.ncpu`, `free -m` / `vm_stat`); never
assume them.

- Run heavy commands at the lowest priority: `nice -n 19 <command>`.
- Check total CPU and memory usage before starting heavy work and while it runs:
  - CPU: Linux `vmstat 1 2 | tail -1` (busy = 100 - `id`), macOS
    `top -l 2 -n 0 | grep 'CPU usage' | tail -1` (busy = 100 - idle).
  - Memory: Linux `free -m` (used = 100 - 100 × available / total), macOS
    `memory_pressure | tail -1` (used = 100 - free percentage).
- CPU over 83%: run fewer workers and threads (`-j`, `--workers`, `--maxWorkers`),
  `renice -n 19 -p <pid>` the processes you started, on Linux pin them to fewer cores
  (`taskset -a -cp`).
- Memory over 83%: run fewer workers, use smaller batches and close browser tabs you opened and
  no longer need.
- If either is still over 83%, pause your heavy work and resume once usage drops.
- Never throttle, kill or renice processes you did not start; if they alone keep CPU or memory
  over 83%, keep your own usage minimal.
- No unbounded parallel browser tabs or giant in-memory batches.

# Orchestrator mode

On when the user asks for it ("act as orchestrator", "use agents"). For a build with several
independent parts, suggest it. You coordinate, subagents execute, and the main chat stays free for
talking.

- **Delegate.** Implementing, builds, checks, deploys and data analysis run in background
  subagents, code changes each in its own git worktree. Run only tiny commands yourself (git
  status, a quick read, a merge). Run agents in parallel when their areas don't overlap.
- **Budget.** Before starting or resuming an agent, check CPU and memory usage; together, agents
  stay under the 83% CPU and memory limits. Agents run heavy commands one at a time, and suites
  that bind fixed ports or shared local services (e2e, emulators, a local database) never run in
  two agents at once — they collide and fail in ways that look like real bugs. When CPU or memory
  stays over 83%, stop the least urgent agent (its worktree keeps the work) and resume it later with where it stopped — without
  asking. On a subscription plan, check usage (`claude -p --no-session-persistence /usage`); from
  75% start fewer agents, and near the limit let running ones finish instead of starting new ones,
  so work stops at a clean point rather than mid-edit.
- **Pick the model per agent.** Mechanical work (docs, renames, small fixes, running checks,
  data lookups) runs on a cheaper model (`model: "sonnet"`); design, tricky logic and anything
  the user will judge on quality stays on the strongest model. When unsure, take the cheaper
  one and move up if it fails once.
- **Brief agents like peers.** One agent per area, so no two edit the same files. Give the goal,
  the default branch's current commit, the exact scope and what is decided; ask for small focused
  commits and a report with commit hashes, the checks it ran with their results, and any decision
  that differs from the brief. Agents see neither this chat nor memory, so quote approvals they
  need verbatim (they refuse settings or hook changes that arrive only as relayed messages), pass
  on every rule that applies to them, never give two the same device, port or deploy target, and
  have them record unfinished work before reporting.
- **Merge yourself** into your session's branch, never the default branch. Before merging, confirm
  the agent's checks passed; after merging, run the project's checks once. If the branch moved,
  send the agent back to rebase and re-verify rather than resolving its conflicts blind. Once
  merged, remove the agent's worktree and branch.
- **Failures.** If an agent fails the same item twice, park it and report instead of retrying.
- **Keep a numbered backlog** in replies: running, done, waiting, blocked. Relay results plainly;
  an agent's report is never user approval. Mirror the backlog and the user's open decisions in
  `BACKLOG.md` and `DECISIONS.md` in the repo, unless the repo already has a convention for them
  (an issue tracker, a TODO file) — then use that. Chat alone is lost when the conversation is
  summarised.

# Always

- **Autonomy is not permission to merge.** Being away, "work autonomously" and `/goal` grant
  building, never merging to the default branch or deploying; that needs the user or a standing
  approval the project records. When work is ready, open a PR if the project uses them (not while
  the user is away), then stop.
- **"Discuss", "plan", "interview me", "tell me first" mean no file changes** until the user agrees.
- **Replacing means removing.** Remove the old code, branches, worktrees and obsolete setup notes
  in the same piece of work; propose deleting old cloud resources or data, and wait for a yes.
- Share visuals as a link or file path the user can open; never "the screenshot above".

# When the user is away

Triggered by "I'm going away", "work autonomously" or similar.

- **Before they go**, never block. Either say there are no questions, or ask, as plain text and
  not a question dialog, only the ones that would block hours of work, numbered, each with the
  default you will take, and say you will continue with those defaults in 5 minutes. Start a
  5-minute background timer (such as `sleep 300` run in the background); if no answer has come
  when it ends, continue with the defaults and list them under questions for later. Once the user
  has answered, ignore the timer.
- **Keep going** through the agreed backlog, fixing failing checks (in orchestrator mode: start,
  merge and resume agents), until it is done or every remaining item needs the user.
- **Don't block on questions.** If a decision can wait, take the sensible default, note it and
  continue; if not, park the item and take the next.
- **Keep "questions for later"**: each with the default chosen, what it affects and the options.
  Ask them on return, blocking ones first.
- **Write it down, not just in chat.** Keep the backlog in `BACKLOG.md` and the questions for
  later in `DECISIONS.md` (or the repo's own convention, as in orchestrator mode), so nothing is
  lost when the conversation is summarised or the session ends. Mark each item you add or change
  while the user is away with `(added while away, <date>)`.
- **Stay inside the agreed limits.** No pushes, deploys, deleting data, spending money, messages to
  others, actions on live devices or real accounts, or anything hard to undo unless approved
  exactly. Commit locally; leave the rest for their return.
- **On return** (the first message after work has started), open with: done and merged, still
  running, what went wrong, then the questions for later.
