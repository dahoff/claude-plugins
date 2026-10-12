---
name: handoff-resume
description: Resume work from a handoff note saved by handoff-save, then archive the note and prune old archived notes.
argument-hint: "[name-or-topic]"
disable-model-invocation: true
---

# Handoff: Resume

Load a handoff note written by `handoff-save`, rebuild enough context to continue, archive the note, and prune the archive. Optional argument: part of a handoff filename or topic.

Archive limits (edit here to change):
- `MAX_AGE_DAYS=10`: notes archived more than this many days ago are deleted.
- `MAX_COUNT=20`: only this many newest archived notes are kept.

## 1. Find the note

```bash
ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
DIR="$HOME/.claude/handoffs/$(printf '%s' "$ROOT" | sed 's|[/.]|-|g')"
echo "DIR=$DIR"
ls -1 "$DIR"/*.md 2>/dev/null | sort -r
```

- **No notes:** tell the user there is no active handoff for this project (mention `$DIR/archive/` if it has notes) and stop.
- **Argument given:** use the note whose filename contains it. If several match, list them and ask.
- **No argument, one note:** use it.
- **No argument, several notes:** list them (newest first) and ask which one. Suggest the newest.

## 2. Read and verify

Read the whole note. Then compare the live state against its header:

```bash
git -C "$ROOT" branch --show-current
git -C "$ROOT" log -1 --format='%h %s'
git -C "$ROOT" status --short
git -C "$ROOT" log --oneline -10
```

If the branch, HEAD or uncommitted files differ from the note, tell the user what changed (e.g., new commits since the save) before continuing. Don't try to fix it.

Read only the referenced files needed for the first next step. Leave the rest for when they're needed.

## 3. Archive and prune

```bash
mkdir -p "$DIR/archive"
mv "<note path>" "$DIR/archive/"
touch "$DIR/archive/<note filename>"  # age counts from archive time
MAX_AGE_DAYS=10; MAX_COUNT=20
find "$DIR/archive" -maxdepth 1 -name '*.md' -type f -mtime +"$MAX_AGE_DAYS" -delete
ls -1t "$DIR/archive"/*.md 2>/dev/null | tail -n +"$((MAX_COUNT + 1))" | xargs -r -d '\n' rm -f
```

## 4. Brief the user and wait

Give a short summary:
- **Goal** — one sentence.
- **Where we left off** — current state in 1–3 sentences.
- **Drift** — anything that changed since the save, or "none".
- **Next step** — the first item from Next steps.
- **Open questions** — if any.

Mention the archived note's path. Then ask whether to proceed with the next step. Don't start work until the user confirms.
