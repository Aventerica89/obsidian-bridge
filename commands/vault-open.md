---
description: Open your Obsidian vault. Reads vault path from ~/.claude/obsidian-config.json.
allowed-tools: [Bash, Read]
---

# Vault Open

Open the obsidian-bridge vault in Obsidian.

## Step 1: Read vault path

Read `~/.claude/obsidian-config.json`.
Extract `vault_path`. Expand `~` to the actual home directory.
Default to `~/Obsidian-Claude` if not set.

## Step 2: Verify vault exists

```bash
ls {vault_path} 2>/dev/null
```

If the directory does not exist, tell the user:
"Vault not found at `{vault_path}`. Run `/vault-init` to set it up."

## Step 3: Open in Obsidian

```bash
open -a Obsidian {vault_path}
```

If Obsidian is not installed, tell the user:
"Obsidian is not installed. Run `/vault-init` which will offer to install it."
