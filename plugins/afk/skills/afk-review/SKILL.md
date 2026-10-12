---
name: afk-review
description: Review the decisions logged by afk while the user was away, apply any changes they want, then mark the log reviewed.
argument-hint: "[name-or-topic]"
disable-model-invocation: true
---

# AFK: Review

Walk the user through the decisions logged during an `afk` session. Optional argument: part of a log filename or topic.

## 1. Find the log

```bash
ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
DIR="$HOME/.claude/afk/$(printf '%s' "$ROOT" | sed 's|[/.]|-|g')"
echo "DIR=$DIR"
ls -1 "$DIR"/*.md 2>/dev/null | sort -r
```

- **No logs:** tell the user there's nothing to review for this project (mention `$DIR/reviewed/` if it has logs) and stop.
- **Argument given:** use the log whose filename contains it. If several match, ask.
- **No argument:** use the only log, or list them newest first and ask which one.

## 2. Present it

Read the whole log. Show:
- **Task, status and summary** from the header.
- **Needs you:** every Deferred and Blocked item, and every low-confidence item. Put these first.
- **Other decisions:** the rest, one line each (number, title, what was chosen).

Keep it scannable. Don't restate the full entries unless asked.

## 3. Act on feedback

Ask the user which items they want changed, or whether to accept everything. For each change: make it, then add a line under that entry in the log, e.g. `→ Reviewed: changed to X.` Carry out Deferred actions only when the user says to.

## 4. Mark reviewed

Once the user is done:

```bash
mv "<log path>" "$DIR/reviewed/"
```

Tell them it's been moved.
