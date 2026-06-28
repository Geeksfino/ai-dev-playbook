# Toolchain Map: Claude Code vs Codex

Loop engineering is a set of capabilities, not a product.
The same five moves are available in both toolchains under different names.

| Capability | Claude Code | Codex |
|---|---|---|
| **Scheduling** | `/loop <interval> <task>` | Automations tab |
| **Run until condition met** | `/goal <condition>` | Automation + rerun + judge agent |
| **Parallel isolation** | `--worktree` / `-w` flag | Background worktree |
| **Sub-agents** | `.claude/agents/<name>.md` | `.codex/agents/<name>` |
| **External connectors** | MCP + plugins | MCP connector |
| **Named skill** | `SKILL.md` in `.claude/skills/` | `$skill-name` |
| **Machine-off scheduling** | Cloud Routines | Cloud (planned) |

---

## Scheduling Details

### Claude Code `/loop`

```bash
/loop 5m check the deploy        # fixed: every 5 minutes
/loop check the deploy            # agent paces itself
/loop                             # runs .claude/loop.md if it exists
```

- Session-scoped: recurring tasks expire after 7 days
- Requires machine on and session open
- Minimum interval: 1 minute
- Can see local files and local dev servers

### Claude Code Cloud Routines

- Runs without machine on
- Minimum interval: 1 hour (runs on a fresh clone)
- Cannot see local files or local dev servers
- Set up via Claude Code settings

### Codex Automations

- Configured in the Automations tab of the Codex UI
- Can be triggered by schedule or by event
- Cloud-native: does not require local machine

### GitHub Actions (toolchain-agnostic)

```yaml
on:
  schedule:
    - cron: '0 6 * * 1-5'
```

- Works with either Claude Code or Codex via CLI invocation
- Runs on GitHub infrastructure (no local machine required)
- Full access to repo, secrets, and CI context
- Free tier: 2,000 minutes/month for public repos

---

## Skill Invocation

### Claude Code
Skills live in `.claude/skills/<skill-name>/SKILL.md`.
Invoke by name: `--skill <skill-name>` or reference in prompt.

### Codex
Skills use `$skill-name` syntax in prompts.
Skill files follow the same SKILL.md format.

### Cross-toolchain rule
A connector or skill written for one toolchain can often be dropped into the other
and used as-is. Write the logic once; add the toolchain-specific invocation wrapper.

---

## Sub-agent / Evaluator Setup

### Claude Code evaluator agent

File: `.claude/agents/loop-reviewer.md`

The agent is invoked automatically when its description matches the task context,
or explicitly: `claude --agent loop-reviewer "review this change"`.

### Codex evaluator agent

File: `.codex/agents/<name>`

Same pattern, different directory. Configuration syntax follows Codex agent spec.

---

## Stop Condition Implementation

### Claude Code `/goal`

```bash
/goal all tests in test/auth pass and the lint step is clean
```

After each turn, a small fast model checks whether the condition holds.
If not, another turn runs. Available after Claude Code v2.1.139.

### Codex

Equivalent: automation + rerun + judge agent configuration.
The judge agent checks the condition after each agent turn.

**Note:** Do not confuse `/goal` (runs until condition met) with `/loop`
(reruns on an interval regardless of outcome). They compose: a `/loop` can
invoke tasks that internally use `/goal`.

---

## Which Toolchain to Choose

The choice rarely matters for the loop's logic. It matters for:

1. **Local file access** — need to see a local dev server? → Claude Code local
2. **Machine-off operation** — need to run overnight? → Cloud Routines or GitHub Actions
3. **Team familiarity** — using Codex already? → stay in Codex
4. **Existing CI** — have GitHub Actions? → use the schedule trigger there

A mature loop often uses both: local `/loop` for tight inner checks during the day,
cloud scheduling for the overnight sweep.
