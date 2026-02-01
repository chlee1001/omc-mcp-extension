# omc-mcp-extension

> MCP server extension for oh-my-claudecode

A lightweight extension plugin that adds additional MCP servers to [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)'s powerful skill/hook/agent system.

---

## Features

| MCP Server | Purpose | Required |
|------------|---------|----------|
| **Serena** | Semantic code analysis + session memory persistence | uvx |
| **Sequential** | Structured multi-step reasoning (30-50% token savings) | npx |
| **Morphllm** | Pattern-based bulk code editing | npx + API Key |

> **Note**: Context7 is already built into OMC, so it's not included here.

---

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version   # v18.0.0 or higher

# uvx required (for Serena)
pip install uv   # or: pipx install uv
which uvx        # verify installation
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

### Post-Installation Setup (Recommended)

After installation, run the setup skill to add MCP behavior guides to your CLAUDE.md:

```
/omc-mcp-extension:setup
```

This will add behavior guides that help Claude automatically select the right MCP tool for each task.

### Morphllm API Key (Required for Morphllm)

To use Morphllm, you **must** set the environment variable:

```bash
# Add to ~/.zshrc or ~/.bashrc
echo 'export MORPH_API_KEY="your-api-key-here"' >> ~/.zshrc

# Restart terminal or source the file
source ~/.zshrc
```

> **Warning**: Without the API key, the Morphllm MCP server will not function. If you don't need Morphllm, you can skip this step.

---

## Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| **setup** | Install MCP behavior guides to CLAUDE.md | `/omc-mcp-extension:setup` |

### Setup Skill

The setup skill adds MCP behavior guides to your `~/.claude/CLAUDE.md`:

```markdown
<!-- OMC-MCP-EXT:START -->
# MCP Server Behavior Guides
... guides for Serena, Sequential, Morphllm ...
<!-- OMC-MCP-EXT:END -->
```

**Features:**
- Preserves existing CLAUDE.md content
- Uses markers for easy update/removal
- Run again to update guides

**To remove:** Delete content between `<!-- OMC-MCP-EXT:START -->` and `<!-- OMC-MCP-EXT:END -->` markers.

---

## How It Works

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code Runtime                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  MCP Servers                                                 │
│  ├── OMC Built-in                                           │
│  │   ├─ "t" (OMC Bridge) - LSP, AST tools (18 tools)       │
│  │   └─ context7 - Official docs lookup (built-in)         │
│  │                                                          │
│  └── omc-mcp-extension (this plugin)                        │
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

Detailed usage guides for each MCP server are in the `mcp/` directory:

- `mcp/MCP_Serena.md` - Semantic analysis and memory usage
- `mcp/MCP_Sequential.md` - Structured reasoning usage
- `mcp/MCP_Morphllm.md` - Bulk editing usage

These guides are installed to your CLAUDE.md when you run `/omc-mcp-extension:setup`.

---

## Compatibility

| Component | Version |
|-----------|---------|
| **oh-my-claudecode** | 3.x+ |
| **Claude Code** | Latest |
| **Node.js** | 18+ |
| **Python** | 3.10+ (for Serena) |

### Namespace Separation (No Conflicts)

| Source | Namespace | Tools |
|--------|-----------|-------|
| OMC | `mcp__t__*`, `mcp__context7__*` | LSP, AST, Context7 |
| Extension | `mcp__serena__*`, `mcp__sequential-thinking__*`, `mcp__morphllm-fast-apply__*` | 3 servers |

---

## Troubleshooting

### Morphllm not working
```bash
# Check API key
echo $MORPH_API_KEY

# If not set, add and restart terminal
export MORPH_API_KEY="your-key"
```

### Serena not starting
```bash
# Verify uvx installation
which uvx

# If not found, install
pip install uv
```

### Behavior guides not working
```bash
# Re-run setup
/omc-mcp-extension:setup

# Or manually check ~/.claude/CLAUDE.md for markers
grep "OMC-MCP-EXT" ~/.claude/CLAUDE.md
```

---

## Credits

- Built to extend [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)

---

## License

MIT
