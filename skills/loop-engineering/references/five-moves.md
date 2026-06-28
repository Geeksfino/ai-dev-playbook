# The Five Moves of One Loop Turn

From: Loop Engineering — The Anthropic Playbook (IEEE working note, 2026)

A loop is not idle spinning. Each turn does something concrete across five moves.
Drop any one and the loop fails to run, or runs in place.

```
Discovery → Handoff → Verification → Persistence → Scheduling
                           ↑
                      the "say no"
```

---

## Discovery

**What it does:** Figures out what this turn should do, without being told by a human.

The agent reads live sources (CI results, issue tracker, recent commits, previous
state file) and surfaces work worth acting on. The key is *letting the agent find
its own work* rather than being handed a list each morning.

**Where quality is decided:** Discovery sets the ceiling on the whole loop's quality.
Surface work of no value and the other four moves execute beautifully in service of nothing.

**Implementation:** Always put discovery logic in a named skill (SKILL.md), not inlined
in a cron command. A skill can be maintained; a wall of instructions in a schedule rots.

---

## Handoff

**What it does:** Moves each found task into the hands of an isolated agent.

Each finding gets its own git worktree — a separate working directory in the same
repo. Two agents writing the same file simultaneously is the same headache as two
engineers committing to the same lines.

**Without worktrees:** Parallelism looks fine with one agent. The tangled-loop failure
appears the first morning five agents run at once and their edits collide.

**Implementation:** `claude --worktree fix/<slug> "<task>"` or equivalent in Codex.
One worktree per finding, named after the finding slug.

---

## Verification

**What it does:** A separate agent judges whether the result is actually right.

This is the move most often skipped and least affordable to skip.

**Why a separate agent is required:** An agent grading its own output tends to pass it.
The context in which code was written is full of the reasons it was written that way.
The generator cannot step outside its own perspective. A dedicated evaluator with
*different instructions* — carrying none of the generator's self-persuasion — catches
what the generator rationalized away.

**The evaluator should act, not just read:** Reading code judges "does this look right."
Executing code judges "does this run right." Hook the evaluator to test runners,
linters, and where appropriate, browser automation (Playwright MCP for frontend tasks).

**Calibration:** Tell the evaluator to *assume the code is broken until proven otherwise*.
The default stance should be doubt, not trust. Using a different underlying model than
the generator also helps — the same model with new instructions often keeps its blind spots.

**The stop condition:** After each turn, a *fresh small model* (not the generator, not
the evaluator) checks whether the success condition holds. If not, another turn runs.
Completion is decided by something that did not do the work. This is the maker–checker
principle, borrowed from banking, applied to agent loops.

**Implementation:** Claude Code `/goal` command. Codex: automation + rerun + judge agent.

---

## Persistence

**What it does:** Lands the result somewhere that survives the conversation.

A loop's memory cannot live in the context window. When the window clears, the agent
forgets everything. For a loop to pick up today where it left off yesterday, findings
and progress must be written to disk.

**Memory vs context:**
- *Context* is what the agent sees this round. It is flushed on refresh.
- *Memory* is what is written to the repo. It persists across rounds and days.

The agent forgets. The repo does not.

**What to persist:**
- `./state/triage.md` — the running list of findings and their status
- `./inbox/` — findings the agent isn't confident enough to act on (human review queue)
- PRs opened via connector (gh CLI or MCP) — the permanent record of work done

**Implementation:** The triage skill writes the state file and commits it before exiting.
The next run reads it first. Connectors (MCP integrations) open PRs and update tickets.

---

## Scheduling

**What it does:** Makes one turn into a loop by triggering it again automatically.

Without scheduling, what you have is a script you ran once and forgot.

**Scheduling options:**

| Option | Requires machine on | Min interval | Sees local files |
|--------|--------------------:|:------------:|:----------------:|
| GitHub Actions `schedule:` | No | 1 hour | No (fresh clone) |
| Claude Code `/loop` | Yes | 1 min | Yes |
| Desktop task scheduler | Yes | 1 min | Yes |

**Key rule:** If the loop's work can run from a fresh git clone → GitHub Actions.
If the loop needs to see a local dev server or local files → run locally.
These are different capabilities. Conflating them leads to disappointment when the
laptop lid closes and the "autonomous" loop quietly stops.

**Implementation:** The automation should invoke a named skill, not a wall of
instructions pasted into the schedule. The skill is maintainable; the inline prompt rots.

---

## How the Moves Compound

The five moves are not independent. A loop missing verification tends also to miss
persistence, because a team careless about one check is usually careless about the others.

In practice they cluster:
- **Disciplined loop:** installs all five moves
- **Hasty loop:** installs only Discovery and Handoff — the two that produce visible
  output — and skips Verification, Persistence, and Scheduling — the three that produce safety

The hasty loop impresses on demo day. The disciplined loop runs while you sleep.
