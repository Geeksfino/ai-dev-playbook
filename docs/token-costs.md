# Token Costs

Honest numbers for what each file costs and when it's loaded.
Use this to make informed decisions about what to include in tight-context-budget projects.

---

## How Claude Code loads files

`CLAUDE.md` is read once at session start and included in the system prompt for the
entire session. It does not get re-sent with every message — it's a fixed overhead
per session, amortized across all turns.

Skills (SKILL.md files) are loaded when Claude determines the skill is relevant to
the current task. Reference files inside a skill are loaded only when the skill
body explicitly directs Claude to read them.

Templates are never loaded by the model — they're files for humans to copy.

---

## File costs

| File | Tokens | When loaded | Notes |
|------|-------:|-------------|-------|
| `CLAUDE.md` | ~670 | Every session | Fixed per-session overhead |
| `skills/loop-engineering/SKILL.md` | ~2,800 | When loop setup is requested | Not loaded in normal coding sessions |
| `references/five-moves.md` | ~1,400 | When building a loop (BUILD mode) | |
| `references/failure-modes.md` | ~1,300 | When auditing a loop (AUDIT mode) | |
| `references/toolchain-map.md` | ~950 | When configuring scheduling | |
| Templates | 0 | Never | Copy-paste only |

**Worst case** (all loop references loaded simultaneously): ~6,500 tokens.
This happens only when a developer is doing a full loop build or audit in a single session.

**Typical session with CLAUDE.md only:** ~670 tokens overhead.

**Typical loop setup session:** ~4,200 tokens (SKILL.md + one reference file).

---

## If you need to reduce costs

### CLAUDE.md is already trimmed

The version here is 90 lines / ~670 tokens, down from the 184-line / ~1,800 token
version it was derived from. Further trimming risks removing behavioral signal.
If you must go shorter, §7 (Debugging) and §8 (Dependencies) have the lowest
impact-per-token ratio and can be cut first.

### Loop engineering skill is already progressive

The skill body (SKILL.md) directs Claude to load reference files only when needed.
You don't need to do anything — the progressive loading is already designed in.

### If context pressure is severe

Consider putting the loop-engineering skill in a separate project where context
budget is not shared with large codebases. It's most useful in the planning phase
before implementation begins, not during active coding.

---

## What token cost actually means in practice

At current Claude Sonnet pricing, 6,500 tokens of input is approximately $0.02.
For most projects, the token cost of these files is not a meaningful expense.

The more relevant cost is **context window dilution**: in very long sessions with
many large files already loaded, adding ~670 tokens (CLAUDE.md) or ~2,800 tokens
(loop skill) competes for space with code context. If you notice Claude losing
track of earlier context in long sessions, consider:

1. Starting a fresh session for loop setup tasks
2. Removing reference files you've already acted on
3. Using `/compact` to compress conversation history before loading large files
