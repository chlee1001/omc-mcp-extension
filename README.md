# omc-mcp-extension

> MCP server integration for oh-my-claudecode

A lightweight extension plugin that adds [SuperClaude](https://github.com/SuperClaude-Org/SuperClaude_Framework)'s MCP integration capabilities to [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)'s powerful skill/hook/agent system.

---

## Features

| MCP Server | Purpose | Required |
|------------|---------|----------|
| **Context7** | Official library documentation lookup (React, Vue, Next.js, etc.) | npx |
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
```

### Install

```bash
# 1. Install OMC first (skip if already installed)
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode

# 2. Install this extension plugin
/plugin marketplace add https://github.com/chlee1001/omc-mcp-extension
/plugin install omc-mcp-extension
```

### Post-Installation (Required for Morphllm)

To use Morphllm, you **must** set the environment variable:

```bash
# Add to ~/.zshrc or ~/.bashrc
echo 'export MORPH_API_KEY="your-api-key-here"' >> ~/.zshrc

# Restart terminal or source the file
source ~/.zshrc
```

> **Warning**: Without the API key, the Morphllm MCP server will not function. If you don't need Morphllm, you can skip this step.

---

## How It Works

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code Runtime                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  MCP Servers                                                 │
│  ├── "t" (OMC Bridge) - OMC's own tools                     │
│  │   └─ mcp__t__lsp_*, mcp__t__ast_* (18 tools)             │
│  │                                                          │
│  └── External MCP (omc-mcp-extension)                       │
│      ├─ omc-mcp-extension:context7      → Official docs     │
│      ├─ omc-mcp-extension:serena        → Semantic analysis │
│      ├─ omc-mcp-extension:sequential-thinking → Reasoning   │
│      └─ omc-mcp-extension:morphllm-fast-apply → Bulk edit   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

> **Note**: Server names follow the `{plugin-name}:{server-name}` pattern as per OMC's discovery system.

### Workflow Example

```
User: "autopilot: build a React dashboard"
         │
         ▼
OMC Hook: keyword-detector.mjs
  → Detects "autopilot" → Injects skill
         │
         ▼
MCP Guide: MCP_Context7.md
  → Detects "React" keyword
  → Calls mcp__context7__query-docs
  → Retrieves official React patterns
         │
         ▼
OMC Agents + MCP Tools Collaboration
  Analyst  → mcp__t__lsp_diagnostics (code analysis)
  Architect → mcp__sequential-thinking (design)
  Executor → mcp__serena__find_symbol (navigation)
           → mcp__morphllm__edit_file (editing)
```

---

## MCP Selection Guide

Claude automatically selects the appropriate MCP based on these guides:

| When you need... | Use | Example |
|------------------|-----|---------|
| Official library docs | **Context7** | "How to use React useEffect" |
| Symbol rename/find refs | **Serena** | "Rename getUserData to fetchUser" |
| Complex multi-step analysis | **Sequential** | "Why is this API slow?" |
| Pattern-based bulk edits | **Morphllm** | "Convert all var to const" |
| Simple tasks | **Native Claude** | "Explain this function" |

### Collaboration Patterns

| Scenario | Workflow |
|----------|----------|
| Build React app | Context7(docs) → Sequential(design) → Serena(implement) |
| Refactor legacy code | Serena(analyze) → Sequential(plan) → Morphllm(transform) |
| Debug issues | Sequential(analyze) → Serena(navigate) → Context7(solution) |
| Unify code style | Morphllm(bulk transform) |

---

## MCP Behavior Guides

Detailed usage guides for each MCP server are in the `mcp/` directory:

- `mcp/MCP_Context7.md` - Official documentation lookup triggers and usage
- `mcp/MCP_Serena.md` - Semantic analysis and memory usage
- `mcp/MCP_Sequential.md` - Structured reasoning usage
- `mcp/MCP_Morphllm.md` - Bulk editing usage

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
| OMC | `mcp__t__*` | LSP, AST, Python REPL (18 tools) |
| Extension | `mcp__context7__*`, `mcp__serena__*`, etc. | External MCP (4 servers) |

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

### Context7 is slow
- First invocation may be slow due to npx download
- Subsequent calls are cached and faster

---

## Credits

- MCP configurations adapted from [SuperClaude Framework](https://github.com/SuperClaude-Org/SuperClaude_Framework)
- Built to extend [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)

---

## License

MIT
