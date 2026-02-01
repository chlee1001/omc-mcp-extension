---
name: setup
description: Install MCP behavior guides and configure environment variables
---

# omc-mcp-extension Setup

Install MCP server behavior guides and configure required environment variables.

## What This Does

1. **Backs up** existing `~/.claude/CLAUDE.md` and `~/.claude/settings.json`
2. Copies MCP guide files (`MCP_*.md`) to `~/.claude/` directory
3. Adds `@import` references to `~/.claude/CLAUDE.md` with markers
4. Adds `MORPH_API_KEY` to `~/.claude/settings.json` env (if provided)

## Execution Steps

### Step 1: Backup Existing Files

Create timestamped backups before making any changes:

```bash
CLAUDE_DIR="$HOME/.claude"
BACKUP_DIR="$CLAUDE_DIR/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Backup CLAUDE.md
if [[ -f "$CLAUDE_DIR/CLAUDE.md" ]]; then
  cp "$CLAUDE_DIR/CLAUDE.md" "$BACKUP_DIR/CLAUDE.md.backup_$TIMESTAMP"
  echo "✅ Backup: CLAUDE.md → backups/CLAUDE.md.backup_$TIMESTAMP"
fi

# Backup settings.json
if [[ -f "$CLAUDE_DIR/settings.json" ]]; then
  cp "$CLAUDE_DIR/settings.json" "$BACKUP_DIR/settings.json.backup_$TIMESTAMP"
  echo "✅ Backup: settings.json → backups/settings.json.backup_$TIMESTAMP"
fi
```

### Step 2: Copy MCP Guide Files

Copy the MCP behavior guide files from the plugin to `~/.claude/`:

```bash
PLUGIN_DIR="${CLAUDE_PLUGIN_ROOT:-$(pwd)}"
CLAUDE_DIR="$HOME/.claude"

for md_file in "$PLUGIN_DIR/mcp/MCP_"*.md; do
  if [[ -f "$md_file" ]]; then
    filename=$(basename "$md_file")
    cp "$md_file" "$CLAUDE_DIR/$filename"
    echo "✅ Copied: $filename"
  fi
done
```

### Step 3: Add @import References to CLAUDE.md

Add import references between markers:

```bash
CLAUDE_MD="$HOME/.claude/CLAUDE.md"
START_MARKER="<!-- OMC-MCP-EXT:START -->"
END_MARKER="<!-- OMC-MCP-EXT:END -->"

build_content() {
  cat << 'EOF'
<!-- OMC-MCP-EXT:START -->
# MCP Server Guides (omc-mcp-extension)

@MCP_Serena.md
@MCP_Sequential.md
@MCP_Morphllm.md
<!-- OMC-MCP-EXT:END -->
EOF
}

# Create CLAUDE.md if not exists
if [[ ! -f "$CLAUDE_MD" ]]; then
  mkdir -p "$(dirname "$CLAUDE_MD")"
  echo "# User Instructions" > "$CLAUDE_MD"
fi

# Remove existing markers if present
if grep -q "$START_MARKER" "$CLAUDE_MD"; then
  temp_file=$(mktemp)
  awk -v start="$START_MARKER" -v end="$END_MARKER" '
    $0 ~ start { skip=1; next }
    $0 ~ end { skip=0; next }
    !skip { print }
  ' "$CLAUDE_MD" > "$temp_file"
  mv "$temp_file" "$CLAUDE_MD"
fi

# Append new content
echo "" >> "$CLAUDE_MD"
build_content >> "$CLAUDE_MD"
echo "✅ Added @import references to CLAUDE.md"
```

### Step 4: Configure MORPH_API_KEY (Interactive)

Ask user for MORPH_API_KEY and add to settings.json:

**Question to ask user:**
- "Enter your MORPH_API_KEY (get it from https://morphllm.com) or press Enter to skip:"

If user provides a key, update settings.json:

```bash
SETTINGS_FILE="$HOME/.claude/settings.json"
MORPH_API_KEY="<user-provided-key>"

if [[ -n "$MORPH_API_KEY" && "$MORPH_API_KEY" != "" ]]; then
  # Use jq to add/update env.MORPH_API_KEY
  if [[ -f "$SETTINGS_FILE" ]]; then
    tmp=$(mktemp)
    jq --arg key "$MORPH_API_KEY" '.env.MORPH_API_KEY = $key' "$SETTINGS_FILE" > "$tmp"
    mv "$tmp" "$SETTINGS_FILE"
    echo "✅ Added MORPH_API_KEY to settings.json"
  else
    echo '{"env":{"MORPH_API_KEY":"'"$MORPH_API_KEY"'"}}' > "$SETTINGS_FILE"
    echo "✅ Created settings.json with MORPH_API_KEY"
  fi
else
  echo "⏭️  Skipped MORPH_API_KEY configuration"
  echo "   To add later: claude config set env.MORPH_API_KEY \"your-key\""
fi
```

### Step 5: Show Completion Message

```
✅ omc-mcp-extension Setup Complete!

BACKUPS CREATED:
  ~/.claude/backups/CLAUDE.md.backup_[timestamp]
  ~/.claude/backups/settings.json.backup_[timestamp]

MCP GUIDE FILES:
  ~/.claude/MCP_Serena.md
  ~/.claude/MCP_Sequential.md
  ~/.claude/MCP_Morphllm.md

CLAUDE.MD UPDATED:
  Added @MCP_Serena.md, @MCP_Sequential.md, @MCP_Morphllm.md

ENVIRONMENT:
  MORPH_API_KEY: [configured/skipped]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Please restart Claude Code to apply changes.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TO RESTORE:
  cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
  cp ~/.claude/backups/settings.json.backup_[timestamp] ~/.claude/settings.json

TO UPDATE: Run /omc-mcp-extension:setup again
```

## Uninstall

To completely remove omc-mcp-extension configuration:

```bash
# Remove from CLAUDE.md
sed -i '' '/<!-- OMC-MCP-EXT:START -->/,/<!-- OMC-MCP-EXT:END -->/d' ~/.claude/CLAUDE.md

# Remove MCP guide files
rm -f ~/.claude/MCP_Serena.md ~/.claude/MCP_Sequential.md ~/.claude/MCP_Morphllm.md

# Remove MORPH_API_KEY from settings.json (optional)
jq 'del(.env.MORPH_API_KEY)' ~/.claude/settings.json > tmp.json && mv tmp.json ~/.claude/settings.json
```

## Restore from Backup

```bash
# List available backups
ls -la ~/.claude/backups/

# Restore CLAUDE.md
cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md

# Restore settings.json
cp ~/.claude/backups/settings.json.backup_[timestamp] ~/.claude/settings.json
```
