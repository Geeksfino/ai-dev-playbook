# AI Dev Playbook

Three files that fix the most common failure modes in AI-assisted software development.

---

## What's in here

| File | When it's active | What it fixes |
|------|-----------------|---------------|
| `CLAUDE.md` | Every session, always | Silent assumptions, over-engineering, collateral edits |
| `skills/loop-engineering/` | On demand | No structure for building automated agent loops |
| `templates/` | Copy-paste start | Skip the interview, get loop files pre-filled |

These are **independent**. Take one, take all three, take none. Each works without the others.

---

## Install

### CLAUDE.md — drop into any project root

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/CLAUDE.md
```

That's it. Claude Code reads it automatically at session start.

If you already have a `CLAUDE.md`, append instead:

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/CLAUDE.md >> CLAUDE.md
```

---

### Loop Engineering Skill — install into a project

```bash
mkdir -p .claude/skills/loop-engineering/references

curl -o .claude/skills/loop-engineering/SKILL.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/skills/loop-engineering/SKILL.md

curl -o .claude/skills/loop-engineering/references/five-moves.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/skills/loop-engineering/references/five-moves.md

curl -o .claude/skills/loop-engineering/references/failure-modes.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/skills/loop-engineering/references/failure-modes.md

curl -o .claude/skills/loop-engineering/references/toolchain-map.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/skills/loop-engineering/references/toolchain-map.md
```

Then ask Claude Code: **"help me set up a loop for this project"** and the skill takes over.

---

### Templates — skip the interview, copy pre-filled files

If you know what you want and don't need the interactive setup:

```bash
# Discovery skill
mkdir -p .claude/skills/loop-triage
curl -o .claude/skills/loop-triage/SKILL.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/templates/loop-triage/SKILL.md

# Evaluator agent
mkdir -p .claude/agents
curl -o .claude/agents/loop-reviewer.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/templates/evaluator-agent/loop-reviewer.md

# State file and inbox
mkdir -p state inbox
curl -o state/triage.md \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/templates/state/triage.md
touch inbox/.gitkeep

# GitHub Actions schedule
mkdir -p .github/workflows
curl -o .github/workflows/loop-triage.yml \
  https://raw.githubusercontent.com/YOUR_ORG/ai-dev-playbook/main/templates/github-actions/loop-triage.yml
```

Edit each file to match your project before committing.

---

## Which files do I need?

```
Are you using Claude Code or Codex for the first time?
└── Yes → start with CLAUDE.md only

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
- They do not replace your existing `CLAUDE.md` — they append cleanly.
- They do not work without a `ANTHROPIC_API_KEY` in your environment (that's on you).

---

## The three-layer model

```
CLAUDE.md                        ← harness layer: governs single-agent behavior
  └── loop-engineering skill     ← loop layer: scaffolds automated agent systems
        └── loop-triage skill    ← discovery layer: generated per-project by the skill
```

Each layer is independent. Each layer assumes the one below is in place but does not require it.

---

## Token cost

| File | Loaded when | Approximate cost |
|------|-------------|-----------------|
| `CLAUDE.md` | Every session | ~670 tokens (fixed overhead) |
| `loop-engineering/SKILL.md` | When loop setup is requested | ~2,800 tokens |
| `references/*.md` | Only when skill needs them | ~950–1,400 tokens each |
| Templates | Never loaded by the model | 0 tokens |

See `docs/token-costs.md` for the full breakdown and what to trim if your project has a tight context budget.

---

## Provenance

`CLAUDE.md` is derived from Andrej Karpathy's January 2026 observations about LLM coding pitfalls, distilled by Forrest Chang into the `andrej-karpathy-skills` repo, and trimmed here from 184 lines to 90 lines (~63% token reduction) while keeping all behavioral signal.

The loop-engineering skill is derived from the IEEE working note *Loop Engineering: The Anthropic Playbook* (HuaShu, June 2026), operationalizing its five-move framework and six-part anatomy into project-deployable artifacts.

See `docs/why-these-exist.md` for the full provenance and the specific failure each rule prevents.

---

## License

MIT. Copy, fork, modify, ship. Attribution appreciated but not required.
