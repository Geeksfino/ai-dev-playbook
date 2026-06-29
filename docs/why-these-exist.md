# Why These Files Exist

Every rule in this repo exists because a specific failure mode happens repeatedly
in AI-assisted software development. This document maps each file to the failure
it prevents and where the observation came from.

---

## CLAUDE.md

**Source:** Andrej Karpathy's January 26, 2026 post on X, describing a shift from
80% manual coding to 80% agent-driven coding and cataloguing the failure modes he
observed. Developer Forrest Chang distilled this into the `andrej-karpathy-skills`
GitHub repo. The version here is trimmed from Chang's 184-line expansion to 90 lines
(~63% fewer tokens) while preserving every behavioral rule.

**What Karpathy actually observed:**

> "The models make wrong assumptions on your behalf and just run along with them
> without checking. They don't manage their confusion, don't seek clarifications,
> don't surface inconsistencies, don't present tradeoffs, don't push back when
> they should."

> "They really like to overcomplicate code and APIs, bloat abstractions, don't clean
> up dead code... implement a bloated construction over 1000 lines when 100 would do."

> "They still sometimes change/remove comments and code they don't sufficiently
> understand as side effects, even if orthogonal to the task."

**Rule → Failure it prevents:**

| Rule | Failure prevented |
|------|------------------|
| §1 Read Before You Write | Agent pattern-matches from training instead of reading your codebase; produces code alien to the project |
| §2 Think Before You Code | Agent picks an interpretation silently ("I'm assuming JWT auth") and builds 200 lines on a wrong assumption |
| §3 Simplicity | Agent writes `EmailService` with strategy pattern when you needed `send_welcome_email()` |
| §4 Surgical Changes | Agent reformats the file, reorders imports, renames variables — producing a 200-line diff that hides the 3-line actual change |
| §5 Verification | Agent declares a fix done without running the test that would have caught it |
| §6 Goal-Driven Execution | Agent starts implementing immediately without stating what "done" looks like, produces code that passes a casual review but fails when it matters |
| §7 Debugging | Agent sees "TypeError" and generates a fix based on the error type without reading the stack trace |
| §8 Dependencies | Agent silently adds three packages to package.json to solve a problem the standard library already handles |
| §9 Failure Modes | Named anti-patterns (Kitchen Sink, Runaway Refactor, Knowledge Hallucination) give developers vocabulary to catch and name specific failure behaviors quickly |

---

## Loop Engineering Skill

**Source:** *Loop Engineering: The Anthropic Playbook* — an IEEE-style working note
(HuaShu, Orange Books, v260615, June 2026), itself synthesizing Addy Osmani's
"Loop Engineering" blog post (June 7, 2026), Boris Cherny's (Anthropic, Claude Code
lead) public remarks about writing loops instead of prompts, Peter Steinberger's
viral post on designing loops that prompt agents, and Prithvi Rajasekaran's
(Anthropic) findings on the generator/evaluator pattern.

**What the paper diagnosed:**

The paper's central observation is that the same tools that make agent loops powerful
make their failures silent and compounding. Unlike prompt-layer failures (one wrong
answer, caught immediately) or harness-layer failures (one bad run, visible in the
diff), loop-layer failures survive across many turns before anyone notices. The paper
names four silent costs — verification debt, comprehension rot, cognitive surrender,
token blowout — and shows they reinforce each other.

**The five-move framework → what each move prevents:**

| Move | What happens when it's skipped |
|------|-------------------------------|
| Discovery | Human still decides what the loop works on each morning — the expensive part isn't automated |
| Handoff | Parallel agents write to the same working directory; edits collide; merge is untangleable |
| Verification | Loop generates self-approved output at machine speed; a statistical impossibility of zero REJECTs goes unnoticed |
| Persistence | Loop rediscovers the same work each run; or worse, redoes it and conflicts with the previous attempt |
| Scheduling | Loop runs when the human remembers to run it; last run was the day it was demoed |

**Why the evaluator gets its own template:**

The paper's most empirically grounded claim is that making a generator self-critical
fails, while tuning a separate skeptical evaluator succeeds. Anthropic engineer
Prithvi Rajasekaran found this specifically while building long-running agentic
applications: the context in which code was written is full of the reasons it was
written that way, so the generator cannot see its own output clearly. A fresh agent
with instructions to default to doubt does not have this problem.

The evaluator template is more prescriptive than the other templates because the
failure mode it prevents — the Nodding Loop — is the most common and the most
dangerous. A loop that has never produced a REJECT is not a functioning loop.

---

## Templates

The templates are the canonical loop artifacts. The loop-engineering skill bundles
the same files under `skills/loop-engineering/templates/` — BUILD mode copies from
there, customizes `CUSTOMIZE` markers from the interview, and writes to the project.
Path A (skill) and Path B (direct copy from `templates/`) produce matching output.

They also exist at `templates/` in this repo so developers who already know what
they want can copy them directly without running the interactive setup.

Each template has `<!-- CUSTOMIZE -->` markers at the decision points that must be
adapted per project. Everything outside those markers is intentionally opinionated
and should be left as-is unless you have a specific reason to change it.

The `Stop` section in the loop-triage template is the one section that must never
be removed. It encodes the boundary between what the loop does autonomously and what
requires a human. The IEEE paper calls this "the open door" — the checkpoint that
keeps a human capable of saying no to a machine built to say yes at speed.

---

**中文文档:** [docs/zh/why-these-exist.md](zh/why-these-exist.md)
