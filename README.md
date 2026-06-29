> **English** · [中文](docs/zh/README.md)

# AI Dev Playbook

Two complementary layers for AI-assisted software development: a **harness** that governs how a single agent run behaves, and a **loop** skill that scaffolds automated agent systems when you need them.

They are independent — use one, both, or neither — but designed to stack cleanly.

---

## What's in here

| Artifact | Layer | When it's active | What it fixes |
|----------|-------|-----------------|---------------|
| `CLAUDE.md` | Harness | Every session, always | Silent assumptions, over-engineering, collateral edits |
| `skills/loop-engineering/` | Loop | On demand | No structure for building automated agent loops |
| `templates/` | Loop (shortcuts) | Copy-paste | Skip the interview, get loop files pre-filled |

---

## The two layers

**`CLAUDE.md` operates at the harness layer.** It governs how a single agent run behaves — don't over-engineer, make surgical changes, think before coding, state your assumptions. It's always in context, costs ~670 tokens per session, and applies to every task the agent does.

**The loop-engineering skill operates at the loop layer.** It's invoked only when a developer explicitly wants to scaffold or audit an automated loop. It sits dormant until triggered, then costs ~2,800 tokens for that session.

They don't conflict. The `CLAUDE.md` rules actually *strengthen* the loop the skill produces:

- **§4 Surgical Changes** directly prevents the evaluator agent from touching code outside its assigned worktree
- **§6 Goal-Driven Execution** is exactly what the skill's stop-condition interview enforces — turn "fix bugs" into a verifiable condition before starting
- **§2 Think Before You Code** maps to the skill's 7-question interview gate — both say "state your assumptions before generating anything"

---

## The three-layer stack

When you deploy both, the hierarchy looks like this:

```
CLAUDE.md (or equivalent harness rule)   ← always active, single-agent behavior
  └── loop-engineering/SKILL.md          ← activated on demand, scaffolds loops
        └── loop-triage/SKILL.md         ← generated artifact, runs inside the loop
```

The **triage `SKILL.md`** that loop-engineering generates as Artifact 1 is itself a skill — a discovery skill specific to your project. Each layer is independent but designed to stack on the one below.

---

## Which tool?

This playbook is **not** tied to Claude Code or Codex. The behavioral rules and loop patterns work with any agent that accepts persistent instructions and skills.

| Tool | Harness layer | Loop setup skill (`loop-engineering`) | Loop runtime (generated) |
|------|--------------|--------------------------------------|--------------------------|
| **Cursor** | `.cursor/rules/playbook.mdc` | `~/.cursor/skills/` (personal) or `.cursor/skills/` (project) | `.cursor/skills/loop-triage/`, `state/`, etc. in **each project** |
| **Claude Code** | `CLAUDE.md` in project root | `~/.claude/skills/` (personal) or `.claude/skills/` (project) | `.claude/skills/loop-triage/`, `state/`, etc. in **each project** |
| **Codex** | `AGENTS.md` or project instructions | Codex skill directory | Per-project loop files |
| **Other agents** | System-instruction file | User or project skill directory | Per-project loop files |

Set `REPO=Geeksfino/ai-dev-playbook` (or your fork) in the commands below.

---

## Personal vs project install

Not everything belongs in the same place. The playbook splits **setup tooling** from **running loop files**.

| Artifact | Personal (`~/.cursor/skills/`) | Project (`.cursor/skills/` in repo) |
|----------|--------------------------------|-------------------------------------|
| **`loop-engineering`** (scaffolding + audit) | **Recommended** for solo devs working across many repos — install once, use everywhere | **Recommended** for teams — version-pinned, everyone gets it on clone |
| **`loop-triage`** (discovery) | Never — rules are repo-specific | **Always** — commit in each project that runs a loop |
| **`state/`, `inbox/`, workflow, evaluator, checklist** | Never | **Always** — committed per project |

**Rule of thumb:** install `loop-engineering` where you install your *tools*; install loop *outputs* where your *code* lives.

Do not use `~/.cursor/skills-cursor/` — that directory is reserved for Cursor's built-in skills.

---

## Which path do you need?

```
New to AI coding agents?
└── Start with the harness layer only — nothing else needed yet

Spending your morning manually triaging CI failures and open issues?
└── Install the loop-engineering skill
   Your agent guides you through a 7-question interview and generates
   customised versions of the loop files for your project

Already know your project's triage rules and stop conditions?
└── Copy the templates directly and edit the CUSTOMIZE markers yourself
   Faster than the interview; no interactive setup required
```

Both the skill path and the template path produce the same files in the same
locations inside your project. They use the **same canonical templates** — the
skill path runs the interview and writes customized copies; the template path
lets you copy and edit `CUSTOMIZE` markers yourself.

---

## Install

### Harness layer — always-on agent behavior

**Cursor** — create a project rule:

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/playbook.mdc \
  https://raw.githubusercontent.com/$REPO/main/CLAUDE.md
```

Add this frontmatter at the top of `playbook.mdc` if it isn't there already:

```yaml
---
description: AI dev playbook — harness-layer behavioral rules
alwaysApply: true
---
```

**Claude Code** — drop into project root (read automatically at session start):

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/$REPO/main/CLAUDE.md
```

If you already have a `CLAUDE.md`, append instead:

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/$REPO/main/CLAUDE.md >> CLAUDE.md
```

**Codex / other tools** — copy the contents of `CLAUDE.md` into your tool's equivalent project-instruction file (`AGENTS.md`, workspace rules, etc.).

---

### Loop layer — guided setup (skill path)

Install `loop-engineering` once (personal or project — see above). Your agent
discovers it automatically when relevant. The skill writes loop **runtime** files
into whichever project you have open.

**Cursor** — set `SKILL_HOME` to personal or project path, then run:

```bash
BASE=https://raw.githubusercontent.com/$REPO/main

# Personal (recommended solo): available in all Cursor projects
SKILL_HOME=~/.cursor/skills/loop-engineering

# Project (recommended teams): commit .cursor/skills/loop-engineering/ to the repo
# SKILL_HOME=.cursor/skills/loop-engineering

mkdir -p $SKILL_HOME/references
mkdir -p $SKILL_HOME/templates/{loop-triage,evaluator-agent,state,github-actions,inbox}

curl -o $SKILL_HOME/SKILL.md $BASE/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o $SKILL_HOME/references/$f.md \
    $BASE/skills/loop-engineering/references/$f.md
done

curl -o $SKILL_HOME/templates/loop-triage/loop-triage.md \
  $BASE/skills/loop-engineering/templates/loop-triage/loop-triage.md
curl -o $SKILL_HOME/templates/evaluator-agent/loop-reviewer.md \
  $BASE/skills/loop-engineering/templates/evaluator-agent/loop-reviewer.md
curl -o $SKILL_HOME/templates/state/triage.md \
  $BASE/skills/loop-engineering/templates/state/triage.md
curl -o $SKILL_HOME/templates/github-actions/loop-triage.yml \
  $BASE/skills/loop-engineering/templates/github-actions/loop-triage.yml
curl -o $SKILL_HOME/templates/inbox/.gitkeep \
  $BASE/skills/loop-engineering/templates/inbox/.gitkeep
curl -o $SKILL_HOME/templates/loop-checklist.md \
  $BASE/skills/loop-engineering/templates/loop-checklist.md
```

**Claude Code** — same pattern with `SKILL_HOME`:

```bash
BASE=https://raw.githubusercontent.com/$REPO/main

# Personal: SKILL_HOME=~/.claude/skills/loop-engineering
SKILL_HOME=~/.claude/skills/loop-engineering

# Project: SKILL_HOME=.claude/skills/loop-engineering

mkdir -p $SKILL_HOME/references
mkdir -p $SKILL_HOME/templates/{loop-triage,evaluator-agent,state,github-actions,inbox}

curl -o $SKILL_HOME/SKILL.md $BASE/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o $SKILL_HOME/references/$f.md \
    $BASE/skills/loop-engineering/references/$f.md
done

curl -o $SKILL_HOME/templates/loop-triage/loop-triage.md \
  $BASE/skills/loop-engineering/templates/loop-triage/loop-triage.md
curl -o $SKILL_HOME/templates/evaluator-agent/loop-reviewer.md \
  $BASE/skills/loop-engineering/templates/evaluator-agent/loop-reviewer.md
curl -o $SKILL_HOME/templates/state/triage.md \
  $BASE/skills/loop-engineering/templates/state/triage.md
curl -o $SKILL_HOME/templates/github-actions/loop-triage.yml \
  $BASE/skills/loop-engineering/templates/github-actions/loop-triage.yml
curl -o $SKILL_HOME/templates/inbox/.gitkeep \
  $BASE/skills/loop-engineering/templates/inbox/.gitkeep
curl -o $SKILL_HOME/templates/loop-checklist.md \
  $BASE/skills/loop-engineering/templates/loop-checklist.md
```

**Codex** — same `SKILL.md` and references; place under your Codex skill directory and invoke with `$loop-engineering`.

Then ask your agent: **"help me set up a loop for this project"**

The skill runs a 7-question interview (trigger sources, stop conditions, toolchain,
scheduling preference, budget ceiling, PR vs inbox behaviour, existing harness rules).
After the interview it **copies the bundled `templates/`** into your project,
customizes `CUSTOMIZE` markers from your answers, and writes each file to the
correct path. Output matches the direct template path (Path B).

---

### Templates — direct copy (template path)

Use this instead of the skill if you already know what you want. Each template file
has a `# TEMPLATE — copy this file to:` header with the destination path inside
your project.

**What goes where:**

| Template file | Destination(s) in your project | Purpose |
|---|---|---|
| `templates/loop-triage/loop-triage.md` | `.cursor/skills/loop-triage/SKILL.md` or `.claude/skills/loop-triage/SKILL.md` | Discovery skill — reads CI/issues, writes state file |
| `templates/evaluator-agent/loop-reviewer.md` | `.claude/agents/loop-reviewer.md` (or your tool's agent config) | Adversarial reviewer — says no to bad output |
| `templates/state/triage.md` | `state/triage.md` | Loop memory — persists findings across runs |
| `templates/inbox/.gitkeep` | `inbox/.gitkeep` | Human review queue — findings the agent won't act on alone |
| `templates/github-actions/loop-triage.yml` | `.github/workflows/loop-triage.yml` | Schedule — runs the loop on a cron without the machine on |
| `templates/loop-checklist.md` | `loop-checklist.md` | Pre-flight checklist — verify all five moves before unattended runs |

**Install order matters.** Set up in this sequence or the loop has nothing to read:

```bash
BASE=https://raw.githubusercontent.com/$REPO/main
SKILL_DIR=.cursor/skills/loop-triage   # or .claude/skills/loop-triage
AGENT_DIR=.claude/agents               # or .codex/agents/

# Step 1 — state file and inbox (loop memory; must exist before first run)
mkdir -p state inbox
curl -o state/triage.md $BASE/templates/state/triage.md
curl -o inbox/.gitkeep $BASE/templates/inbox/.gitkeep

# Step 2 — discovery skill (reads CI/issues; writes to state/triage.md)
mkdir -p $SKILL_DIR
curl -o $SKILL_DIR/SKILL.md $BASE/templates/loop-triage/loop-triage.md

# Step 3 — evaluator agent (invoked after each agent turn to verify output)
mkdir -p $AGENT_DIR
curl -o $AGENT_DIR/loop-reviewer.md $BASE/templates/evaluator-agent/loop-reviewer.md

# Step 4 — schedule (triggers the loop; requires steps 1–3 to already be in place)
mkdir -p .github/workflows
curl -o .github/workflows/loop-triage.yml $BASE/templates/github-actions/loop-triage.yml

# Step 5 — checklist (optional but recommended before first unattended run)
curl -o loop-checklist.md $BASE/templates/loop-checklist.md
```

**After copying, edit these before committing:**

- Your loop-triage `SKILL.md` — find all `<!-- CUSTOMIZE -->` markers and replace
  with your project's actual triage rules (label names, priority criteria, modules
  that always go to inbox)
- `.github/workflows/loop-triage.yml` — set the cron schedule to your timezone,
  adjust `--max-tokens`, add `ANTHROPIC_API_KEY` to your repo secrets
- `state/triage.md` — no edits needed; commit as-is so the loop has a file to read

The evaluator agent (`loop-reviewer.md`) has no CUSTOMIZE markers — its
instructions are intentionally opinionated and should not be weakened.

---

## What these files do NOT do

- They do not add dependencies to your project.
- They do not run any code on install.
- They do not phone home or require accounts.
- They do not replace your existing harness rules — they append cleanly.
- They do not provide API keys — that's on you.

---

## Token cost

| File | Loaded when | Approximate cost |
|------|-------------|-----------------|
| `CLAUDE.md` | Every session | ~670 tokens (fixed overhead) |
| `loop-engineering/SKILL.md` | When loop setup is requested | ~2,800 tokens |
| `references/*.md` | Only when skill needs them | ~950–1,400 tokens each |
| `templates/*` (bundled with skill) | Only during loop BUILD/AUDIT | ~3,500 tokens if all loaded |
| Templates at repo root | Never loaded by the model | 0 tokens (copy-paste only) |

See [docs/token-costs.md](docs/token-costs.md) for the full breakdown.

---

## Documentation

| English | 中文 |
|---------|------|
| [Why these exist](docs/why-these-exist.md) | [为什么需要这些文件](docs/zh/why-these-exist.md) |
| [FAQ](docs/faq.md) | [常见问题](docs/zh/faq.md) |
| [Token costs](docs/token-costs.md) | [Token 成本](docs/zh/token-costs.md) |
| — | [完整中文说明](docs/zh/README.md) |

---

## Provenance

`CLAUDE.md` is derived from Andrej Karpathy's January 2026 observations about LLM coding pitfalls, distilled by Forrest Chang into the `andrej-karpathy-skills` repo, and trimmed here from 184 lines to 90 lines (~63% token reduction) while keeping all behavioral signal.

The loop-engineering skill is derived from the IEEE working note *Loop Engineering: The Anthropic Playbook* (HuaShu, June 2026), operationalizing its five-move framework and six-part anatomy into project-deployable artifacts.

See [docs/why-these-exist.md](docs/why-these-exist.md) for the full provenance and the specific failure each rule prevents.

---

## License

MIT. Copy, fork, modify, ship. Attribution appreciated but not required.
