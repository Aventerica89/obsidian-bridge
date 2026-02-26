---
description: Set up an Obsidian vault that browses your ~/.claude/ knowledge base via symlinks. Run once to get started.
allowed-tools: [Bash, Read, Write, Glob]
---

# Vault Init

Create ~/Obsidian-Claude/ (or the path in ~/.claude/obsidian-config.json) with symlinks to all ~/.claude/ content. Generates dashboard pages. Opens Obsidian.

## Step 1: Determine vault path

Read ~/.claude/obsidian-config.json if it exists:
- Extract vault_path field
- Expand ~ to the actual home directory
- Default to ~/Obsidian-Claude if file doesn't exist or field is missing

## Step 2: Check Obsidian is installed

Run: `ls /Applications/Obsidian.app 2>/dev/null || brew list --cask obsidian 2>/dev/null`

If not found, ask the user:
"Obsidian is not installed. Install it now via Homebrew? (brew install --cask obsidian)"

If yes: run `brew install --cask obsidian`
If no: stop and tell user to install Obsidian manually from https://obsidian.md

## Step 3: Create vault directory structure

```bash
mkdir -p {vault_path}/memory
mkdir -p {vault_path}/playgrounds
```

## Step 4: Symlink ~/.claude/ content directories

For each of these directories, check if it exists in ~/.claude/ and create a symlink in {vault_path}/:

Directories to check: rules, skills, agents, docs, commands, knowledge, codemaps, routines, hooks, contexts, plans, sessions

For each existing directory, check before linking:
```bash
[ "$(readlink {vault_path}/{dir})" = "$HOME/.claude/{dir}" ] \
  || ln -sf ~/.claude/{dir} {vault_path}/{dir}
```

This skips the symlink if it already points to the correct target, and only recreates it if it is missing or points elsewhere.

## Step 5: Symlink project memory directories

List all directories in ~/.claude/projects/:
```bash
ls ~/.claude/projects/
```

For each project directory, check if a memory/ subdirectory exists inside it.
If it does, create a symlink in {vault_path}/memory/:

Derive a readable slug from the project directory name:
- Strip leading dashes/hyphens
- Replace encoded path separators (double dashes) with a single dash
- Keep it under 40 characters
- Use the last meaningful path segment if multiple exist

```bash
[ -L {vault_path}/memory/{slug} ] \
  || ln -sf ~/.claude/projects/{project}/memory {vault_path}/memory/{slug}
```

Skip if a symlink already exists at that path.

## Step 6: Symlink Desktop HTML files

Find HTML files:
```bash
ls ~/Desktop/*.html 2>/dev/null
```

For each .html file found, symlink into {vault_path}/playgrounds/:
```bash
ln -sf ~/Desktop/{filename}.html {vault_path}/playgrounds/{filename}.html
```

Skip files whose names contain: hipaa, policy, legal, retention (case-insensitive).

## Step 7: Generate dashboard pages

Read the actual current structure and generate these markdown files in {vault_path}/. If a page already exists, overwrite it.

**Home.md** — Main index. List all symlinked directories with their file counts. Include links to each All *.md dashboard page.

**All Memory.md** — List every file in every {vault_path}/memory/*/ directory, grouped by project slug as a heading. Use [[wikilink]] format for each file.

**All Skills.md** — List everything in {vault_path}/skills/. Separate top-level items (defined skills) from skills/learned/ contents under their own heading.

**All Agents.md** — List each .md file in {vault_path}/agents/ with a one-line description (from the file's description frontmatter field, or first sentence of content if no frontmatter).

**All Rules.md** — List each .md file in {vault_path}/rules/ with a one-line summary from the first heading or opening sentence of the file.

**All Plans.md** — List the 20 most recently modified files in {vault_path}/plans/ (use ls -t), plus a note to use search for more.

**All Contexts.md** — Total count of files in {vault_path}/contexts/. A table mapping project names to search keywords. Guidance to use Obsidian search (Cmd+Shift+F) to find sessions by project.

**All Playgrounds.md** — List each file in {vault_path}/playgrounds/ with a [[wikilink]].

**All Commands.md** — List each .md file in {vault_path}/commands/ in alphabetical order as a table with the file's description frontmatter field.

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
- Vault location: {vault_path}
- Directories symlinked: list them
- Memory projects symlinked: list them
- Dashboard pages generated: list them
- "If Obsidian doesn't open automatically: File → Open Vault → select {vault_path}"
