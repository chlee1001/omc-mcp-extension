# Sequential Thinking MCP Server

**Purpose**: Multi-step reasoning engine for complex analysis and systematic problem solving

## Triggers

Claude should use Sequential when:

- Complex debugging scenarios with multiple layers
- Architectural analysis and system design questions
- Problems requiring hypothesis testing and validation
- Multi-component failure investigation
- Performance bottleneck identification requiring methodical approach
- Problems with 3+ interconnected components

## Choose When

| Scenario | Use Sequential | Alternative |
|----------|---------------|-------------|
| "Why is this API slow?" | ✅ | - |
| Microservices architecture design | ✅ | - |
| Complex bug debugging | ✅ | - |
| Security vulnerability analysis | ✅ | - |
| Simple function explanation | ❌ | Native Claude |
| Code editing | ❌ | Morphllm/Serena |
| Documentation lookup | ❌ | Context7 |

### Decision Criteria

- **Over native reasoning**: When problems have 3+ interconnected components
- **For systematic analysis**: Root cause analysis, architecture review, security assessment
- **When structure matters**: Problems benefit from decomposition and evidence gathering
- **For cross-domain issues**: Problems spanning frontend, backend, database, infrastructure
- **Not for simple tasks**: Basic explanations, single-file changes, straightforward fixes

## Token Efficiency

- **30-50% token reduction**: Structured reasoning reduces unnecessary iteration
- **Most effective for complex problems**: Simple problems may have overhead

## Works Best With

```
Sequential + Context7
├── Sequential: Coordinates analysis and step-by-step reasoning
└── Context7: Provides official documentation/patterns at each step

Sequential + Serena
├── Serena: Provides codebase structure and dependency information
└── Sequential: Performs architectural analysis using that information

Sequential + Morphllm
├── Sequential: Plans edit strategy and sequence
└── Morphllm: Executes systematic changes following the plan
```

## Available Tools

- `mcp__sequential-thinking__sequentialthinking`: Execute step-by-step reasoning

### Parameters

```typescript
{
  thought: string;            // Current reasoning step
  thoughtNumber: number;      // Current step number
  totalThoughts: number;      // Estimated total steps
  nextThoughtNeeded: boolean; // Whether next step is needed
  isRevision?: boolean;       // Whether revising previous reasoning
  revisesThought?: number;    // Which step is being revised
  branchFromThought?: number; // Branching point step number
  branchId?: string;          // Branch identifier
  needsMoreThoughts?: boolean; // Whether additional steps needed
}
```

## Examples

```
"why is this API slow?" → Sequential (systematic performance analysis)
"design a microservices architecture" → Sequential (structured system design)
"debug this authentication flow" → Sequential (multi-component investigation)
"analyze security vulnerabilities" → Sequential (comprehensive threat modeling)
"explain this function" → Native Claude (simple explanation)
"fix this typo" → Native Claude (straightforward change)
```

### Detailed Usage

```
User: "Why is our API response time over 2 seconds?"

→ mcp__sequential-thinking__sequentialthinking({
    thought: "First, let's identify the API endpoint flow:
              Request → Auth Middleware → Controller → Service → Database → Response",
    thoughtNumber: 1,
    totalThoughts: 5,
    nextThoughtNeeded: true
  })

→ mcp__sequential-thinking__sequentialthinking({
    thought: "Analyzing database queries: Found N+1 query problem in
              UserService.getWithRelations() - executing 50+ queries per request",
    thoughtNumber: 2,
    totalThoughts: 5,
    nextThoughtNeeded: true
  })

→ mcp__sequential-thinking__sequentialthinking({
    thought: "Checking caching layer: No Redis/memory cache for frequently
              accessed data. Each request hits database directly",
    thoughtNumber: 3,
    totalThoughts: 5,
    nextThoughtNeeded: true
  })

→ mcp__sequential-thinking__sequentialthinking({
    thought: "Root causes identified: 1) N+1 query problem, 2) Missing cache layer.
              Recommendations: Use eager loading, add Redis caching",
    thoughtNumber: 4,
    totalThoughts: 4,
    nextThoughtNeeded: false
  })
```

## When NOT to Use

- Simple Q&A → Native Claude
- Code writing/editing → Morphllm/Serena
- Documentation lookup → Context7
- Single-file analysis → Native Read tool

## Best Practices

1. **Use for complex problems only**: Simple problems have overhead
2. **Set appropriate step count**: Too many = inefficient, too few = incomplete
3. **Use revision feature**: If direction is wrong, use `isRevision`
4. **Use branching**: Explore alternatives with `branchFromThought`
