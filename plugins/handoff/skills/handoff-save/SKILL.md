---
name: handoff-save
description: Save the current session state to a handoff note so context can be cleared and the work resumed later with handoff-resume.
argument-hint: "[topic]"
disable-model-invocation: true
---

# Handoff: Save

Write a handoff note that lets a fresh session, with no memory of this conversation, pick up the work exactly where it stopped. Optional argument: a short topic for the filename.

The note is a pointer map, not a transcript. Capture current state and next steps in full; compress everything finished to one line; reference sources (files, commits, docs) instead of copying them.

## 1. Gather facts

Run this and use its output. Don't write the header from memory.

```bash
ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
DIR="$HOME/.claude/handoffs/$(printf '%s' "$ROOT" | sed 's|[/.]|-|g')"
mkdir -p "$DIR/archive"
echo "ROOT=$ROOT"; echo "DIR=$DIR"; echo "STAMP=$(date +%Y-%m-%d-%H%M)"
if git -C "$ROOT" rev-parse --git-dir >/dev/null 2>&1; then
  echo "BRANCH=$(git -C "$ROOT" branch --show-current)"
  echo "HEAD=$(git -C "$ROOT" log -1 --format='%h %s')"
  echo "--- status"; git -C "$ROOT" status --short
  echo "--- recent commits"; git -C "$ROOT" log --oneline -10
fi
```

## 2. Write the note

Path: `$DIR/<STAMP>-<topic>.md`. Topic is the argument if given, otherwise 2–4 words describing the work, kebab-case.

Use this structure. Drop a section only if it would be empty; write "None" for Open questions rather than omitting it.

```markdown
# Handoff: <topic>

- **Saved:** <YYYY-MM-DD HH:MM>
- **Project:** <ROOT>
- **Branch / HEAD:** <branch> @ <short hash> <subject>
- **Uncommitted:** <files from git status, or "clean">

## Goal
<1–3 sentences: what we're ultimately trying to achieve and why.>

## Current state
<What is in progress right now and exactly where it stopped. Name the file/function/step. Say what works and what doesn't yet.>

## Next steps
1. <Concrete, ordered actions a new session can execute without asking. Include commands to run where relevant.>

## Decisions & constraints
- <Choice made — why. Include preferences and rules the user stated this session.>

## Dead ends & gotchas
- <What was tried and failed, and why — so it isn't retried.>

## Open questions
- <Unresolved questions for the user, or blockers.>

## Completed
- ✅ <Task name> (<commit hash, if any>)

## References
- `<path>:<line>` — <why it matters>
- <commit hash> — <what it contains>
- <spec/plan/doc path, PR, issue, or URL> — <what it's for>
```

Guidelines:
- **Completed tasks are one line each.** No detail unless a later step depends on it, in which case put that detail under Current state or Decisions.
- **Reference, don't paste.** Point to `path:line`, commit hashes, spec/plan files, PRs and URLs. Only inline a snippet if it lives nowhere else (e.g., an unsaved command or error message).
- **Be specific.** "Fix the parser" is useless; "`src/parse.ts:142` drops trailing commas — add a case in `parseList` and rerun `npm test -- parse`" is useful.
- **Record unverified claims as such.** If something is believed but not tested, say so.
- Keep it under ~150 lines.

## 3. Report

Tell the user the saved file path, and that they can now run `/clear` and then `/handoff-resume`.
