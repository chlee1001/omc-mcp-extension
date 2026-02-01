---
name: setup
description: Install MCP servers and behavior guides to ~/.claude/
---

# omc-mcp-extension Setup

Complete setup for MCP servers with behavior guides.

## What This Does

1. **Backs up** existing `~/.claude/CLAUDE.md` and `~/.claude/settings.json`
2. **Updates** `~/.claude/settings.json` mcpServers (skips duplicates)
3. **Copies MCP guide files** to `~/.claude/` directory
4. **Adds `@import` references** to `~/.claude/CLAUDE.md`

## Execution Steps

### Step 1: Backup Existing Files

```bash
CLAUDE_DIR="$HOME/.claude"
BACKUP_DIR="$CLAUDE_DIR/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Backup CLAUDE.md
if [[ -f "$CLAUDE_DIR/CLAUDE.md" ]]; then
  cp "$CLAUDE_DIR/CLAUDE.md" "$BACKUP_DIR/CLAUDE.md.backup_$TIMESTAMP"
  echo "✅ Backup: CLAUDE.md"
fi

# Backup settings.json
if [[ -f "$CLAUDE_DIR/settings.json" ]]; then
  cp "$CLAUDE_DIR/settings.json" "$BACKUP_DIR/settings.json.backup_$TIMESTAMP"
  echo "✅ Backup: settings.json"
fi
```

### Step 2: Update settings.json mcpServers (Skip Duplicates)

Add MCP servers to `~/.claude/settings.json` mcpServers field. Skip if already exists.

**Ask user:** "Enter your MORPH_API_KEY (from https://morphllm.com) or press Enter to skip Morphllm:"

```bash
SETTINGS_FILE="$HOME/.claude/settings.json"
# MORPH_API_KEY from user input

# Ensure mcpServers exists
if ! jq -e '.mcpServers' "$SETTINGS_FILE" > /dev/null 2>&1; then
  jq '.mcpServers = {}' "$SETTINGS_FILE" > tmp.json && mv tmp.json "$SETTINGS_FILE"
fi

# Add context7 (skip if exists)
if ! jq -e '.mcpServers.context7' "$SETTINGS_FILE" > /dev/null 2>&1; then
  jq '.mcpServers.context7 = {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp"]
  }' "$SETTINGS_FILE" > tmp.json && mv tmp.json "$SETTINGS_FILE"
  echo "✅ Added: context7"
else
  echo "⏭️  Skipped: context7 (already exists)"
fi

# Add serena (skip if exists)
if ! jq -e '.mcpServers.serena' "$SETTINGS_FILE" > /dev/null 2>&1; then
  jq '.mcpServers.serena = {
    "command": "uvx",
    "args": ["--from", "git+https://github.com/oraios/serena", "serena", "start-mcp-server", "--context", "ide-assistant"]
  }' "$SETTINGS_FILE" > tmp.json && mv tmp.json "$SETTINGS_FILE"
  echo "✅ Added: serena"
else
  echo "⏭️  Skipped: serena (already exists)"
fi

# Add sequential-thinking (skip if exists)
if ! jq -e '.mcpServers["sequential-thinking"]' "$SETTINGS_FILE" > /dev/null 2>&1; then
  jq '.mcpServers["sequential-thinking"] = {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  }' "$SETTINGS_FILE" > tmp.json && mv tmp.json "$SETTINGS_FILE"
  echo "✅ Added: sequential-thinking"
else
  echo "⏭️  Skipped: sequential-thinking (already exists)"
fi

# Add morphllm-fast-apply (skip if exists, include API key in env)
if ! jq -e '.mcpServers["morphllm-fast-apply"]' "$SETTINGS_FILE" > /dev/null 2>&1; then
  if [[ -n "$MORPH_API_KEY" && "$MORPH_API_KEY" != "" ]]; then
    jq --arg apikey "$MORPH_API_KEY" '.mcpServers["morphllm-fast-apply"] = {
      "command": "npx",
      "args": ["@morph-llm/morph-fast-apply"],
      "env": {
        "MORPH_API_KEY": $apikey
      }
    }' "$SETTINGS_FILE" > tmp.json && mv tmp.json "$SETTINGS_FILE"
    echo "✅ Added: morphllm-fast-apply (with API key)"
  else
    echo "⏭️  Skipped: morphllm-fast-apply (no API key provided)"
    echo "   To add later manually in ~/.claude/settings.json"
  fi
else
  echo "⏭️  Skipped: morphllm-fast-apply (already exists)"
fi
```

### Step 3: Copy MCP Guide Files

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

### Step 4: Add @import References to CLAUDE.md

```bash
CLAUDE_MD="$HOME/.claude/CLAUDE.md"
START_MARKER="<!-- OMC-MCP-EXT:START -->"
END_MARKER="<!-- OMC-MCP-EXT:END -->"

build_content() {
  cat << 'EOF'
<!-- OMC-MCP-EXT:START -->
# MCP Server Guides (omc-mcp-extension)

@MCP_Context7.md
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

### Step 5: Completion Message

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ omc-mcp-extension Setup Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BACKUPS:
  ~/.claude/backups/CLAUDE.md.backup_[timestamp]
  ~/.claude/backups/settings.json.backup_[timestamp]

MCP SERVERS (in ~/.claude/settings.json → mcpServers):
  • context7 (official library documentation)
  • serena (semantic code analysis + memory)
  • sequential-thinking (structured reasoning)
  • morphllm-fast-apply (bulk code editing) - if API key provided

MCP GUIDE FILES:
  ~/.claude/MCP_Context7.md
  ~/.claude/MCP_Serena.md
  ~/.claude/MCP_Sequential.md
  ~/.claude/MCP_Morphllm.md

CLAUDE.MD IMPORTS:
  @MCP_Context7.md
  @MCP_Serena.md
  @MCP_Sequential.md
  @MCP_Morphllm.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  Restart Claude Code to apply MCP changes!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ADD MORPHLLM LATER (if skipped):
  Edit ~/.claude/settings.json and add to mcpServers:
  "morphllm-fast-apply": {
    "command": "npx",
    "args": ["@morph-llm/morph-fast-apply"],
    "env": { "MORPH_API_KEY": "your-key-here" }
  }

RESTORE:
  cp ~/.claude/backups/settings.json.backup_[timestamp] ~/.claude/settings.json
  cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
```

## Uninstall

```bash
# Remove MCP servers from settings.json
jq 'del(.mcpServers.context7, .mcpServers.serena, .mcpServers["sequential-thinking"], .mcpServers["morphllm-fast-apply"])' \
  ~/.claude/settings.json > tmp.json && mv tmp.json ~/.claude/settings.json

# Remove from CLAUDE.md
sed -i '' '/<!-- OMC-MCP-EXT:START -->/,/<!-- OMC-MCP-EXT:END -->/d' ~/.claude/CLAUDE.md

# Remove MCP guide files
rm -f ~/.claude/MCP_Context7.md ~/.claude/MCP_Serena.md ~/.claude/MCP_Sequential.md ~/.claude/MCP_Morphllm.md
```

## Restore from Backup

```bash
# List backups
ls -la ~/.claude/backups/

# Restore
cp ~/.claude/backups/settings.json.backup_[timestamp] ~/.claude/settings.json
cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
```
