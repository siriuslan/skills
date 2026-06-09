---
name: journal
description: Write a structured journal entry after completing a task. Records decisions, gotchas, and results for future sessions. Use after code changes, config changes, investigations, Postman modifications, or skill creation.
---

# Journal Skill

Write a structured journal entry to record completed work, decisions, and gotchas for future sessions.

## When to Use

Invoke this skill after completing a task that involves:
- Code changes (features, bug fixes, refactoring)
- Configuration or infrastructure changes
- Investigation/research that produced non-obvious findings
- Postman collection/flow modifications
- Skill creation or project setup decisions

Do NOT journal trivial tasks like:
- Answering a simple question
- Reading a file without making changes
- One-off lookups or explanations

## How It Works

### 1. Ask for Jira Task ID

Before writing the journal entry, ask the user:
> "Is there a Jira task ID for this work? (e.g., DAI-123). If not, I'll use a date-based name."

Use the response to determine the detail file name:
- **If a Jira task exists**: `{JIRA-ID}.md` (e.g., `DAI-123.md`)
- **If no Jira task**: `{YYYY-MM-DD}-{short-slug}.md` (e.g., `2026-05-29-postman-login-flow.md`)

### 2. Write the Detail File

Location: `{project-root}/.claude/journal/{filename}`

Structure:
```markdown
# {Title}

## What
One paragraph summarising what was done.

## Decisions
- Key decision and why it was made
- Alternative considered and why it was rejected
- Constraint or gotcha discovered

## Result
What the end state looks like — IDs, file paths, commands, or URLs that a future session would need to pick up the work.
```

Rules:
- Keep each section concise — aim for 3-8 bullet points max in Decisions
- Only record what isn't obvious from the code/git diff
- Include IDs, URLs, or paths that would be hard to re-discover
- Do not duplicate content already in CLAUDE.md or code comments

### 3. Update the Journal Index in MEMORY.md

Add one line under the `## Journal` heading in the project's `MEMORY.md`:

```
- [{Title}](journal/{filename}) — {one-line hook, under 100 chars}
```

If `MEMORY.md` doesn't exist yet, create it with:
```markdown
## Journal

- [{Title}](journal/{filename}) — {one-line hook}
```

If the `## Journal` heading doesn't exist, append it.

### 4. Create the journal directory if needed

Run `mkdir -p {project-root}/.claude/journal/` before writing the detail file.

## Example

**MEMORY.md entry:**
```
- [Split Postman login into 5 steps](journal/2026-05-29-postman-login-flow.md) — collection variables, clean TOTP impl, no environment needed
```

**Detail file** (`journal/2026-05-29-postman-login-flow.md`):
```markdown
# Split Postman login into 5 steps

## What
Decomposed the monolithic "Login to SF360" request (73KB pre-request script with bundled jsSHA) into 5 sequential requests in the "Requests for flow: reset-roni-workpaper" Postman collection.

## Decisions
- All state uses pm.collectionVariables (not pm.environment) — environment requires manual selection, fails silently without one
- Rewrote minified jsSHA as clean inline SHA-1/HMAC/TOTP from RFC specs — same output, readable
- Runtime tokens (cognito_session, totp_code, id_token) stored as collection variables with empty defaults

## Result
Collection ID: 30002654-8481e873-e391-4544-a791-8dd0009c957b
5 requests: InitiateAuth → RespondToAuthChallenge → login_token_check → selectfirm → EntryPoint
Postman Flow must be updated manually (no Flow editing API available)
```
