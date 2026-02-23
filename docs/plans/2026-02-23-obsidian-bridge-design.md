# obsidian-bridge — Design Document

**Date:** 2026-02-23
**Status:** Approved
**Author:** JB

---

## Overview

A Claude Code plugin that creates a zero-copy Obsidian vault over an existing `~/.claude/` directory. All of Claude's knowledge (rules, agents, skills, contexts, plans, memory, sessions) stays exactly where it is — a vault directory of symlinks brings Obsidian's search and browsing UI to the data, with no duplication and no sync.

---

## The Problem

Claude Code accumulates knowledge across sessions:
- Rules and conventions in `~/.claude/rules/`
- Agents in `~/.claude/agents/`
- Skills in `~/.claude/skills/`
- Session saves in `~/.claude/contexts/`
- Plans in `~/.claude/plans/`
- Per-project memory in `~/.claude/projects/*/memory/`

None of this is browsable. There's no way to search across 154 session saves, or see the graph of how your memory files reference each other, or open a plan from three weeks ago.

---

## The Wrong Solutions

**Sync tools** — export to Notion, Obsidian Sync, etc. Create a second source of truth. Always drift. Double the maintenance.

**Export scripts** — snapshot and copy. Stale immediately. Now you have two copies and neither is authoritative.

**Writing directly to a new location** — changes Claude's write paths. Breaks existing workflows.

---

## The Right Solution: Zero-Copy Vault

Don't move the data. Bring the viewer to the data.

A vault directory (`~/Obsidian-Claude/`) contains symlinks to each `~/.claude/` subdirectory. Obsidian reads the symlinked files live. Claude writes to `~/.claude/` as always. The vault never needs updating — new files appear instantly.

The only writable content in the vault is a set of dashboard `.md` pages (Home, All Memory, All Skills, etc.) that serve as curated indexes. These are regenerated on demand by `/vault-sync`.

---

## Repository Structure

```
obsidian-bridge/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── vault-init.md
│   ├── vault-sync.md
│   └── vault-open.md
├── README.md
└── LICENSE
```

---

## Commands

### `/vault-init`

Full first-time setup. Idempotent — safe to run again.

**Steps:**
1. Check Obsidian installed — offer `brew install --cask obsidian` if not (macOS); note manual install for Linux/Windows
2. Read `~/.claude/obsidian-config.json` for vault path (default: `~/Obsidian-Claude`)
3. Create vault directory + `memory/` + `playgrounds/` subdirs
4. Dynamically discover `~/.claude/` directories (rules, skills, agents, docs, commands, knowledge, codemaps, routines, hooks, contexts, plans, sessions)
5. Create symlinks for each discovered directory
6. Dynamically discover `~/.claude/projects/*/memory/` folders — symlink each under `memory/<project-slug>/`
7. Discover `~/Desktop/*.html` — symlink into `playgrounds/` (skip HIPAA/private files)
8. Generate all dashboard pages from discovered structure (not hardcoded)
9. Write/update `~/.claude/obsidian-config.json`
10. Open vault in Obsidian

### `/vault-sync`

Regenerates dashboard `.md` pages from current `~/.claude/` structure. Run after adding a major new project. Reports added/removed entries.

### `/vault-open`

Reads vault path from `~/.claude/obsidian-config.json`. Opens in Obsidian.

---

## Dashboard Pages Generated

| Page | Contents |
|------|----------|
| `Home.md` | Main index, links to all category pages |
| `All Memory.md` | All project memory files |
| `All Skills.md` | Defined skills + learned skills |
| `All Agents.md` | Agent catalog with purpose |
| `All Rules.md` | Convention files with summaries |
| `All Plans.md` | Plans index, recent first |
| `All Contexts.md` | Session saves, search guidance |
| `All Playgrounds.md` | HTML prototypes |
| `All Commands.md` | All slash commands |

---

## Configuration

`~/.claude/obsidian-config.json`:
```json
{
  "vault_path": "~/Obsidian-Claude"
}
```

User can set `vault_path` to any location — iCloud, Google Drive, etc. for cross-device access.

---

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Dynamic discovery | Works for any user's `~/.claude/` structure, not just the author's |
| Idempotent init | Safe to re-run; skips existing symlinks, updates dashboards |
| No agents | Shell + file operations don't need agent overhead |
| No sync infrastructure | Symlinks are inherently live |
| Dashboards in vault, not `~/.claude/` | Keeps `~/.claude/` clean; vault is the browsing layer |

---

## README / Manifesto Sections

1. What is obsidian-bridge?
2. The problem — Claude's knowledge is invisible
3. The wrong solutions — sync, export, duplicate
4. The right solution — zero-copy vault via symlinks
5. How it works
6. What you get (search, graph, always current)
7. Installation
8. Commands reference
9. Configuration
10. Recommended Obsidian plugin: Dataview

---

## What's Out of Scope

- Windows support (symlinks work differently; out of scope for v1)
- Obsidian plugin / Dataview queries (those live in Obsidian, not here)
- Writing back to `~/.claude/` from Obsidian (read-only browsing is the intent)
- Auto-opening Obsidian on Claude startup (hook could do this but it's invasive)
