# obsidian-bridge

A Claude Code plugin that makes all of Claude's accumulated knowledge browsable
in Obsidian — without moving a single file.

## What is obsidian-bridge?

Claude Code builds up a rich knowledge base as you work: rules and conventions
in `~/.claude/rules/`, specialized agents in `~/.claude/agents/`, hundreds of
session saves in `~/.claude/contexts/`, per-project memory in
`~/.claude/projects/*/memory/`, learned patterns in `~/.claude/skills/learned/`.

None of it is browsable. It's all locked inside `~/.claude/` with no search,
no graph view, no way to see how it all connects.

obsidian-bridge fixes that in one command.

## The Problem

After months of working with Claude Code, a typical user accumulates:

- 100+ session context saves
- 50+ implementation plans
- 15+ custom agents
- 140+ learned skill patterns
- Per-project memory files across a dozen repos

Finding anything requires knowing exactly where to look. There's no full-text
search, no cross-project navigation, no way to browse the graph of what Claude
knows about your work.

## The Wrong Solutions

**Export to Notion / sync tools** — create a second source of truth that
immediately starts drifting from the original. Now you're maintaining two
systems. The exported copy is stale by the time you open it.

**Copy scripts** — same problem. A snapshot isn't a live view.

**Write to a new location** — changes Claude's write paths, breaks existing
workflows, requires migrating years of accumulated knowledge.

## The Right Solution: Zero-Copy Vault

Don't move the data. Bring the viewer to the data.

`~/Obsidian-Claude/` is a directory of symlinks. Every entry points directly
into `~/.claude/`. Obsidian reads the files live through the symlinks. Claude
writes to `~/.claude/` as always.

When Claude saves a new context, it appears in the vault instantly — not after
a sync, not after a push, not after running a script. The symlink is just a
path. There is no copy.

## What You Get

**Full-text search** — `Cmd+Shift+F` in Obsidian searches across every context
save, plan, memory file, rule, skill, and agent simultaneously.

**Graph view** — see how your memory files and contexts reference each other
across projects.

**Always current** — the vault can never be stale. New files appear the moment
Claude creates them.

**Zero maintenance** — no sync to run, no export to trigger, no copy to update.
The vault just exists.

**Configurable location** — point the vault at iCloud or Google Drive to browse
your Claude knowledge base from any device.

## Installation

```bash
claude plugin install github:Aventerica89/obsidian-bridge
```

Requires: Claude Code, Obsidian (the plugin will offer to install it via Homebrew).

## Setup

After installing the plugin, run:

```
/vault-init
```

That's it. The command will:

1. Create `~/Obsidian-Claude/` (or your configured path)
2. Symlink all `~/.claude/` content directories
3. Generate browsable dashboard pages
4. Open the vault in Obsidian

## Commands

| Command | Description |
|---------|-------------|
| `/vault-init` | First-time setup. Creates vault, symlinks, dashboards. Opens Obsidian. |
| `/vault-sync` | Regenerate dashboard pages after adding new projects. |
| `/vault-open` | Open the vault in Obsidian from anywhere. |

## Configuration

Set a custom vault path in `~/.claude/obsidian-config.json`:

```json
{
  "vault_path": "~/Library/Mobile Documents/com~apple~CloudDocs/Obsidian/Claude"
}
```

This puts the vault in iCloud, making your Claude knowledge base available
across all your devices. The symlinks still point to your local `~/.claude/` —
iCloud syncs the dashboard pages and vault metadata. The actual content files
are local only (which is correct — `~/.claude/` isn't synced and shouldn't be).

## Recommended Plugin: Dataview

Install the [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
community plugin to build dynamic tables from your vault content.

Example: all context saves from the last 30 days, sorted by project.

## How It Works

```
~/.claude/                    ~/Obsidian-Claude/
├── rules/          <------- rules -> ~/.claude/rules/
├── agents/         <------- agents -> ~/.claude/agents/
├── skills/         <------- skills -> ~/.claude/skills/
├── contexts/       <------- contexts -> ~/.claude/contexts/
├── plans/          <------- plans -> ~/.claude/plans/
└── projects/
    └── my-app/
        └── memory/ <------- memory/my-app -> ~/.claude/projects/my-app/memory/
```

The vault directory is a container for symlinks and a few dashboard `.md` files.
Claude Code never writes to the vault. The vault never writes to `~/.claude/`.
They share the same files through the filesystem.

## Philosophy

Claude's knowledge base is yours. You built it across hundreds of sessions.
obsidian-bridge gives you a window into it without asking you to move it,
duplicate it, or trust a third-party sync service with it.

The vault is a view, not a copy.

## License

MIT
