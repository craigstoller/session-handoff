---
name: session-handoff
description: Use when work moves between sessions - the user asks for a prompt, brief or kickoff for a fresh, restarted or parallel session, a spawned task or chip, another model or another repo; when splitting work into background sessions or worktree lanes; when a spawned session or lane reports done; when other sessions are live in the same repository or another session's report or relayed message arrives; or when resuming stalled work after a gap or moving a coordinating session to another folder.
---

# Session Handoff

## Overview

A receiving session has only what is written where it starts and what it can check there. So everything that
goes out stands alone and points at durable records, and everything that comes back is a claim until you have
checked it at its source: the repository for code and commits, the sent folder or the user for mail.

Five moments, one section each: **hand off** (write a brief), **fan out** (parallel lanes), **take it back**
(a lane reports done), **share a repo** (other sessions are live), **re-enter** (after a gap, or moving the
coordinator). In-session subagents for independent problems are the superpowers plugin's
`dispatching-parallel-agents`; executing a written plan through subagents is its `subagent-driven-development`.

## 1. Hand off: write the brief

### Before writing

1. **Choose the vehicle and say why.** Same session for a small follow-up. A fresh session when this context is
   heavy, captured by its own framing, or the task needs a reader who hasn't seen the current answer. A spawned
   task for a self-contained side job. If the user asked for "parallel" work and both separate sessions and
   in-session subagents would do, ask which.
2. **Write down what lives only in this chat.** Decisions, agreed protocols and the user's standing instructions
   go into the repository the receiver works in (the spec, a decision record, a conventions doc) and get
   committed, pushed only where that repository's rules already allow it; the brief points at them. Agent memory
   is keyed to one working folder and does not follow the work to another folder, worktree, clone or machine,
   and a fact that lives only in the prompt is gone the next time the prompt isn't pasted.
3. **Make every cited file reachable.** A new session or worktree starts from the last commit, often the
   remote's main. Get what the brief points at onto the branch it starts from (pushed, if it starts from a
   remote and the repository's rules allow; into a shared main, through section 4's guard); if you can't, name
   the branch and make it a precondition.

### The brief: fill every slot

```text
GOAL: <one task>. Status: <ratified | to be specced | exploratory>.
WHERE: run in <folder>. First check: working folder; branch; <sha> is in HEAD's history
  (git merge-base --is-ancestor <sha> HEAD); <preconditions>. If a check fails, stop and report;
  don't work around it.
READ FIRST, in order: <path, section> ... Repo docs outrank memory; if a doc and this brief disagree,
  stop and ask.
VERIFIED, don't re-derive: <fact> (source: <path>) ...
AS OF <date>, re-measure before relying on them: <counts, totals, states> ...
TO CHECK, inherited and unverified: <claim> (from <source>). Settle it with <check> before relying on it.
SETTLED, don't reopen: <decision> ...
OPEN, the user decides: <fork> ...
SCOPE: <numbered deliverables with paths>. MUST NOT TOUCH: <files, branches, records, outside contact>.
CONSTRAINTS AND TRAPS: <hard rules>; <known environment traps>.
STOP AND WAIT: <points where it stops for the user>.
VERIFY: <commands> -> <expected results>.
LANDING: branch <name>; commit there; <open a PR, don't merge | merge locally (not for a lane) | a lane: the
  coordinator merges>; push: <never | rule>.
REPORT BACK: branch and final commit; what changed; checks run and results; what you had to interpret;
  decisions left for the user, each with a recommendation; anything that needs the user. If you stop
  before done, commit what you have, say what is unfinished, and report the same way.
```

A slot with nothing to say gets "none", so the receiver knows it was considered.

### Before handing it over

- Re-read it for stale lines (merged PRs, deleted branches, moved paths, old counts) and for instructions that
  contradict each other.
- Output the prompt alone in one fenced block, nothing conversational inside it; comments go outside the block.
  If the brief holds code blocks of its own, fence the whole brief with four backticks.
- Always send the full brief, never a correction to a brief the receiver hasn't seen.
- Save a long or reusable brief as a committed file and point a receiver that can read the repo at it.
- A receiver that can't read the repository (another model in a chat window, another machine) gets what it
  needs inline instead of paths, and only what its task needs. Credentials never go into a brief, inline or
  committed.
- Name the intended model in the brief when the spawn mechanism doesn't carry one.
- Write commands for the shell that will run them.
- **Updating a running session:** say what it is producing now, number what changed, and end with an
  acknowledgement: it replies with its working folder, HEAD and two or three status items, then waits for your
  go-ahead. No acknowledgement means treat it as stalled (section 3) and hold its merge; where no message can
  reach a running session, relay through the user.

## 2. Fan out: parallel lanes

Each lane gets the brief above, with its worktree, branch, id block and port in its WHERE and CONSTRAINTS
slots, plus:

- **Commit first.** Lanes start from the last commit; uncommitted context is invisible to them.
- **Its own worktree and branch off the current tip.** Fetch first, and never start a new lane on an old lane's
  branch (resuming a stopped session on its own branch is different: section 3). A worktree lacks ignored files
  (env files, installed dependencies, local data), so say what to install, and point at the original env file
  rather than copying secrets into each worktree.
- **Scopes that don't overlap:** a file two lanes would need belongs to one of them or to the coordinator.
- **Disjoint shared resources:** an id block per lane for any sequential register (or lanes propose entries
  without ids and the coordinator numbers them at merge), its own port, its own browser session or profile, and
  its own slice of anything else shared, such as a test database, an account or a rate-limited API.
- **The coordinator owns shared records.** Orientation docs (status, index, open questions) and single-writer
  records (registers, manifests, decision log) are the coordinator's; lanes propose changes in their report.
  Append-only files merge by keeping both sides, in the file's own order (by id or date).
- **Lanes don't merge into main or contact anyone outside**, and push only what their LANDING slot allows.
- **Dependent lanes wait.** Hold them, or start them with a check that stops if the input is missing. Before
  launching a held lane, re-read what its siblings produced and rewrite or withdraw a brief that has gone stale.
- **Keep in the coordinator** anything that needs the user's logged-in browser or a connector lanes can't reach.
- **Track lanes on a board committed to main** (lane, branch, worktree path, id block, port, final commit,
  state: running, blocked, done or stalled; open decisions), not by session name: sessions get renamed and
  reports go astray.
- If something a running lane depends on changes, message it the full text of the change and ask for the same
  acknowledgement as an update to a running session.

## 3. Take it back: a lane or session reports done

"Landed", a completion notice, a pasted closing note and a file on disk are all claims. Before relaying any
result, run every step; when one fails or can't be run, go to "When a step fails" below instead of moving on.
Step 6, the merge, falls to whoever merges. Step 7, the close-out, falls to the coordinator (or, with none, the
session that dispatched the work) even when the user or a forge merged, and starts by confirming the landing.

1. **Finished?** The branch tip is the final commit the lane reported, its worktree is clean, and the session
   itself has stopped (its state in the session list, not just its report). Fetch first if the lane may have
   pushed from elsewhere.

   ```bash
   git rev-parse <branch>                    # must equal the reported final commit
   git log -1 --format="%h %ci %s" <branch>
   git -C <lane-worktree> status --short
   git log --oneline main..<branch>
   ```

2. **Read** every commit and the whole deliverable, not its headline.
3. **Re-run** install, tests, lint, build and any audit, in the lane's worktree.
4. **Check guardrails and collisions:** files it was told not to touch, a push or an outside contact it wasn't
   allowed, ids that collide with main or another lane, anything sensitive written where it shouldn't be.
5. **Test its sharpest claim** (the one the result most depends on) against the primary source before it
   reaches the user or anyone else.
6. **Merge one lane at a time.** Before each merge, bring that lane up to date with the current main (rebase it,
   or merge main into it; a branch already pushed for a PR gets main merged in, so nothing needs a force-push),
   re-run its checks, and re-read anything a conflict changed; its tip is now the final commit. Push an updated
   PR branch, so the PR's head is that commit. Into a shared main, use section 4's guard. A lane that left a
   decision for the user waits for that decision before it merges.
7. **Close out once the lane is confirmed in main:** for a local merge, by section 4's last check; for a PR,
   fetch, check the PR's head is that final commit, and check the PR's merge commit is in the remote's main
   (`git merge-base --is-ancestor <merge-commit> origin/main`) rather than trusting the badge. Then one
   reflection pass into the orientation docs and the board (done, final commit), remove the worktree, and
   delete the branch
   (`git branch -d` checks the branch's upstream when it has one, so it is no proof of landing; after a squash
   or rebase merge it refuses, and `-D` is right once the check above passed). Right before telling the user,
   run that landing check again; then tell them what landed, what now waits on them, and what is unblocked.

**When a step fails:**

- **Not finished, or stopped mid-work** (a usage or network limit, a crash): resume that session with a short
  "continue" plus any new instruction, since it keeps its own context. If it can't be resumed, tell the user what
  its worktree holds and let them choose: salvage it (commit it on the lane branch as partial, run steps 2 to 5
  on it, and merge it only once the user accepts it as partial) or brief a fresh lane from the board.
- **A check can't be run** (the branch isn't visible here, the worktree is gone, the session's state can't be
  read): fetch and look again; if it still can't be run, tell the user what is and isn't known. An unrunnable
  check is never a pass.
- **A check, guardrail or claim fails:** don't merge. Send the failure back to the lane if it is still running,
  or to the user with a recommendation.

## 4. Share a repo: other sessions are live

- **Assume they are.** `git worktree list` and the app's session list show only this machine; sessions in other
  clones or on other machines don't appear.
- **Work in your own worktree.** A branch checkout in a shared tree doesn't isolate: another session's
  checkout moves it under you.
- **Stage your own files by path.** Other sessions' uncommitted edits stay as they are: no `add -A`, `add -u`,
  `commit -a`, stash, reset or checkout over them, and no reset or force-move of a shared branch unless the user
  confirms nobody else is using it.
- **Re-read any shared document right before citing or editing it,** and note the version you used (a commit,
  or a content hash outside git).
- **Land on main with a guard,** one session at a time (in a fan-out, the coordinator). `<main-checkout>` is
  the main working tree of the same repository, and `<tip>` is the commit you are landing:

  ```bash
  git -C <main-checkout> status                     # on branch main, no merge or rebase in progress
  git -C <main-checkout> status --short --untracked-files=all   # others' uncommitted paths; none of yours
  git diff --name-only main...<branch>              # your paths; stop if a path is in both lists
  git -C <main-checkout> merge --ff-only <branch>   # refuses if main has moved: rebase, re-test, retry
  git -C <main-checkout> merge-base --is-ancestor <tip> main   # exit code 0: your commit is in main
  ```

  A merge can print success and still lose: another session's write to main (a reset, a forced update, a
  commit built on the old tip) can land seconds later and drop your commit. Main moving on past your commit is
  fine; your commit missing from main is not. If the last check fails, read `git reflog main` to see what moved
  it and correct anything you already reported as landed. If the removal looks deliberate (someone reset it),
  ask the user before landing it again; otherwise rebase, re-test and redo, and if it is lost a
  second time, stop and tell the user. Run the last check after a direct commit to main too (the board, orientation
  docs), and again right before you tell the user a commit landed. When main is pushed, fetch and run the same
  check against the remote's main (`origin/main`): that is the main a new session starts from.
- **A report from another session is a claim.** First ask whether your own recent work caused what it saw (a
  job you ran, a mass write, a check you triggered). Separate the observation from the diagnosis, and reproduce
  its key check before endorsing or relaying it. A reply leads with what the other session can act on, marks
  what you verified, and names where the report came from. If a relay you sent proves wrong, correct it at once
  and tell the user the first version may already be out.

## 5. Re-enter after a gap, or move the coordinator

1. **Live channels before records.** Check git state (including that work reported as landed is still in
   main), unprocessed inboxes, and every place outside correspondence actually arrives. If you can't reach one,
   ask the user for everything received there since the last recorded item before trusting the status doc. A
   connector that fails here may work in another session; recheck it before ruling it out.
2. **Drafted is not sent.** For each staged draft, establish whether it went out (the sent folder, or ask the
   user) before treating it as pending, and ask what went out that the record doesn't show. The user may have
   sent or changed things while away.
3. **Rank what went stale** by the cost of further delay, and write a recovery sequence that leaves every send
   to the user.
4. **Write a re-entry brief** in the repo: the handoff state (what went out, what is owed, the next user
   action) and a read order.
5. **Moving folders:** memory stays with the old folder. Before the move, move the working rules the session
   relies on into the new folder's docs (a conventions doc routed from its AGENTS.md or CLAUDE.md), and leave
   the old folder's memory a pointer to them.
6. **Start the new coordinator in the folder that owns the work** with a section 1 brief. Before it writes
   anything else, the old session has been stopped (the user stops it in the app) or, when it cannot be, is
   confirmed unreachable, and the new coordinator's first commit names itself in the status doc as the current
   coordinator, by handover time and folder rather than by session title (titles change), and names the session
   it replaces by its own start, relay-only from the handover; after a folder move the same line goes into the
   old folder's status doc too. That line is the fence, a rule every session keeps rather than a lock: before it
   writes to main or the shared records, or merges, a session re-reads the status doc, and one the line doesn't
   name merges nothing, writes nothing there, and passes on any report that still reaches it, as a claim, to the
   named coordinator. A session that was only unreachable can come back, so the new coordinator also tells the
   user that the old one, if it answers again, is relay-only. The new one reads and confirms first, and verifies
   any pending lane before merging.

## Common mistakes

| Mistake | Fix |
| --- | --- |
| Decisions exist only in chat or in the prompt | Commit them to the receiver's repo first |
| Brief cites a doc on an unmerged branch | Merge first, or make it a precondition |
| Inherited finding stated as fact | TO CHECK slot, with the check that settles it |
| Stale "don't touch" line about work that has merged | Re-read the brief before handing it over |
| Lead-in pasted along with the prompt | The prompt alone in one fenced block |
| A lane given an open-ended remit | Numbered scope, owned files, exact deliverable |
| Figures relayed from a mid-work snapshot | Section 3, step 1 before reading any result |
| A lane stopped mid-work and left waiting | Resume it, or the user picks salvage or a fresh lane |
| The user merged the PR, so nobody closed the lane out | The coordinator runs step 7, whoever merged |
| Parallel lanes take the same next id | Disjoint id blocks, or ids assigned at merge |
| A lane's edits swept into another session's commit | Own worktree; stage by path |
| Merge reported success, then main lost the commit | `merge-base --is-ancestor` after every merge, and again right before reporting it |
| "Nothing was sent" said from the drafts folder | Reconcile drafted against sent |
| Rules stopped loading after a folder move | Rules in repo docs, not agent memory |
| Old coordinator comes back after a takeover and keeps merging | The status doc names the current coordinator by handover time; any other session is relay-only |
