---
name: setup
description: Install MCP behavior guides to ~/.claude/ using @import references
---

# omc-mcp-extension Setup

Install MCP server behavior guides using `@import` references for clean CLAUDE.md organization.

## What This Does

1. **Backs up** existing `~/.claude/CLAUDE.md` before any changes
2. Copies MCP guide files (`MCP_*.md`) to `~/.claude/` directory
3. Adds `@import` references to `~/.claude/CLAUDE.md` with markers
4. Preserves existing CLAUDE.md content

## Execution Steps

### Step 1: Backup Existing CLAUDE.md

Create a timestamped backup before making any changes:

```bash
CLAUDE_MD="$HOME/.claude/CLAUDE.md"
BACKUP_DIR="$HOME/.claude/backups"

if [[ -f "$CLAUDE_MD" ]]; then
  mkdir -p "$BACKUP_DIR"
  TIMESTAMP=$(date +%Y%m%d_%H%M%S)
  BACKUP_FILE="$BACKUP_DIR/CLAUDE.md.backup_$TIMESTAMP"
  cp "$CLAUDE_MD" "$BACKUP_FILE"
  echo "✅ Backup created: $BACKUP_FILE"
fi
```

### Step 2: Copy MCP Guide Files

Copy the MCP behavior guide files from the plugin to `~/.claude/`:

```bash
PLUGIN_DIR="${CLAUDE_PLUGIN_ROOT:-$(pwd)}"
CLAUDE_DIR="$HOME/.claude"

# Copy each MCP guide file
for md_file in "$PLUGIN_DIR/mcp/MCP_"*.md; do
  if [[ -f "$md_file" ]]; then
    filename=$(basename "$md_file")
    cp "$md_file" "$CLAUDE_DIR/$filename"
    echo "Copied: $filename"
  fi
done
```

### Step 3: Add @import References to CLAUDE.md

Add import references between markers:

```markdown
<!-- OMC-MCP-EXT:START -->
# MCP Server Guides (omc-mcp-extension)

@MCP_Serena.md
@MCP_Sequential.md
@MCP_Morphllm.md
<!-- OMC-MCP-EXT:END -->
```

### Step 4: Update CLAUDE.md

**If markers exist:** Replace content between markers
**If no markers:** Append to end of file

```bash
#!/bin/bash
CLAUDE_MD="$HOME/.claude/CLAUDE.md"
START_MARKER="<!-- OMC-MCP-EXT:START -->"
END_MARKER="<!-- OMC-MCP-EXT:END -->"

# Build import references
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
  echo "" >> "$CLAUDE_MD"
fi

# Check if markers exist
if grep -q "$START_MARKER" "$CLAUDE_MD"; then
  # Replace existing content between markers
  temp_file=$(mktemp)
  awk -v start="$START_MARKER" -v end="$END_MARKER" '
    $0 ~ start { skip=1; next }
    $0 ~ end { skip=0; next }
    !skip { print }
  ' "$CLAUDE_MD" > "$temp_file"

  # Find insertion point and add new content
  mv "$temp_file" "$CLAUDE_MD"
  echo "" >> "$CLAUDE_MD"
  build_content >> "$CLAUDE_MD"
  echo "✅ Updated MCP guide references in CLAUDE.md"
else
  # Append to end
  echo "" >> "$CLAUDE_MD"
  build_content >> "$CLAUDE_MD"
  echo "✅ Added MCP guide references to CLAUDE.md"
fi
```

### Step 5: Show Completion Message

```
✅ MCP Behavior Guides Installed!

BACKUP CREATED:
~/.claude/backups/CLAUDE.md.backup_[timestamp]

FILES COPIED TO ~/.claude/:
- MCP_Serena.md
- MCP_Sequential.md
- MCP_Morphllm.md

REFERENCES ADDED TO ~/.claude/CLAUDE.md:
@MCP_Serena.md
@MCP_Sequential.md
@MCP_Morphllm.md

TO RESTORE: cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
TO UPDATE: Run /omc-mcp-extension:setup again
TO REMOVE:
  1. Delete markers section from CLAUDE.md
  2. Remove MCP_*.md files from ~/.claude/
```

## Uninstall

To remove the MCP guides completely:

```bash
# Remove references from CLAUDE.md
sed -i '' '/<!-- OMC-MCP-EXT:START -->/,/<!-- OMC-MCP-EXT:END -->/d' ~/.claude/CLAUDE.md

# Remove copied files
rm -f ~/.claude/MCP_Serena.md ~/.claude/MCP_Sequential.md ~/.claude/MCP_Morphllm.md
```

## Restore from Backup

To restore your previous CLAUDE.md:

```bash
# List available backups
ls -la ~/.claude/backups/

# Restore a specific backup
cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
```
