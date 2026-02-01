# Serena MCP Server

**Purpose**: Semantic code understanding with project memory and session persistence

## Triggers

Claude should use Serena when:

- Symbol operations: rename, extract, move functions/classes
- Project-wide code navigation and exploration
- Multi-language projects requiring LSP integration
- Session lifecycle management (project activation, memory save/load)
- Large codebase analysis (>50 files, complex architecture)
- Finding symbol references and tracking dependencies

## Choose When

| Scenario | Use Serena | Alternative |
|----------|-----------|-------------|
| Rename function across codebase | ✅ | - |
| Find all symbol references | ✅ | - |
| Save/load project memory | ✅ | - |
| Understand code structure | ✅ | - |
| Pattern-based text replacement | ❌ | Morphllm |
| Official documentation lookup | ❌ | Context7 |
| Complex reasoning | ❌ | Sequential |

### Decision Criteria

- **Over Morphllm**: For symbol operations, not pattern-based edits
- **For semantic understanding**: Symbol references, dependency tracking, LSP integration
- **For session persistence**: Project context, memory management, cross-session learning
- **For large projects**: Multi-language codebases requiring architectural understanding
- **Not for simple edits**: Basic text replacements, style enforcement, bulk operations

## Works Best With

```
Serena + Morphllm
├── Serena: Analyzes semantic context (what needs to change)
└── Morphllm: Executes precise edits (actual code modifications)

Serena + Sequential
├── Serena: Provides project structure and dependency information
└── Sequential: Performs architectural analysis and refactoring strategy

Serena + Context7
├── Context7: Provides framework patterns
└── Serena: Applies patterns within project context
```

## Available Tools

### Navigation
- `mcp__serena__find_symbol`: Find symbols (functions, classes, variables)
- `mcp__serena__find_referencing_symbols`: Find symbol references
- `mcp__serena__get_symbols_overview`: Get file symbols overview
- `mcp__serena__search_for_pattern`: Search for patterns in codebase

### Editing
- `mcp__serena__replace_symbol_body`: Replace symbol body
- `mcp__serena__insert_after_symbol`: Insert after symbol
- `mcp__serena__insert_before_symbol`: Insert before symbol
- `mcp__serena__rename_symbol`: Rename symbol across entire codebase

### Memory
- `mcp__serena__write_memory`: Save project information
- `mcp__serena__read_memory`: Read saved information
- `mcp__serena__list_memories`: List saved memories

### Project Management
- `mcp__serena__activate_project`: Activate project
- `mcp__serena__list_dir`: List directory
- `mcp__serena__find_file`: Find files

## Examples

```
"rename getUserData function everywhere" → Serena (symbol operation with dependency tracking)
"find all references to this class" → Serena (semantic search and navigation)
"load my project context" → Serena (project activation)
"save my current work session" → Serena (memory persistence)
"update all console.log to logger" → Morphllm (pattern-based replacement)
"create a login form" → Native Claude + Write tool
```

### Detailed Usage

```
User: "Rename getUserData to fetchUserData everywhere"

→ Step 1: mcp__serena__find_symbol({
    name_path_pattern: "getUserData",
    include_body: false
  })

→ Step 2: mcp__serena__rename_symbol({
    name_path: "getUserData",
    relative_path: "src/api/user.ts",
    new_name: "fetchUserData"
  })
```

## When NOT to Use

- Simple text search/replace → Morphllm
- Official documentation lookup → Context7
- Complex multi-step reasoning → Sequential
- Creating new files → Native Write tool
- Single-line edits → Native Edit tool
