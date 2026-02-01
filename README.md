# omc-mcp-extension

> MCP server extension for oh-my-claudecode

A lightweight extension plugin that adds additional MCP servers to [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)'s powerful skill/hook/agent system.

---

## Features

| MCP Server | Purpose | Required |
|------------|---------|----------|
| **Context7** | Official library documentation lookup | npx |
| **Serena** | Semantic code analysis + session memory persistence | uvx |
| **Sequential** | Structured multi-step reasoning (30-50% token savings) | npx |
| **Morphllm** | Pattern-based bulk code editing | npx + API Key |

---

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version   # v18.0.0 or higher

# uvx required (for Serena)
pip install uv   # or: pipx install uv
which uvx        # verify installation

# jq required (for setup skill)
brew install jq  # macOS
```

### Install

```bash
# 1. Install OMC first (skip if already installed)
claude plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
claude plugin install oh-my-claudecode@omc

# 2. Install this extension plugin
claude plugin marketplace add https://github.com/chlee1001/omc-mcp-extension
claude plugin install omc-mcp-extension@omc-mcp-extension
```

### Post-Installation Setup (Required)

Run the setup skill to complete installation:

```
/omc-mcp-extension:setup
```

This will:
- ✅ Backup your existing files
- ✅ Create/Update `~/.claude/.mcp.json` with MCP servers
- ✅ Copy MCP behavior guides to `~/.claude/`
- ✅ Add `@import` references to `~/.claude/CLAUDE.md`
- ✅ Configure `MORPH_API_KEY` (optional, for Morphllm)

> ⚠️ **Restart Claude Code after setup to apply MCP changes!**

---

## Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| **setup** | Complete MCP setup with backups | `/omc-mcp-extension:setup` |

### What Setup Does

```
~/.claude/
├── .mcp.json              # MCP server configurations (created/updated)
├── CLAUDE.md              # Updated with @import references
├── MCP_Context7.md        # Behavior guide (copied)
├── MCP_Serena.md          # Behavior guide (copied)
├── MCP_Sequential.md      # Behavior guide (copied)
├── MCP_Morphllm.md        # Behavior guide (copied)
└── backups/
    ├── .mcp.json.backup_[timestamp]
    ├── CLAUDE.md.backup_[timestamp]
    └── settings.json.backup_[timestamp]
```

**~/.claude/.mcp.json:**
```json
{
  "mcpServers": {
    "context7": { "command": "npx", "args": ["-y", "@upstash/context7-mcp"] },
    "serena": { "command": "uvx", "args": [...] },
    "sequential-thinking": { "command": "npx", "args": [...] },
    "morphllm-fast-apply": { "command": "npx", "args": [...], "env": {...} }
  }
}
```

**CLAUDE.md additions:**
```markdown
<!-- OMC-MCP-EXT:START -->
# MCP Server Guides (omc-mcp-extension)

@MCP_Context7.md
@MCP_Serena.md
@MCP_Sequential.md
@MCP_Morphllm.md
<!-- OMC-MCP-EXT:END -->
```

---

## How It Works

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code Runtime                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  MCP Servers (in ~/.claude/settings.json)                   │
│  ├── OMC Built-in                                           │
│  │   ├─ "t" (OMC Bridge) - LSP, AST tools (18 tools)       │
│  │   └─ context7 - Official docs lookup (built-in)         │
│  │                                                          │
│  └── omc-mcp-extension (added by setup skill)              │
│      ├─ serena             → Semantic analysis + memory     │
│      ├─ sequential-thinking → Structured reasoning          │
│      └─ morphllm-fast-apply → Bulk editing                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Workflow Example

```
User: "autopilot: refactor the authentication module"
         │
         ▼
OMC Hook: keyword-detector
  → Detects "autopilot" → Injects skill
         │
         ▼
OMC Agents + MCP Tools Collaboration
  Analyst  → mcp__t__lsp_diagnostics (code analysis)
  Architect → mcp__sequential-thinking__* (structured reasoning)
  Executor → mcp__serena__find_symbol (navigation)
           → mcp__morphllm-fast-apply__edit_file (bulk editing)
```

---

## MCP Selection Guide

| When you need... | Use | Example |
|------------------|-----|---------|
| Official library docs | **Context7** (OMC built-in) | "How to use React useEffect" |
| Symbol rename/find refs | **Serena** | "Rename getUserData to fetchUser" |
| Complex multi-step analysis | **Sequential** | "Why is this API slow?" |
| Pattern-based bulk edits | **Morphllm** | "Convert all var to const" |
| Simple tasks | **Native Claude** | "Explain this function" |

### Collaboration Patterns

| Scenario | Workflow |
|----------|----------|
| Build React app | Context7(docs) → Sequential(design) → Serena(implement) |
| Refactor legacy code | Serena(analyze) → Sequential(plan) → Morphllm(transform) |
| Debug issues | Sequential(analyze) → Serena(navigate) |
| Unify code style | Morphllm(bulk transform) |

---

## MCP Behavior Guides

Detailed usage guides for each MCP server:

| File | Description |
|------|-------------|
| `MCP_Serena.md` | Semantic analysis, symbol operations, session memory |
| `MCP_Sequential.md` | Structured reasoning, multi-step analysis |
| `MCP_Morphllm.md` | Bulk editing, pattern-based transformations |

These are automatically copied to `~/.claude/` and imported via `@` references when you run `/omc-mcp-extension:setup`.

---

## Compatibility

| Component | Version |
|-----------|---------|
| **oh-my-claudecode** | 3.x+ |
| **Claude Code** | Latest |
| **Node.js** | 18+ |
| **Python** | 3.10+ (for Serena) |
| **jq** | Latest (for setup) |

### Namespace Separation (No Conflicts)

| Source | Namespace | Tools |
|--------|-----------|-------|
| OMC | `mcp__t__*`, `mcp__context7__*` | LSP, AST, Context7 |
| Extension | `mcp__serena__*`, `mcp__sequential-thinking__*`, `mcp__morphllm-fast-apply__*` | 3 servers |

---

## Troubleshooting

### MCP servers not showing

```bash
# Verify settings.json has mcpServers
cat ~/.claude/settings.json | jq '.mcpServers | keys'

# Expected: ["serena", "sequential-thinking", "morphllm-fast-apply", ...]

# If missing, re-run setup
/omc-mcp-extension:setup
```

### Morphllm not working

```bash
# Check API key in settings.json
cat ~/.claude/settings.json | jq '.env.MORPH_API_KEY'

# If null, add it manually
claude config set env.MORPH_API_KEY "your-key-here"
```

### Serena not starting

```bash
# Verify uvx installation
which uvx

# If not found, install
pip install uv
```

### Restore from backup

```bash
# List backups
ls -la ~/.claude/backups/

# Restore settings.json
cp ~/.claude/backups/settings.json.backup_[timestamp] ~/.claude/settings.json

# Restore CLAUDE.md
cp ~/.claude/backups/CLAUDE.md.backup_[timestamp] ~/.claude/CLAUDE.md
```

---

## Uninstall

### Remove MCP configuration

```bash
# Remove MCP servers from settings.json
jq 'del(.mcpServers.serena, .mcpServers["sequential-thinking"], .mcpServers["morphllm-fast-apply"])' \
  ~/.claude/settings.json > tmp.json && mv tmp.json ~/.claude/settings.json

# Remove MORPH_API_KEY
jq 'del(.env.MORPH_API_KEY)' ~/.claude/settings.json > tmp.json && mv tmp.json ~/.claude/settings.json

# Remove from CLAUDE.md
sed -i '' '/<!-- OMC-MCP-EXT:START -->/,/<!-- OMC-MCP-EXT:END -->/d' ~/.claude/CLAUDE.md

# Remove MCP guide files
rm -f ~/.claude/MCP_Serena.md ~/.claude/MCP_Sequential.md ~/.claude/MCP_Morphllm.md
```

### Remove plugin

```bash
claude plugin uninstall omc-mcp-extension@omc-mcp-extension
claude plugin marketplace remove omc-mcp-extension
```

---

## Credits

- Built to extend [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)
- MCP Servers: [Serena](https://github.com/oraios/serena), [Sequential-Thinking](https://github.com/modelcontextprotocol/servers), [Morphllm](https://morphllm.com)

---

## License

MIT