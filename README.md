# session-handoff

A Claude Code skill for moving work between sessions: the brief a fresh, restarted or parallel session starts from,
parallel lanes on their own git worktrees, checking a lane's report before relaying it, working in a repository that
other sessions are using at the same time, and picking work back up after a gap. It starts from one observation: a
receiving session has only what is written where it starts and what it can check there. So whatever goes out stands
alone and points at durable records, and whatever comes back is a claim until it has been checked at its source.

## What it does

Claude Code loads the skill when a task matches its description: the user asks for a prompt, brief or kickoff for
another session, work is split into background sessions or worktree lanes, a lane reports done, other sessions are
live in the same repository or another session's report arrives, stalled work resumes after a gap, or a coordinating
session moves to another folder. You can also call it by typing `/session-handoff` in a session.

It has one section for each of five moments.

1. Hand off: choose the vehicle (the same session, a fresh one, a spawned task) and say why. Commit what lives only
   in the chat, such as decisions, agreed protocols and the user's standing instructions, to the repository the
   receiver works in, and make every file the brief cites reachable from the branch it starts on. Then fill every
   slot of a fixed brief template, from the goal and the first checks to the stopping points, the landing and what
   to report back. A slot with nothing to say gets "none".
2. Fan out: each lane gets the brief plus its own worktree and branch off the current tip, a scope no other lane
   shares, its own id block, port and slice of any other shared resource, and a row on a board committed to main.
   The coordinator owns the shared records. Lanes propose changes in their report, don't merge into main, and
   contact no one outside.
3. Take it back: before relaying a lane's result, confirm the branch tip is the final commit it reported, its
   worktree is clean and the session has stopped; read every commit; re-run the checks in the lane's worktree; check
   guardrails and id collisions; and test the claim the result most depends on against its primary source. Lanes
   merge one at a time, each brought up to date with main first, and a lane is closed out (board, orientation docs,
   worktree, branch) only once it is confirmed in main. A step that fails or can't be run has its own route, and an
   unrunnable check is never a pass.
4. Share a repo: assume other sessions are live. Work in your own worktree, stage your own files by path (no
   `add -A`, `commit -a`, stash or reset over other sessions' edits), re-read a shared document right before citing
   or editing it, and land on main through a guard: a fast-forward merge, then a check that the commit is in main,
   run again right before reporting it. A report from another session is a claim to reproduce before relaying it.
5. Re-enter: after a gap, check live channels before records (git state, inboxes, every place correspondence
   arrives), establish whether each staged draft went out before treating it as pending, rank what went stale by the
   cost of further delay, and write a re-entry brief. When the coordinating session moves to another folder, its
   working rules move into that folder's docs, because agent memory stays behind, and the new coordinator's first
   commit names it as the current coordinator in the status doc; any other session is relay-only from then on.

## Install

Claude Code reads a skill from a folder that holds a `SKILL.md`. Put this repo's `SKILL.md` in a folder named
`session-handoff` under `~/.claude/skills/` to use it in every project, or under `.claude/skills/` inside one
project. Nothing else is needed.

```sh
git clone https://github.com/craigstoller/session-handoff.git ~/.claude/skills/session-handoff
```

Or download `SKILL.md` and copy it to `~/.claude/skills/session-handoff/SKILL.md`.

## Limits

- It is one file of instructions, with no scripts or hooks. The rules it sets, such as the guard before a merge to
  main and the status-doc line that makes an old coordinator relay-only, hold only as far as each session follows
  them.
- Most of it assumes the work lives in a git repository: lanes, merges and landing checks work on branches,
  worktrees and commits.
- Its checks see one machine: `git worktree list` and the app's session list show only the sessions on this
  computer, so the skill tells you to assume others are live.
- Some of its terms come from the Claude desktop app: a spawned task or chip, and the app's session list.
- It does not cover parallel subagents inside one session. For those it points to two skills from the superpowers
  plugin, `dispatching-parallel-agents` and `subagent-driven-development`, which are installed separately.
- It leaves every send to the user. Where the work involves mail, it checks the sent folder, or asks the user,
  whether each staged draft went out before treating it as pending.

## Contributions

This repo is published read-only. Issues are off, and it accepts no pull requests. New versions arrive as commits
here, each release tagged.

## License

MIT. See `LICENSE`.
