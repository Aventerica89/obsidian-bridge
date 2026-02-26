---
description: Regenerate Obsidian vault dashboard pages from current ~/.claude/ structure. Run after adding new projects.
allowed-tools: [Bash, Read, Write, Glob]
---

# Vault Sync

Regenerate all dashboard .md pages in the vault from the current `~/.claude/` structure.
Does NOT touch symlinks — only updates the `Home.md` and `All *.md` index pages.

## Step 1: Determine vault path

Read `~/.claude/obsidian-config.json`.
Extract `vault_path`. Expand `~` to the actual home directory.
Default to `~/Obsidian-Claude` if not set.

If the vault does not exist at that path, tell the user to run `/vault-init` first.

## Step 2: Snapshot current dashboard state

For each of these files, note whether it exists and its current line count:
- `{vault_path}/Home.md`
- `{vault_path}/All Memory.md`
- `{vault_path}/All Skills.md`
- `{vault_path}/All Agents.md`
- `{vault_path}/All Rules.md`
- `{vault_path}/All Plans.md`
- `{vault_path}/All Contexts.md`
- `{vault_path}/All Playgrounds.md`
- `{vault_path}/All Commands.md`

## Step 3: Regenerate all dashboard pages

Read the actual current state of each symlinked directory.
Overwrite all dashboard pages with fresh content.

**Home.md** — List all symlinked directories with their file counts. Include links to each `All *.md` dashboard page.

**All Memory.md** — List every file in every `{vault_path}/memory/*/` directory, grouped by project slug as a heading. Use `[[wikilink]]` format for each file.

**All Skills.md** — List everything in `{vault_path}/skills/`. Separate top-level items (defined skills) from `skills/learned/` contents under their own heading.

**All Agents.md** — List each `.md` file in `{vault_path}/agents/` with a one-line description (from the file's `description` frontmatter field, or first sentence of content if no frontmatter).

**All Rules.md** — List each `.md` file in `{vault_path}/rules/` with a one-line summary from the first heading or opening sentence of the file.

**All Plans.md** — List the 20 most recently modified files in `{vault_path}/plans/` (use `ls -t`), plus a note to use search for more.

**All Contexts.md** — Total count of files in `{vault_path}/contexts/`. A table mapping project names to search keywords. Guidance to use Obsidian search (`Cmd+Shift+F`) to find sessions by project.

**All Playgrounds.md** — List each file in `{vault_path}/playgrounds/` with a `[[wikilink]]`.

**All Commands.md** — List each `.md` file in `{vault_path}/commands/` in alphabetical order as a table with the file's `description` frontmatter field.

## Step 4: Report changes

For each dashboard page, report the before and after line counts and a summary of added or removed entries.

Example output:
```
Synced vault at ~/Obsidian-Claude

All Memory.md:    42 → 47 lines  (+5 entries)
All Plans.md:     38 → 41 lines  (+3 entries)
All Agents.md:    no change
All Skills.md:    no change
All Contexts.md:  no change
```
