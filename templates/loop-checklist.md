# TEMPLATE — copy this file to: loop-checklist.md (project root)
# Work through every item before the first unattended loop run.
# See README.md → Templates for the full install sequence.

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
