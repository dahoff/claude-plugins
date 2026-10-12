# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal Claude Code plugin marketplace (marketplace name: `dahoff`). It holds skills now, and later agents, hooks, commands, rules and other extensions.

## Layout

- `.claude-plugin/marketplace.json` is the catalog. Every plugin must have an entry in its `plugins` array, or it can't be installed.
- `plugins/<plugin-name>/` holds one plugin per directory. Its manifest is at `.claude-plugin/plugin.json`, with components beside it: `skills/<skill>/SKILL.md`, `agents/*.md`, `commands/*.md`, `hooks/hooks.json`.
- `templates/plugin/` is a starter plugin to copy. It is not listed in the marketplace.

## Adding a plugin

1. `cp -r templates/plugin plugins/<name>`, then set `name`, `description` and `version` in its `plugin.json`. The directory name, manifest `name` and marketplace entry `name` must all match.
2. Add an entry to `marketplace.json`: `{"name": "<name>", "description": "...", "source": "./plugins/<name>"}`.
3. Validate both:
   ```
   claude plugin validate .
   claude plugin validate plugins/<name>
   ```

## Versioning

Installs are copied into `~/.claude/plugins/cache/dahoff/<plugin>/<version>/`. They aren't symlinked, so edits here don't reach installed copies until the plugin is updated (`/plugin` → update, or `claude plugin update <name>@dahoff`). Bump `version` in `plugin.json` when releasing changes. Without a version, the cache directory is named `unknown` and updates are unreliable. `claude plugin tag plugins/<name>` creates a `{name}--v{version}` release tag.

## Local testing

- Load one plugin without installing it: `claude --plugin-dir plugins/<name>`
- Add this repo as a marketplace: `/plugin marketplace add /home/david/git/claude-skills`, then `/plugin install <name>@dahoff`
