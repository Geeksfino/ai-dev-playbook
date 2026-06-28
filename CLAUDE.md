# CLAUDE.md

LLMs make the same mistakes repeatedly. These rules exist to prevent them.

## 1. Read Before You Write

Before writing anything:
- Read the files you're about to modify. Not skim. Read.
- Find how similar things are done elsewhere and follow that pattern.
- Check existing imports — don't introduce a library the project doesn't use.
- Look at test files to understand expected behavior, not assumed behavior.

If you don't see a pattern for something, ask: "I don't see a pattern for X — should I follow Y or do something different?"

## 2. Think Before You Code

**State your assumptions.** If the task is ambiguous, say what you're assuming before you start. "I'm assuming JWT auth with httpOnly cookies — let me know if you want something different." Guessing silently wastes hours.

**Name the tradeoffs.** "This trades memory for speed and introduces cache invalidation." Say it before writing 200 lines.

**Present options briefly when they exist.** Two, maybe three. With a recommendation. Not five.

**If something is confusing, stop.** Say what's confusing and ask. Don't fill confusion with plausible-sounding code.

## 3. Simplicity

Write the minimum code that solves this specific problem right now.

- **No premature abstraction.** One email type needed? Write `send_welcome_email(user)`, not an `EmailService` with a strategy pattern.
- **No speculative error handling.** Only handle errors that can actually occur.
- **No unnecessary configurability.** Hardcode things until there's a real reason not to.
- **No dead flexibility.** Interfaces with one implementation, abstract classes with one child — these cost cognitive overhead and provide nothing until a second case exists.

Test: if someone asks "why is this abstracted?" and the answer is "in case we need to…" — revert it.

## 4. Surgical Changes

Your diff should be as small as possible.

- Don't touch what you weren't asked to touch — not variable names, not comments, not import order.
- Match the existing style exactly: quotes, casing, semicolons, indentation.
- Clean up after yourself only: remove imports/variables/functions made unused by *your* change. Leave pre-existing dead code alone.
- Don't reformat. Don't run a linter on a file that wasn't formatted with that linter.

Test: can you justify every changed line with a direct connection to what was asked?

## 5. Verification

- **Bug fix? Write a failing test first.** Watch it fail. Fix the bug. Watch it pass. This is the only way to prove you fixed it.
- **Run existing tests before and after.** If tests were already failing before your change, say so explicitly.
- **Test behavior, not implementation.** A test that checks a constructor sets properties is worthless. Test what actually matters.
- **Can't write a test? Say why.** "I can't easily test this because DB calls are tightly coupled to business logic" is useful signal.

## 6. Goal-Driven Execution

Every task needs a clear success criterion before you start. Transform vague tasks into verifiable ones:

- "Add validation" → "reject missing/invalid email, return 400 with a descriptive message, add tests for both cases"
- "Fix the bug" → "write a test that reproduces it, make it pass, verify existing tests still pass"
- "Improve performance" → "profile first, identify the bottleneck, fix that specific thing, measure again"

For multi-step tasks, state the plan before executing and let the user catch wrong assumptions before you waste time implementing them.

## 7. Debugging

- Read the full error message including the stack trace before generating any fix.
- Reproduce the problem before changing anything. If you can't reproduce it, you can't verify the fix.
- Change one thing at a time. Test after each change.
- Don't add a null check without understanding why the value is null. The workaround hides the bug.
- If stuck after two attempts, say so: "I've tried X and Y. I think the issue is Z but I'm not sure."

## 8. Dependencies

Before adding a package: can you use what's already in the project? Can the standard library do it? Is the package maintained? How large is it?

When you do add one, say why: "I'm adding zod because this project needs runtime schema validation and nothing in the existing dependencies does this."

## 9. Common Failure Modes

**Kitchen Sink** — asked to add one feature, restructures half the codebase. Do the one thing.  
**Wrong Abstraction** — builds a generic solution for a problem that exists in one place. Copy-paste twice before abstracting.  
**Invisible Decision** — makes an architectural choice (schema, API shape, auth) without flagging it. These are hard to reverse.  
**Optimistic Path** — handles the happy path, ignores everything else. Think: API returns 500. File doesn't exist. Form is empty.  
**Knowledge Hallucination** — uses an API or parameter that doesn't exist or was removed. If not 100% sure, say so and verify.  
**Style Drift** — writes in preferred style instead of matching the project. Match the codebase.  
**Runaway Refactor** — fix cascades into 15 files. Stop. Tell the user. Get buy-in before continuing.

---

*These guidelines are working when: diffs contain only requested changes, clarifying questions come before implementation, and code doesn't need to be rewritten after delivery.*
