# obsidian-bridge Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build and publish a Claude Code plugin that creates a zero-copy Obsidian vault over `~/.claude/` via symlinks, with three commands: `/vault-init`, `/vault-sync`, `/vault-open`.

**Architecture:** The plugin is three command files (markdown prompts) + a manifesto README + plugin.json. No compiled code. Each command file instructs Claude to perform shell operations at runtime. The commands are idempotent and dynamically discover `~/.claude/` structure rather than hardcoding paths.

**Tech Stack:** Claude Code plugin system (markdown commands), Bash (symlinks, file discovery), Obsidian (the viewer)

---

## Task 1: Initialize Git Repo

**Files:**
- Modify: `~/obsidian-bridge/` (already exists)
- Create: `~/obsidian-bridge/.gitignore`

**Step 1: Init git and add gitignore**

```bash
cd ~/obsidian-bridge
git init
```

Create `.gitignore`:
```
.DS_Store
*.local
```

**Step 2: Initial commit**

```bash
cd ~/obsidian-bridge
git add .gitignore
git commit -m "chore: initialize obsidian-bridge repo"
```

**Expected:** Git repo initialized, one commit.

---

## Task 2: Plugin Manifest

**Files:**
- Create: `~/obsidian-bridge/.claude-plugin/plugin.json`

**Step 1: Create the manifest**

```json
{
  "name": "obsidian-bridge",
  "description": "Zero-copy Obsidian vault over your ~/.claude/ knowledge base via symlinks. One command to make all of Claude's memory, rules, skills, agents, contexts, and plans browsable in Obsidian — no duplication, no sync.",
  "author": {
    "name": "JB",
    "url": "https://github.com/Aventerica89"
  }
}
```

**Step 2: Commit**

```bash
cd ~/obsidian-bridge
git add .claude-plugin/plugin.json
git commit -m "feat: add plugin manifest"
```

---

## Task 3: `/vault-init` Command

**Files:**
- Create: `~/obsidian-bridge/commands/vault-init.md`

This is the main command. It must be self-contained — no assumed context.

**Step 1: Write the command file**

```markdown
---
description: Set up an Obsidian vault that browses your ~/.claude/ knowledge base via symlinks. Run once to get started.
allowed-tools: [Bash, Read, Write, Glob]
---

# Vault Init

Create ~/Obsidian-Claude/ (or the path in ~/.claude/obsidian-config.json) with
symlinks to all ~/.claude/ content. Generates dashboard pages. Opens Obsidian.

## Step 1: Determine vault path

Read ~/.claude/obsidian-config.json if it exists:
- Extract vault_path field
- Expand ~ to the actual home directory
- Default to ~/Obsidian-Claude if file doesn't exist or field is missing

## Step 2: Check Obsidian is installed

Run: `ls /Applications/Obsidian.app 2>/dev/null || brew list --cask obsidian 2>/dev/null`

If not found, ask the user:
"Obsidian is not installed. Install it now via Homebrew? (brew install --cask obsidian)"

If yes: `brew install --cask obsidian`
If no: stop and tell user to install Obsidian manually from obsidian.md

## Step 3: Create vault directory structure

```bash
mkdir -p {vault_path}/memory
mkdir -p {vault_path}/playgrounds
```

## Step 4: Symlink ~/.claude/ content directories

For each of these directories, check if it exists in ~/.claude/ and create a symlink in {vault_path}/:

Directories to check:
- rules, skills, agents, docs, commands, knowledge, codemaps, routines, hooks, contexts, plans, sessions

For each existing directory:
```bash
ln -sf ~/.claude/{dir} {vault_path}/{dir}
```

If the symlink already exists and points to the right target, skip it silently.

## Step 5: Symlink project memory directories

List all directories in ~/.claude/projects/:
```bash
ls ~/.claude/projects/
```

For each project directory, check if a memory/ subdirectory exists inside it.
If it does, create a symlink in {vault_path}/memory/:

Derive a readable slug from the project directory name:
- Strip leading dashes/hyphens
- Replace the encoded path separators with dashes
- Keep it under 40 characters

```bash
ln -sf ~/.claude/projects/{project}/memory {vault_path}/memory/{slug}
```

Skip if symlink already exists.

## Step 6: Symlink Desktop HTML files

Find HTML files:
```bash
ls ~/Desktop/*.html 2>/dev/null
```

For each .html file found, symlink into {vault_path}/playgrounds/:
```bash
ln -sf ~/Desktop/{filename}.html {vault_path}/playgrounds/{filename}.html
```

Skip files that appear to be sensitive documents (HIPAA, policy, legal in filename).

## Step 7: Generate dashboard pages

Read the actual current structure and generate these markdown files in {vault_path}/:

**Home.md** — List all symlinked directories with file counts and links to All *.md pages.

**All Memory.md** — List every file in every {vault_path}/memory/*/ directory,
grouped by project. Use [[wikilinks]] format.

**All Skills.md** — List everything in {vault_path}/skills/. Separate
top-level items (defined skills) from skills/learned/ contents.

**All Agents.md** — List each .md file in {vault_path}/agents/ with a
one-line description extracted from the file's description frontmatter or first paragraph.

**All Rules.md** — List each .md file in {vault_path}/rules/ with a
one-line summary from the file's first heading or opening sentence.

**All Plans.md** — List the 20 most recently modified files in {vault_path}/plans/,
plus guidance to search for more.

**All Contexts.md** — Count of files in {vault_path}/contexts/, search guidance,
project keyword table.

**All Playgrounds.md** — List each file in {vault_path}/playgrounds/ with
a [[wikilink]].

**All Commands.md** — List each .md file in {vault_path}/commands/ grouped
by category (infer category from filename prefix or content).

When generating pages: if a page already exists, overwrite it. These pages
are always regenerated from current state.

## Step 8: Update obsidian-config.json

Write ~/.claude/obsidian-config.json:
```json
{
  "vault_path": "{vault_path}",
  "note": "Vault uses symlinks to all ~/.claude/ content. Zero duplication."
}
```

## Step 9: Open vault in Obsidian

```bash
open -a Obsidian {vault_path}
```

## Report to user

Show a summary:
- Vault location
- Symlinks created (count + list)
- Dashboard pages generated
- "Open Obsidian → 'Open folder as vault' → select {vault_path} if it doesn't open automatically."
```

**Step 2: Verify the command reads correctly**

Read `commands/vault-init.md` back and confirm it's complete and self-contained.

**Step 3: Commit**

```bash
cd ~/obsidian-bridge
git add commands/vault-init.md
git commit -m "feat: add /vault-init command"
```

---

## Task 4: `/vault-sync` Command

**Files:**
- Create: `~/obsidian-bridge/commands/vault-sync.md`

**Step 1: Write the command file**

```markdown
---
description: Regenerate Obsidian vault dashboard pages from current ~/.claude/ structure. Run after adding new projects.
allowed-tools: [Bash, Read, Write, Glob]
---

# Vault Sync

Regenerate all dashboard .md pages in the vault from the current ~/.claude/ structure.
Does NOT touch symlinks — only updates the Home.md and All *.md index pages.

## Step 1: Determine vault path

Read ~/.claude/obsidian-config.json.
Extract vault_path. Expand ~ to home directory.
Default to ~/Obsidian-Claude if not set.

If vault does not exist at that path, tell user to run /vault-init first.

## Step 2: Snapshot current dashboard state

For each of these files, note whether it exists and its current line count:
- {vault_path}/Home.md
- {vault_path}/All Memory.md
- {vault_path}/All Skills.md
- {vault_path}/All Agents.md
- {vault_path}/All Rules.md
- {vault_path}/All Plans.md
- {vault_path}/All Contexts.md
- {vault_path}/All Playgrounds.md
- {vault_path}/All Commands.md

## Step 3: Regenerate all dashboard pages

Follow the same dashboard generation logic as /vault-init Step 7.
Read the actual current state of each symlinked directory.
Overwrite all dashboard pages with fresh content.

## Step 4: Report changes

For each dashboard page, report:
- Lines before → lines after
- Any new entries added
- Any entries removed

Example output:
```
Synced vault at ~/Obsidian-Claude

All Memory.md:  42 → 47 lines (+5 entries: clarity/new-feature.md, ...)
All Plans.md:   38 → 41 lines (+3 entries)
All Agents.md:  no change
All Skills.md:  no change
```
```

**Step 2: Commit**

```bash
cd ~/obsidian-bridge
git add commands/vault-sync.md
git commit -m "feat: add /vault-sync command"
```

---

## Task 5: `/vault-open` Command

**Files:**
- Create: `~/obsidian-bridge/commands/vault-open.md`

**Step 1: Write the command file**

```markdown
---
description: Open your Obsidian vault. Reads vault path from ~/.claude/obsidian-config.json.
allowed-tools: [Bash, Read]
---

# Vault Open

Open the obsidian-bridge vault in Obsidian.

## Step 1: Read vault path

Read ~/.claude/obsidian-config.json.
Extract vault_path. Expand ~ to home directory.
Default to ~/Obsidian-Claude.

## Step 2: Verify vault exists

Check that vault_path directory exists:
```bash
ls {vault_path} 2>/dev/null
```

If it doesn't exist, tell user: "Vault not found at {vault_path}. Run /vault-init to set it up."

## Step 3: Open in Obsidian

```bash
open -a Obsidian {vault_path}
```

If Obsidian is not installed: "Obsidian is not installed. Run /vault-init which will offer to install it."
```

**Step 2: Commit**

```bash
cd ~/obsidian-bridge
git add commands/vault-open.md
git commit -m "feat: add /vault-open command"
```

---

## Task 6: README / Manifesto

**Files:**
- Create: `~/obsidian-bridge/README.md`

**Step 1: Write the full README**

The README IS the manifesto. Write it in full — this is the shareable document.

```markdown
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

Requires: Claude Code, Obsidian (the plugin will offer to install it)

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

## Recommended Obsidian Plugin: Dataview

Install the [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
community plugin to build dynamic tables from your vault content.

Example: all context saves from the last 30 days, sorted by project.

## How It Works

```
~/.claude/                    ~/Obsidian-Claude/
├── rules/          ←─────── rules -> ~/.claude/rules/
├── agents/         ←─────── agents -> ~/.claude/agents/
├── skills/         ←─────── skills -> ~/.claude/skills/
├── contexts/       ←─────── contexts -> ~/.claude/contexts/
├── plans/          ←─────── plans -> ~/.claude/plans/
└── projects/
    └── my-app/
        └── memory/ ←─────── memory/my-app -> ~/.claude/projects/my-app/memory/
```

The vault directory itself is just a container for symlinks and a few
dashboard `.md` files. Claude Code never writes to the vault. The vault
never writes to `~/.claude/`. They share the same files through the filesystem.

## Philosophy

Claude's knowledge base is yours. You built it across hundreds of sessions.
obsidian-bridge gives you a window into it without asking you to move it,
duplicate it, or trust a third-party sync service with it.

The vault is a view, not a copy.

## License

MIT
```

**Step 2: Commit**

```bash
cd ~/obsidian-bridge
git add README.md
git commit -m "docs: add manifesto README"
```

---

## Task 7: LICENSE

**Files:**
- Create: `~/obsidian-bridge/LICENSE`

**Step 1: Write MIT license**

```
MIT License

Copyright (c) 2026 JB (Aventerica89)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

**Step 2: Commit**

```bash
cd ~/obsidian-bridge
git add LICENSE
git commit -m "chore: add MIT license"
```

---

## Task 8: Move design docs and final commit

**Files:**
- Existing: `~/obsidian-bridge/docs/plans/` (already has design docs)

**Step 1: Add design docs to git**

```bash
cd ~/obsidian-bridge
git add docs/
git commit -m "docs: add design documents"
```

**Step 2: Verify final structure**

```bash
cd ~/obsidian-bridge
find . -not -path './.git/*' | sort
```

Expected output:
```
.
./.claude-plugin
./.claude-plugin/plugin.json
./.gitignore
./LICENSE
./README.md
./commands
./commands/vault-init.md
./commands/vault-open.md
./commands/vault-sync.md
./docs
./docs/plans
./docs/plans/2026-02-23-obsidian-bridge-design.md
./docs/plans/2026-02-23-obsidian-bridge-implementation.md
```

---

## Task 9: Publish to GitHub

**Step 1: Create the GitHub repo**

```bash
cd ~/obsidian-bridge
gh repo create Aventerica89/obsidian-bridge --public --description "Zero-copy Obsidian vault over your ~/.claude/ knowledge base" --push --source .
```

**Step 2: Verify repo is live**

```bash
gh repo view Aventerica89/obsidian-bridge
```

**Step 3: Test the install command**

In a Claude Code session, verify the plugin can be referenced:

```
claude plugin install github:Aventerica89/obsidian-bridge
```

(Optional: test in a fresh environment if available)

**Step 4: Add topics to repo**

```bash
gh api repos/Aventerica89/obsidian-bridge/topics \
  -X PUT \
  -f names[]="claude-code" \
  -f names[]="obsidian" \
  -f names[]="claude-code-plugin" \
  -f names[]="knowledge-base"
```

---

## Verification Checklist

After all tasks:

- [ ] `~/obsidian-bridge/` has clean git history (8-9 commits)
- [ ] `.claude-plugin/plugin.json` has name, description, author
- [ ] All three commands exist in `commands/`
- [ ] Each command is self-contained — no assumed context
- [ ] README covers: what, problem, wrong solutions, right solution, how it works, install, commands, config
- [ ] GitHub repo is public at `Aventerica89/obsidian-bridge`
- [ ] `claude plugin install github:Aventerica89/obsidian-bridge` resolves
