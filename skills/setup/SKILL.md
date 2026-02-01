---
name: setup
description: Install MCP servers and behavior guides to ~/.claude/
---

# omc-mcp-extension Setup

Complete setup for MCP servers with behavior guides.

## What This Does

1. **Backs up** existing `~/.claude/CLAUDE.md` and `~/.claude/.mcp.json`
2. **Creates/Updates** `~/.claude/.mcp.json` with MCP servers (Context7, Serena, Sequential, Morphllm)
3. **Copies MCP guide files** to `~/.claude/` directory
4. **Adds `@import` references** to `~/.claude/CLAUDE.md`
5. **Configures `MORPH_API_KEY`** in `~/.claude/settings.json` env (if provided)

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

# Backup .mcp.json
if [[ -f "$CLAUDE_DIR/.mcp.json" ]]; then
  cp "$CLAUDE_DIR/.mcp.json" "$BACKUP_DIR/.mcp.json.backup_$TIMESTAMP"
  echo "✅ Backup: .mcp.json"
fi

# Backup settings.json
if [[ -f "$CLAUDE_DIR/settings.json" ]]; then
  cp "$CLAUDE_DIR/settings.json" "$BACKUP_DIR/settings.json.backup_$TIMESTAMP"
  echo "✅ Backup: settings.json"
fi
```

### Step 2: Create/Update ~/.claude/.mcp.json

Create or merge MCP servers into `~/.claude/.mcp.json`:

```bash
MCP_FILE="$HOME/.claude/.mcp.json"

# Define MCP servers to add
read -r -d '' NEW_SERVERS << 'EOF'
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@upstash/context7-mcp"]
  },
  "serena": {
    "command": "uvx",
    "args": ["--from", "git+https://github.com/oraios/serena", "serena", "start-mcp-server", "--context", "ide-assistant"]
  },
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  },
  "morphllm-fast-apply": {
    "command": "npx",
    "args": ["@morph-llm/morph-fast-apply", "$HOME"],
    "env": {
      "MORPH_API_KEY": "${MORPH_API_KEY}",
      "ALL_TOOLS": "true"
    }
  }
}
EOF

if [[ -f "$MCP_FILE" ]]; then
  # Merge with existing .mcp.json
  tmp=$(mktemp)
  jq --argjson new "$NEW_SERVERS" '.mcpServers += $new' "$MCP_FILE" > "$tmp"
  mv "$tmp" "$MCP_FILE"
  echo "✅ Updated existing .mcp.json"
else
  # Create new .mcp.json
  echo "{\"mcpServers\": $NEW_SERVERS}" | jq '.' > "$MCP_FILE"
  echo "✅ Created new .mcp.json"
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

### Step 5: Configure MORPH_API_KEY (Interactive)

**Ask user:** "Enter your MORPH_API_KEY (from https://morphllm.com) or press Enter to skip:"

```bash
SETTINGS_FILE="$HOME/.claude/settings.json"

if [[ -n "$MORPH_API_KEY" && "$MORPH_API_KEY" != "" ]]; then
  jq --arg key "$MORPH_API_KEY" '.env.MORPH_API_KEY = $key' "$SETTINGS_FILE" > tmp.json && mv tmp.json "$SETTINGS_FILE"
  echo "✅ Added MORPH_API_KEY to settings.json"
else
  echo "⏭️  Skipped MORPH_API_KEY (add later: claude config set env.MORPH_API_KEY \"your-key\")"
fi
```

### Step 6: Completion Message

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ omc-mcp-extension Setup Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BACKUPS:
  ~/.claude/backups/CLAUDE.md.backup_[timestamp]
  ~/.claude/backups/.mcp.json.backup_[timestamp]
  ~/.claude/backups/settings.json.backup_[timestamp]

MCP SERVERS (~/.claude/.mcp.json):
  • context7 (official library documentation)
  • serena (semantic code analysis + memory)
  • sequential-thinking (structured reasoning)
  • morphllm-fast-apply (bulk code editing)

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

RESTORE:
  cp ~/.claude/backups/.mcp.json.backup_[timestamp] ~/.claude/.mcp.json
  cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
```

## Uninstall

```bash
# Remove MCP servers from .mcp.json
jq 'del(.mcpServers.context7, .mcpServers.serena, .mcpServers["sequential-thinking"], .mcpServers["morphllm-fast-apply"])' \
  ~/.claude/.mcp.json > tmp.json && mv tmp.json ~/.claude/.mcp.json

# Remove MORPH_API_KEY from env
jq 'del(.env.MORPH_API_KEY)' ~/.claude/settings.json > tmp.json && mv tmp.json ~/.claude/settings.json

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
cp ~/.claude/backups/.mcp.json.backup_[timestamp] ~/.claude/.mcp.json
cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
```
