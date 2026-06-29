# TEMPLATE — copy this file to your loop-triage skill path as SKILL.md:
#   Cursor:      .cursor/skills/loop-triage/SKILL.md
#   Claude Code: .claude/skills/loop-triage/SKILL.md
# Edit all sections marked CUSTOMIZE before committing.
# See README.md → Templates for the full install sequence.

---
name: loop-triage
trigger: invoked by daily automation or on-demand
---

# Loop Triage Skill

Discovers work worth acting on and writes it to ./state/triage.md.
Edit the sections marked CUSTOMIZE before deploying.

## Read (Discovery inputs)

- CI runs that failed since the last run
- Issues opened in the last 24h labeled `bug` or `needs-fix`  <!-- CUSTOMIZE: match your label names -->
- Commits merged since the last run
- The previous ./state/triage.md (carry forward unresolved findings)

## Judge

For each candidate finding, decide:
- Is it actionable now, or noise? Skip noise.
- Does it block a release? → mark priority:high
- Is it already tracked in triage.md with status open or fixing? → skip, no duplicates
- <!-- CUSTOMIZE: add your project's triage rules here -->
  <!-- Example: "Does it touch the payments module? → inbox, always" -->

Keep only findings worth opening a worktree for today.
Aim for 3–5 findings per run. More than 10 is a signal the filter is too loose.

## Write (Persistence output)

Append findings to ./state/triage.md using this format:

| finding | source | priority | status | worktree |
|---------|--------|----------|--------|----------|

Commit the file before exiting so the next run can read it.
Use commit message: `chore: loop triage YYYY-MM-DD`

## Hand off

For each kept finding, emit a task line:
```
worktree=fix/<slug>  goal=<stop-condition>  description=<one-line summary>
```

The stop condition must be verifiable. Examples:
- `tests in test/auth pass and lint is clean`
- `the null pointer in issue #92 no longer reproduces`
- `migration runs without error on a fresh database`

If you cannot write a verifiable stop condition for a finding, send it to ./inbox/ instead.

## Stop (the boundary you keep for yourself)

<!-- These are non-negotiable. Do not remove them. -->
- Never merge to main. Never push directly to main.
- Never delete files unless the finding explicitly asks for deletion.
- Anything touching auth, payments, database migrations, or secrets → inbox, always.
- Any finding where confidence is less than high → inbox.
- If the stop condition cannot be verified by running tests or a linter → inbox.
