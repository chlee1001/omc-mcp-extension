# Context7 MCP Server

**Purpose**: Official library documentation lookup and framework pattern guidance

## Triggers
- Import statements: `import`, `require`, `from`, `use`
- Framework keywords: React, Vue, Angular, Next.js, Express, etc.
- Library-specific questions about APIs or best practices
- Need for official documentation patterns vs generic solutions
- Version-specific implementation requirements

## Tools

| Tool | Description |
|------|-------------|
| `resolve-library-id` | Find Context7-compatible library ID from package name |
| `query-docs` | Query documentation with specific questions |

## Usage Flow

```
1. resolve-library-id("react") → "/facebook/react"
2. query-docs("/facebook/react", "useEffect cleanup") → Documentation
```

## Choose When
- **Over WebSearch**: When you need curated, version-specific documentation
- **Over native knowledge**: When implementation must follow official patterns
- **For frameworks**: React hooks, Vue composition API, Angular services
- **For libraries**: Correct API usage, authentication flows, configuration
- **For compliance**: When adherence to official standards is mandatory

## Works Best With
- **Sequential**: Context7 provides docs → Sequential analyzes implementation strategy
- **Serena**: Context7 supplies patterns → Serena implements with semantic understanding

## Examples
```
"implement React useEffect" → Context7 (official React patterns)
"add authentication with Auth0" → Context7 (official Auth0 docs)
"migrate to Vue 3" → Context7 (official migration guide)
"optimize Next.js performance" → Context7 (official optimization patterns)
"just explain this function" → Native Claude (no external docs needed)
```

## Best Practices

1. **Always resolve library ID first** before querying docs
2. **Be specific in queries** - "useEffect cleanup function" not just "useEffect"
3. **Check version compatibility** - ensure docs match your project version
4. **Combine with Sequential** for complex implementation strategies
