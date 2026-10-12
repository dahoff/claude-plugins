# dahoff's Claude Code plugins

Personal skills, agents, hooks and other Claude Code extensions, packaged as a plugin marketplace.

## Install

```
/plugin marketplace add <github-user>/<repo>
/plugin install <plugin-name>@dahoff
```

## Plugins

- **handoff**: `/handoff-save` writes a handoff note for the current session; after `/clear`, `/handoff-resume` picks it back up. Notes are stored in `~/.claude/handoffs/<project>/`.
