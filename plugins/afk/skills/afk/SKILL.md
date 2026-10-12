---
name: afk
description: The user is away and can't answer questions or review work. Keep working autonomously, make the choices they'd most likely want, and log significant decisions for review with afk-review.
argument-hint: "[task or topic]"
disable-model-invocation: true
---

# AFK: work while the user is away

The user is away. Don't stop to ask questions or wait for review. When you'd normally ask, pick what the user would most likely want, log it if it matters, and keep going. Optional argument: the task, or a short topic for the log filename.

## 1. Start the log

```bash
ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
DIR="$HOME/.claude/afk/$(printf '%s' "$ROOT" | sed 's|[/.]|-|g')"
mkdir -p "$DIR/reviewed"
echo "DIR=$DIR"; echo "STAMP=$(date +%Y-%m-%d-%H%M)"
```

Create `$DIR/<STAMP>-<topic>.md` (topic: 2–4 words, kebab-case):

```markdown
# AFK: <topic>

- **Started:** <YYYY-MM-DD HH:MM>
- **Project:** <ROOT>
- **Task:** <one sentence>
- **Status:** in progress

## Decisions
```

## 2. Work

- **Don't ask.** No AskUserQuestion, no "should I…?" pauses. If another skill tells you to ask the user something, answer it yourself as the user likely would and log it if it's significant.
- **Choose like the user.** Use their stated preferences, CLAUDE.md, memory, and existing code conventions. With no signal, pick the conventional, simpler, easier-to-undo option.
- **Hold back irreversible or outward-facing actions.** Don't push, deploy, publish, delete data, send messages or spend money unless the task explicitly said to. Prepare them and log them as **Deferred**.
- **When blocked**, work around it if you can and log how. Otherwise log it as **Blocked** and move to other work.

## 3. Log decisions as you make them

Append to the log right away, not at the end, so nothing is lost if context is compacted.

Log the bigger calls: interpreting an unclear requirement, choosing an approach or design, adding or dropping scope, picking a dependency, workarounds, anything the user might reasonably have chosen differently. Skip small things like naming, formatting and obvious fixes.

Keep each entry to a few short lines:

```markdown
### <n>. <short title> · <Decision | Assumption | Deferred | Blocked> · <high | med | low confidence>
<What you chose, in one sentence.> **Why:** <one sentence.> **Alt:** <main alternative, if any.> **Where:** <file/commit, if any.>
```

## 4. Finish

When the work is done, or you can't go further, update the header: set **Status** to `done`, `partial` or `blocked`, and add a **Summary** line (1–2 sentences). Then tell the user the log path and that `/afk-review` will walk them through it.
