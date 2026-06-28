# Five Loop Failure Modes

From: Loop Engineering — The Anthropic Playbook (IEEE working note, 2026)

Each failure mode is one of the five moves skipped or done badly.
They map one-to-one onto the moves.

```
Blind loop       ← Discovery skipped
Tangled loop     ← Handoff skipped
Nodding loop     ← Verification skipped
Amnesiac loop    ← Persistence skipped
Manual loop      ← Scheduling skipped
```

---

## The Nodding Loop (Verification skipped)

**What happens:** The loop runs, the agent writes code, and the same agent declares
it good. With no independent check, every turn produces self-approved output. The
loop accumulates plausible-looking mistakes at machine speed.

**Diagnostic signal:** A loop that has never once produced a REJECT across hundreds
of turns. A real workload always has failures. If the evaluator has never said no,
it isn't functioning as an evaluator.

**Secondary signal:** PRs that merge with green tests but introduce behavior regressions
discovered later in production.

**Fix:** Install the generator/evaluator split. The evaluator must be a separate
agent with separate instructions, defaulting to doubt. It must execute code, not
read it. The final stop condition must be checked by a fresh model, not the generator.

---

## The Amnesiac Loop (Persistence skipped)

**What happens:** The loop discovers good work, does it, and then forgets it happened,
because the result lived only in a context window that was flushed. The next turn
rediscovers the same work, or worse, redoes it and conflicts with the first attempt.

**Diagnostic signal:** Each morning the loop starts from the same place. No cumulative
progress across runs. The state file either doesn't exist or isn't being committed.

**Secondary signal:** Duplicate PRs for the same finding. Conflicting changes in
the same file from different runs.

**Fix:** A state file on disk — `./state/triage.md` committed to the repo after
every run. The agent forgets; the repo does not. Also establish an `./inbox/` for
findings the agent can't confidently handle, so they aren't lost and aren't acted
on blindly.

---

## The Manual Loop (Scheduling skipped)

**What happens:** A loop with four good moves but no automation is not a loop; it's
a script the human runs by hand and then forgets to run again. It works impressively
the day it's built and silently stops the day attention wanders.

**Diagnostic signal:** The last run timestamp is the day the loop was demoed or the
day the human last remembered to run it. No runs during nights or weekends.

**Fix:** A real trigger that does not depend on the human remembering. Either a
`schedule:` in GitHub Actions (cloud, machine-off safe) or a `/loop` command in
Claude Code (local, requires machine on). The trigger invokes a named skill, not
an inline prompt.

---

## The Blind Loop (Discovery skipped)

**What happens:** The human still hands the loop its work each morning — "fix these
three bugs" — so the loop has automated the doing but not the finding. This saves
less than it appears to, because choosing what to work on is often the expensive part.

**Diagnostic signal:** A human who is still spending their morning deciding what the
loop should work on. The loop is an executor, not a discoverer.

**Secondary signal:** The loop's trigger is a manual message or Slack command with a
specific task attached, rather than a schedule that reads live sources.

**Fix:** A discovery skill that reads CI failures, open issues, recent commits, and
the previous state file, and decides what is worth a worktree. The discovery logic
must live in a SKILL.md, not in the schedule command itself.

---

## The Tangled Loop (Handoff skipped)

**What happens:** The loop runs several agents in parallel but lets them all change
the same working directory. Their edits collide and the merge is a mess no one can
untangle cleanly.

**Diagnostic signal:** The problem only appears under parallelism. A single-agent
loop looks fine. The failure shows up the first morning multiple agents run concurrently
and produce conflicting changes to shared files.

**Fix:** One isolated git worktree per task. `claude --worktree fix/<slug>` or
Codex equivalent. Each agent gets its own checkout. One agent's edits cannot touch
another's working directory.

---

## How Failures Cluster

The five are not independent. In practice:

**The disciplined loop** installs all five moves. Discovery finds real work. Handoff
isolates it. Verification rejects bad output. Persistence remembers progress. Scheduling
makes it run without a human.

**The hasty loop** installs only Discovery and Handoff — the two that produce visible
output — and skips the three that produce safety. It looks productive for the first
few days, then accumulates debt silently.

**The compounding failure:** Unverified output (Nodding) erodes the developer's
understanding of the codebase (comprehension rot). Comprehension rot invites the
developer to stop reviewing outputs (cognitive surrender). Cognitive surrender lets
the loop run longer unwatched, spending more tokens on worse decisions (token blowout).
The four silent costs of loop engineering reinforce each other and come due all at once.

**The guard against all four:** Keep a human capable of saying "no", and install a
check the human doesn't have to be awake to run.
