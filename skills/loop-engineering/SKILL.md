---
name: loop-engineering
description: >
  Use this skill whenever a developer wants to set up, design, audit, or debug an automated
  agent loop for their project. Triggers include: "help me set up a loop", "I want agents to
  run while I sleep", "automate my CI triage", "set up a goal loop", "review my loop setup",
  "design an agentic workflow", "loop engineering", "build a self-running agent", or any
  request to make Claude Code or Codex run autonomously on a schedule or goal. Also trigger
  when a developer describes wanting agents to find work, fix bugs, open PRs, or run
  verification without manual prompting. This skill produces concrete, project-specific
  artifacts: a SKILL.md for discovery, a state file template, a GitHub Actions schedule,
  evaluator agent config, and a first-loop checklist. Do not just explain the concept —
  scaffold the actual files.
---

# Loop Engineering Skill

Loop engineering replaces manual prompting with designed systems that prompt agents
automatically. This skill scaffolds those systems for a specific project.

**Two modes:**
- **BUILD** — developer wants to create a loop from scratch → run the Interview, then produce artifacts
- **AUDIT** — developer has an existing loop and wants it reviewed → run the Audit Checklist

Determine which mode applies from context, or ask if unclear.

---

## Mode 1: BUILD — Scaffolding a First Loop

### Step 1: Interview the Project (ask these, don't skip)

Before producing any files, collect the following. Ask all at once to save turns.

```
1. What triggers the work? (CI failures / open issues / commits / manual Slack trigger / all of the above)
2. What does "done" look like for a task? (tests pass / lint clean / PR approved / custom condition)
3. Which toolchain? (Claude Code / Codex / both)
4. Where should the loop run? (local machine / GitHub Actions / both)
5. Should agents open PRs automatically, or land everything in an inbox for human review?
6. Do you already have a CLAUDE.md or AGENTS.md? (affects where to put the skill)
7. What's the budget ceiling per run? (dollar amount or token count — required before shipping)
```

If the developer can't answer #2 (the stop condition), stop and resolve it before
proceeding. A loop without a verifiable stop condition is the Nodding Loop anti-pattern.

### Step 2: Produce the Five Artifacts

Once the interview is complete, produce all five in order. Each maps to one of the
five loop moves from the IEEE paper. See `references/five-moves.md` for the theoretical
grounding if needed.

#### Artifact 1 — Discovery Skill (maps to: Discovery move)

File: `.claude/skills/loop-triage/SKILL.md`

```markdown
---
name: loop-triage
trigger: invoked by daily automation or on-demand
---

## Read (Discovery inputs)
- CI runs that failed since the last run
- Issues opened in the last 24h
- Commits merged since the last run
- The previous ./state/triage.md (carry forward unresolved findings)

## Judge
For each candidate finding, decide:
- Is it actionable now, or noise? (skip noise)
- Does it block a release? → mark priority:high
- Is it already tracked in triage.md? → skip, don't duplicate

Keep only findings worth opening a worktree for today.

## Write (Persistence output)
Append to ./state/triage.md:
| finding | source | priority | status | worktree |
Commit the file so the next run can read it.

## Hand off
For each kept finding, emit a task line:
  worktree=fix/<slug>  goal=<stop-condition>  description=<one-line>

## Stop (the boundary you keep for yourself)
- Never merge. Never delete files. Never push to main directly.
- Anything with confidence < high goes to ./inbox/ for human review.
- If a finding touches auth, payments, or migrations: inbox, always.
```

> **Customize:** Replace the Judge section's priority rules with your project's
> actual triage criteria (e.g., label names from your issue tracker, CI job names).

---

#### Artifact 2 — State File Template (maps to: Persistence move)

File: `./state/triage.md`

```markdown
# Loop State — Triage

_Updated automatically by loop-triage skill. Do not edit manually._

| finding | source | priority | status | worktree | notes |
|---------|--------|----------|--------|----------|-------|
| (empty) | —      | —        | —      | —        | —     |
```

Commit this file to the repo. The agent reads it at the start of each run and writes
back to it at the end. This is the loop's memory — it must survive context window
clears. The agent forgets; the repo does not.

---

#### Artifact 3 — Evaluator Agent Config (maps to: Verification move)

File: `.claude/agents/loop-reviewer.md`

```markdown
---
name: loop-reviewer
description: Adversarial reviewer for loop-generated changes. Assumes broken until proven otherwise.
---

ROLE: You are an adversarial code reviewer. Your default stance is doubt, not trust.

ASSUME: The code you are reviewing is broken until you have proven otherwise through
execution, not reading.

DO NOT praise. Find what fails.

CHECK in order:
1. Does it run? Execute it. Don't just read it.
2. Run existing tests. Paste the actual output, not a summary.
3. Check edge cases the author skipped. Empty inputs. Null values. Auth failures.
4. Does the behavior match the original ticket or finding?
5. Did the change touch anything outside its stated scope? Flag it.

USE available tools: run tests, inspect files, check git diff.

VERDICT:
- PASS: every check above holds. State which checks passed.
- REJECT: list each reason. Be specific. "Test X failed with output Y" not "tests may fail".

A PASS from you is the stop condition. Do not pass code you have not executed.
```

> **Why a separate agent matters:** An agent grading its own output tends to pass it.
> The context in which code was written is full of reasons it was written that way,
> so the generator cannot see the result clearly. A fresh agent with different
> instructions — defaulting to doubt — catches what the generator rationalized away.

---

#### Artifact 4 — Schedule / Automation (maps to: Scheduling move)

**Option A: GitHub Actions (runs cloud, machine-off safe)**

File: `.github/workflows/loop-triage.yml`

```yaml
name: Loop Triage

on:
  schedule:
    - cron: '0 6 * * 1-5'   # 06:00 weekdays UTC — adjust to your timezone
  workflow_dispatch:          # allow manual trigger from GitHub UI

jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run triage skill
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          npx @anthropic-ai/claude-code \
            --skill loop-triage \
            --budget-tokens 50000 \
            --no-auto-merge \
            "Run the loop-triage skill. Write findings to ./state/triage.md.
             Open worktrees for actionable findings. Do not merge anything."

      - name: Commit state file
        run: |
          git config user.name "loop-agent"
          git config user.email "loop@noreply"
          git add state/triage.md
          git diff --staged --quiet || git commit -m "chore: loop triage $(date -u +%Y-%m-%d)"
          git push
```

**Option B: Claude Code local `/loop` (requires machine on)**

```bash
# In your project root:
/loop 0 6 * * 1-5 run the loop-triage skill
```

> **Choose based on one question:** Does the loop need to see local files or a local
> dev server? → local. Can it work from a fresh git clone? → GitHub Actions.
> Cloud scheduling means true autonomy (runs with lid closed); local means frequency
> and local file access at the cost of requiring the machine on.

---

#### Artifact 5 — First Loop Checklist (maps to: all five moves)

File: `./loop-checklist.md`

```markdown
# Loop Engineering Checklist

Before shipping the loop unattended, verify every item.

## Discovery
- [ ] The triage skill reads at least one live source (CI / issues / commits)
- [ ] The skill is in a SKILL.md file, not inlined in a cron command
- [ ] The skill skips noise (not every open issue is worth a worktree)

## Handoff
- [ ] Each finding gets its own isolated worktree (--worktree flag)
- [ ] Two agents never write to the same working directory simultaneously

## Verification
- [ ] A separate evaluator agent exists with instructions to default to doubt
- [ ] The evaluator executes code, not just reads it
- [ ] The stop condition is a fresh model checking the condition, not the generator

## Persistence
- [ ] ./state/triage.md is committed to the repo after each run
- [ ] ./inbox/ exists for findings the agent isn't confident about
- [ ] PRs are opened, never auto-merged without human review

## Scheduling
- [ ] A real trigger exists (cron / workflow_dispatch / /loop command)
- [ ] The trigger calls the skill by name, not a wall of inline instructions
- [ ] A budget ceiling is set (tokens or dollars) before first unattended run

## Safety
- [ ] The loop cannot merge to main without a human reviewing the PR
- [ ] Auth / payments / migrations are hard-coded to the inbox
- [ ] Someone knows how to stop the loop if it runs off (cancel workflow / kill /loop)
```

---

## Mode 2: AUDIT — Reviewing an Existing Loop

When a developer has a loop already running (or partially built), audit it against
the five failure modes from the IEEE paper. Read `references/failure-modes.md` for
the full diagnostic criteria.

Run through each anti-pattern and report: **present / absent / partial**.

| Anti-pattern | Symptom to check for |
|---|---|
| **Nodding Loop** | Has the loop ever produced a REJECT from its evaluator? If no REJECTs in >50 turns, the evaluator is broken. |
| **Amnesiac Loop** | Does each run start from the same place? Check if ./state/ exists and is committed. |
| **Manual Loop** | When was the last automated run? If the answer is "the day we built it", scheduling is broken. |
| **Blind Loop** | Is a human still deciding what the loop works on each morning? Discovery is missing. |
| **Tangled Loop** | Do parallel agents use separate worktrees? Check for shared working directory collisions. |

For each **absent** or **partial** item, produce the missing artifact from Mode 1.

---

## Token Budget Guidance

Set these before the first unattended run. Hard caps prevent a spinning bug from
consuming an entire night's quota.

| Loop type | Suggested per-run cap | Suggested daily cap |
|---|---|---|
| Single-agent triage | 50k tokens | 200k tokens |
| Multi-agent with worktrees | 100k tokens | 500k tokens |
| Enterprise / parallel fleet | Set by cost ceiling, not token count |

A loop without a budget cap has delegated its spending authority to its own bugs.

---

## Reference Files

- `references/five-moves.md` — Theory: discovery, handoff, verification, persistence, scheduling
- `references/failure-modes.md` — The five anti-patterns and how to diagnose them
- `references/toolchain-map.md` — Claude Code vs Codex command equivalents

Read these only when you need the underlying reasoning. The artifacts above are the
deliverables; the references explain why.
