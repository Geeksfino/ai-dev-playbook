---
name: loop-engineering
description: >
  Use this skill whenever a developer wants to set up, design, audit, or debug an automated
  agent loop for their project. Triggers include: "help me set up a loop", "I want agents to
  run while I sleep", "automate my CI triage", "set up a goal loop", "review my loop setup",
  "design an agentic workflow", "loop engineering", "build a self-running agent", or any
  request to make Claude Code or Codex run autonomously on a schedule or goal. Also trigger
  when a developer describes wanting agents to find work, fix bugs, open PRs, or run
  verification without manual prompting. This skill copies canonical templates from
  templates/ (bundled with this skill), customizes them from a 7-question interview, and
  writes the files to the correct paths in the developer's project. Do not paraphrase or
  shorten template content — copy first, then customize only CUSTOMIZE markers and
  interview-driven fields.
---

# Loop Engineering Skill

Loop engineering replaces manual prompting with designed systems that prompt agents
automatically. This skill scaffolds those systems for a specific project.

**Canonical templates:** All loop artifacts live in `templates/` next to this SKILL.md.
Path A (this skill) and Path B (README direct copy) use the **same files**. BUILD mode
copies from `templates/`, applies interview answers at `CUSTOMIZE` points, and writes
to the project. Never invent structure from memory — read the template files first.

**Two modes:**
- **BUILD** — developer wants to create a loop from scratch → Interview, then copy templates
- **AUDIT** — developer has an existing loop and wants it reviewed → Audit Checklist

Determine which mode applies from context, or ask if unclear.

---

## Mode 1: BUILD — Scaffolding a First Loop

### Step 1: Interview the Project (ask these, don't skip)

Before producing any files, collect the following. Ask all at once to save turns.

```
1. What triggers the work? (CI failures / open issues / commits / manual Slack trigger / all of the above)
2. What does "done" look like for a task? (tests pass / lint clean / PR approved / custom condition)
3. Which toolchain? (Cursor / Claude Code / Codex / other)
4. Where should the loop run? (local machine / GitHub Actions / both)
5. Should agents open PRs automatically, or land everything in an inbox for human review?
6. Do you already have a CLAUDE.md, AGENTS.md, or .cursor/rules? (affects harness + skill paths)
7. What's the budget ceiling per run? (dollar amount or token count — required before shipping)
```

If the developer can't answer #2 (the stop condition), stop and resolve it before
proceeding. A loop without a verifiable stop condition is the Nodding Loop anti-pattern.

### Step 2: Resolve destination paths

From interview answer #3, set these paths in the **target project** (not this skill dir):

| Toolchain | loop-triage skill | loop-engineering (already installed) | evaluator agent |
|-----------|-------------------|--------------------------------------|-----------------|
| **Cursor** | `.cursor/skills/loop-triage/SKILL.md` | `.cursor/skills/loop-engineering/` | project agent config or `.cursor/agents/loop-reviewer.md` if supported |
| **Claude Code** | `.claude/skills/loop-triage/SKILL.md` | `.claude/skills/loop-engineering/` | `.claude/agents/loop-reviewer.md` |
| **Codex** | `.codex/skills/loop-triage/SKILL.md` | `.codex/skills/loop-engineering/` | `.codex/agents/loop-reviewer.md` |

Shared paths (all toolchains):

| Artifact | Destination |
|----------|-------------|
| State file | `state/triage.md` |
| Inbox | `inbox/.gitkeep` |
| Schedule | `.github/workflows/loop-triage.yml` (if answer #4 includes GitHub Actions) |
| Checklist | `loop-checklist.md` (project root) |

Tell the developer the resolved paths before writing files.

### Step 3: Copy templates and customize

Read each file from `templates/` (relative to this skill). Copy content to the
destination paths above. **Do not paraphrase, summarize, or omit sections** outside
`CUSTOMIZE` markers. If a bundled template is missing, fetch the matching file from
`https://raw.githubusercontent.com/Geeksfino/ai-dev-playbook/main/templates/`.

Apply interview answers only where indicated:

| Template | Source file | Customize using |
|----------|-------------|-----------------|
| Discovery skill | `templates/loop-triage/loop-triage.md` | Q1 (triggers → Read), Q2 (stop condition → Hand off examples), Q5 (inbox vs PR → Stop), project labels at `<!-- CUSTOMIZE -->` |
| State file | `templates/state/triage.md` | Usually none — commit as-is |
| Evaluator | `templates/evaluator-agent/loop-reviewer.md` | **No edits** — copy verbatim |
| Schedule | `templates/github-actions/loop-triage.yml` | Q4 (cron if Actions), Q7 (`--max-tokens`), Q5 (PR vs inbox in run prompt) |
| Inbox | `templates/inbox/.gitkeep` | Copy as-is |
| Checklist | `templates/loop-checklist.md` | Copy as-is; developer checks boxes manually |

**Strip TEMPLATE header comments** (lines starting with `# TEMPLATE`) from files written
into the target project — those headers are for humans copying from the playbook repo,
not for deployed loop files.

**Scheduling (#4):** If GitHub Actions only → write workflow file. If local only → skip
workflow; document the `/loop` command from `references/toolchain-map.md`. If both → write
workflow and document local command.

**Evaluator:** Never weaken `loop-reviewer.md`. A separate skeptical evaluator prevents
the Nodding Loop anti-pattern.

### Step 4: Verify before finishing

Confirm for the developer:

1. All five loop moves are covered (see `references/five-moves.md`)
2. Generated loop-triage and workflow match bundled templates except CUSTOMIZE substitutions
3. `loop-reviewer.md` is byte-identical to `templates/evaluator-agent/loop-reviewer.md` (minus TEMPLATE header)
4. `state/triage.md` and `inbox/` exist before any schedule is enabled
5. Budget ceiling from Q7 is reflected in workflow `--max-tokens` or documented for local runs
6. `loop-checklist.md` is in the project root — walk through it together if this is the first loop

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

For each **absent** or **partial** item, copy the relevant file from `templates/`,
customize from what you know about the project, and write to the correct destination
(same rules as BUILD Step 3).

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

- `templates/` — Canonical loop artifacts (same as playbook `templates/` at repo root)
- `references/five-moves.md` — Theory: discovery, handoff, verification, persistence, scheduling
- `references/failure-modes.md` — The five anti-patterns and how to diagnose them
- `references/toolchain-map.md` — Claude Code vs Codex vs Cursor command equivalents

Read reference files only when you need underlying reasoning. Always read `templates/`
when producing or repairing artifacts.
