# Blueshoe plugins for Claude

A plugin marketplace for firms that run matters in [Blueshoe](https://blueshoe.com).

| Plugin | What it does |
|---|---|
| [`blueshoe-matter-work`](plugins/blueshoe-matter-work) | Start every matter task from the Blueshoe record; capture every email, document and deadline back into it; firm-agnostic practice rules. |

## Install

**Firm-wide, from claude.ai organization settings** (chat, Cowork, and Claude Code sessions signed in with the firm account): an Owner opens Organization settings > Plugins & skills > Add > Sync from GitHub, picks this repository, leaves "Sync automatically" on, and sets each plugin's availability. Members need no GitHub access. Firms outside Blueshoe: ask for read access to this repository for your GitHub organization first.

**Claude Code, any user:** `/plugin marketplace add blueshoeai/claude-plugins-public` then `/plugin install blueshoe-matter-work@blueshoe`.

## Release

1. Edit under `plugins/<name>/`.
2. Bump `version` in `plugins/<name>/.claude-plugin/plugin.json` and in `.claude-plugin/marketplace.json`.
3. Open a pull request and merge to `main`. Org-synced catalogs re-sync on merge; Claude Code picks the new version up on next start.

The public mirror `blueshoeai/claude-plugins-public` is pushed from the same commit.
