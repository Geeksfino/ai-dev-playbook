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

| Tool | Harness layer | Loop skill |
|------|--------------|------------|
| **Cursor** | `.cursor/rules/playbook.mdc` (`alwaysApply: true`) | `.cursor/skills/loop-engineering/` |
| **Claude Code** | `CLAUDE.md` in project root | `.claude/skills/loop-engineering/` |
| **Codex** | `AGENTS.md` or project instructions | `.codex/skills/` or `$skill-name` invocation |
| **Other agents** | Whatever your tool calls its system-instruction file | Copy `SKILL.md` to your tool's skill directory |

Set `REPO=Geeksfino/ai-dev-playbook` (or your fork) in the commands below.

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

### Loop layer — on-demand loop scaffolding

**Cursor:**

```bash
mkdir -p .cursor/skills/loop-engineering/references

curl -o .cursor/skills/loop-engineering/SKILL.md \
  https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o .cursor/skills/loop-engineering/references/$f.md \
    https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/references/$f.md
done
```

Then ask your agent: **"help me set up a loop for this project"** and the skill takes over.

**Claude Code:**

```bash
mkdir -p .claude/skills/loop-engineering/references

curl -o .claude/skills/loop-engineering/SKILL.md \
  https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o .claude/skills/loop-engineering/references/$f.md \
    https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/references/$f.md
done
```

**Codex** — same `SKILL.md` and references; place under your Codex skill directory and invoke with `$loop-engineering`.

---

### Templates — skip the interview

If you already know what you want and don't need the interactive setup:

```bash
# Discovery skill (Artifact 1 — adjust path for your tool)
mkdir -p .cursor/skills/loop-triage   # or .claude/skills/loop-triage
curl -o .cursor/skills/loop-triage/SKILL.md \
  https://raw.githubusercontent.com/$REPO/main/templates/loop-triage/SKILL.md

# Evaluator agent
mkdir -p .claude/agents               # or .codex/agents/
curl -o .claude/agents/loop-reviewer.md \
  https://raw.githubusercontent.com/$REPO/main/templates/evaluator-agent/loop-reviewer.md

# State file and inbox
mkdir -p state inbox
curl -o state/triage.md \
  https://raw.githubusercontent.com/$REPO/main/templates/state/triage.md
touch inbox/.gitkeep

# GitHub Actions schedule (toolchain-agnostic)
mkdir -p .github/workflows
curl -o .github/workflows/loop-triage.yml \
  https://raw.githubusercontent.com/$REPO/main/templates/github-actions/loop-triage.yml
```

Edit each file to match your project before committing.

---

## Which files do I need?

```
Are you using an AI coding agent for the first time?
└── Yes → start with the harness layer only (CLAUDE.md or .cursor/rules/)

Are you spending time each morning triaging CI / issues / PRs manually?
└── Yes → add the loop-engineering skill

Do you want agents running while you sleep without manual setup?
└── Yes → copy the templates and edit them for your project
```

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
| Templates | Never loaded by the model | 0 tokens |

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
