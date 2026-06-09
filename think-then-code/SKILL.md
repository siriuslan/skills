---
name: think-then-code
description: Cost-optimised workflow where Opus thinks and plans, then delegates implementation to a Sonnet sub-agent via a spec file. Use when implementing features, fixing bugs, or refactoring code to reduce token cost while maintaining quality.
---

# Think-then-Code Skill

Opus plans and reviews. Sonnet implements. A spec file bridges the two.

## When to Use

Use this workflow for tasks that involve writing or modifying code:
- Implementing a new feature
- Fixing a bug with a known or investigated root cause
- Refactoring existing code
- Adding tests

Do NOT use for:
- Pure research or investigation (no code output)
- Trivial one-line changes
- Tasks where the implementation is ambiguous and needs back-and-forth with the user

## Workflow

### Phase 1: Think (Opus)

1. Analyse the task and read all relevant files
2. Make all architectural decisions
3. Write the spec file to `.claude/impl-spec.md` in the project root

### Phase 2: Code (Sonnet sub-agent)

Spawn a Sonnet sub-agent with the Agent tool:

```
Agent({
  description: "<short task summary>",
  model: "sonnet",
  prompt: "Read the implementation spec at .claude/impl-spec.md and implement exactly as described. After implementation, run the verification steps listed in the spec. Report what you did and any issues encountered."
})
```

### Phase 3: Review (Opus)

1. Read the files that Sonnet modified
2. Compare against the spec — check nothing was missed or done incorrectly
3. If issues exist: either fix directly (if small) or re-spawn Sonnet with a correction prompt
4. Delete `.claude/impl-spec.md` after the task is complete
5. Report result to the user

## Spec File Format

Write `.claude/impl-spec.md` with this structure:

```markdown
# Task: {title}

## Context
Brief background — what exists today and why this change is needed.

## Changes Required

### File: {path/to/file}
- What to add, modify, or remove
- Be specific: function names, signatures, behaviour
- Reference line numbers when helpful

### File: {path/to/another-file}
- ...

## Constraints
- Style rules (match existing patterns, no comments, etc.)
- Dependencies (don't add new ones unless specified)
- Boundaries (don't touch unrelated code)

## Verification
- Tests to run (command)
- Expected outcomes
- Manual checks if applicable
```

### Spec Writing Rules

- Be precise enough that Sonnet can implement without guessing
- Include file paths and function names — not just abstract descriptions
- State what NOT to do (prevents Sonnet from over-engineering)
- Include the project's coding conventions from CLAUDE.md
- Reference existing patterns in the codebase for Sonnet to follow

## Example

### Spec (`impl-spec.md`):
```markdown
# Task: Add retry logic to DocumentApiClient

## Context
DocumentApiClient (src/main/scala/.../DocumentApiClient.scala) makes HTTP calls
to Document API V2. Currently no retry on transient failures.

## Changes Required

### File: src/main/scala/.../DocumentApiClient.scala
- Add private method `withRetry[T](op: => Future[T], maxAttempts: Int = 3): Future[T]`
- Uses exponential backoff: 500ms, 1000ms, 2000ms
- Only retry on 5xx responses and timeouts, NOT on 4xx
- Wrap `downloadSingleDocument` and `uploadDocument` with `withRetry`

### File: src/test/scala/.../DocumentApiClientSpec.scala
- Test: retries on 503, succeeds on 2nd attempt
- Test: does not retry on 400
- Test: gives up after 3 attempts, surfaces last error

## Constraints
- Use existing Akka scheduler for delays
- Scala 2 syntax, no comments
- Do not modify build.sbt

## Verification
- Run: `sbt test`
- All existing tests pass
- 3 new tests pass
```

### Sub-agent prompt:
```
Read the implementation spec at .claude/impl-spec.md and implement exactly
as described. Follow the constraints section strictly. Run the verification
steps when done and report results.
```

### Review checklist:
- Did Sonnet modify only the files listed in the spec?
- Do the changes match the described behaviour?
- Are constraints respected (no new deps, correct style)?
- Did verification pass?
