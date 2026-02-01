# Context7 MCP Server

**Purpose**: Official library documentation lookup and framework pattern guidance

## Triggers

Claude should use Context7 when:

- Import statements detected: `import`, `require`, `from`, `use`
- Framework keywords: React, Vue, Angular, Next.js, Express, Svelte, Django, FastAPI, etc.
- Library-specific questions about APIs or best practices
- Version-specific implementation requirements (e.g., "in React 18...")
- Need for official documentation patterns vs generic solutions

## Choose When

| Scenario | Use Context7 | Alternative |
|----------|-------------|-------------|
| Official documentation patterns needed | ✅ | - |
| Version-specific API verification | ✅ | - |
| Framework best practices | ✅ | - |
| Simple concept explanation | ❌ | Native Claude |
| Code structure analysis | ❌ | Serena |
| Complex multi-step reasoning | ❌ | Sequential |

### Decision Criteria

- **Over WebSearch**: When you need curated, version-specific documentation
- **Over native knowledge**: When implementation must follow official patterns
- **For frameworks**: React hooks, Vue composition API, Angular services
- **For libraries**: Correct API usage, authentication flows, configuration
- **For compliance**: When adherence to official standards is mandatory

## Works Best With

```
Context7 + Sequential
├── Context7: Retrieves official documentation patterns
└── Sequential: Analyzes implementation strategy using those patterns

Context7 + Serena
├── Context7: Provides framework-specific patterns
└── Serena: Applies patterns to current project context

Context7 + Morphllm
├── Context7: Identifies migration patterns (e.g., Vue 2 → Vue 3)
└── Morphllm: Executes bulk transformations following patterns
```

## Available Tools

- `mcp__context7__resolve-library-id`: Convert library name to Context7 ID
- `mcp__context7__query-docs`: Query official documentation for the library

## Examples

```
"implement React useEffect" → Context7 (official React patterns)
"add authentication with Auth0" → Context7 (official Auth0 docs)
"migrate to Vue 3" → Context7 (official migration guide)
"optimize Next.js performance" → Context7 (official optimization patterns)
"just explain this function" → Native Claude (no external docs needed)
```

### Detailed Usage

```
User: "How do I use React useEffect correctly?"

→ Step 1: mcp__context7__resolve-library-id({
    libraryName: "react",
    query: "useEffect hook usage and cleanup"
  })

→ Step 2: mcp__context7__query-docs({
    libraryId: "/vercel/react",
    query: "useEffect hook usage cleanup dependencies"
  })
```

## When NOT to Use

- Simple function explanations → Native Claude
- Project-specific code analysis → Serena
- Complex debugging with multiple components → Sequential
- Bulk code transformations → Morphllm
- Single-file edits → Native Edit tool
