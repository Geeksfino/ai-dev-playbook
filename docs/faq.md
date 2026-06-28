# FAQ

**Did Andrej Karpathy write CLAUDE.md?**

No — not directly. Karpathy posted observations about LLM coding pitfalls on X in
late 2025. Developer Forrest Chang turned those observations into a CLAUDE.md file
in the `andrej-karpathy-skills` repo. The wording is Chang's; the diagnosis is
Karpathy's. The version here is a further trimmed edit of Chang's work.
See `docs/why-these-exist.md` for the full provenance chain.

---

**Do I need both CLAUDE.md and the loop skill?**

No. They operate at different layers and each works independently.
`CLAUDE.md` governs single-agent behavior in every session.
The loop skill is only relevant when you want agents running on a schedule without
manual prompting. Most projects benefit from CLAUDE.md immediately; the loop skill
becomes relevant when manual prompting is the bottleneck.

---

**Does CLAUDE.md work with Cursor / Windsurf / other editors?**

The behavioral rules in CLAUDE.md work with any agent that accepts a system-level
instruction file. For Cursor, copy the contents into `.cursor/rules/playbook.mdc`.
For other editors, check their documentation for the equivalent configuration file.
The loop skill is Claude Code-specific.

---

**Will this slow down simple tasks?**

CLAUDE.md's §2 (Think Before You Code) adds a clarification step before
implementation. For a one-line fix this adds a small amount of friction.
For a multi-file architectural change it saves hours. If the friction bothers
you on simple tasks, you can tell Claude to skip clarification: "just do it"
overrides the rule in practice.

---

**Can I add project-specific rules to CLAUDE.md?**

Yes. Add them after the last section. The existing rules are generic and apply
to any project; your additions should be project-specific (naming conventions,
forbidden patterns, required test locations, etc.).

---

**The loop keeps nodding at itself — evaluator always passes. Why?**

The Nodding Loop is the most common failure mode. Causes:

1. The evaluator agent doesn't have separate instructions — it's the same agent
   re-reading its own output. Fix: use the template in
   `templates/evaluator-agent/loop-reviewer.md` as a separate `.claude/agents/`
   file.

2. The evaluator reads code instead of executing it. Fix: the evaluator template
   explicitly requires running tests and pasting actual output.

3. The stop condition is checked by the same model that did the work. Fix: use
   Claude Code's `/goal` command, which uses a fresh model to check the condition.

If your evaluator has never produced a REJECT in more than 50 turns on a real
codebase, it is not functioning.

---

**How do I stop the loop if something goes wrong?**

- **GitHub Actions:** Cancel the workflow run from the Actions tab. The workflow
  has a 30-minute `timeout-minutes` ceiling as a safety net.
- **Claude Code `/loop`:** Type `/stop` or close the session.
- **Worktrees:** Each finding runs in an isolated worktree. Even if one agent
  goes wrong, it cannot affect other worktrees or the main branch (PRs are never
  auto-merged).

The `Stop` section in the triage skill template encodes what the loop must never
do. Read it before deploying and make sure it matches your project's risk tolerance.

---

**Should I commit CLAUDE.md to the repo?**

Yes, if you want the rules to apply for everyone on the team using Claude Code.
The file is project-level configuration, not personal preference.

If you have personal rules that only apply to you, put them in
`~/.claude/CLAUDE.md` (user-level) rather than the project root.

---

**Will these files become outdated as models improve?**

CLAUDE.md will need less over time as models internalize these behaviors more
reliably. The loop-engineering skill will evolve as Claude Code's primitives
change (`/goal`, `/loop`, worktrees). Watch the repo for updates.

The underlying principles — separate generator from evaluator, persist state
outside the context window, don't skip verification — are unlikely to become
wrong. The implementation details (specific commands, file locations) will
change with tooling updates.
