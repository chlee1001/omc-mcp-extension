# Morphllm Fast Apply MCP Server

**Purpose**: Pattern-based code editing engine with token optimization for bulk transformations

## Triggers

Claude should use Morphllm when:

- Multi-file edits with consistent patterns
- Framework updates (e.g., React class → hooks)
- Style guide enforcement, code cleanup
- Bulk text replacements
- Natural language edit instructions within specific scope

## Requirements

**MORPH_API_KEY environment variable required**

```bash
# Add to ~/.zshrc or ~/.bashrc
export MORPH_API_KEY="your-api-key-here"

# Restart terminal or run
source ~/.zshrc
```

Without the API key, this MCP server will not function.

## Choose When

| Scenario | Use Morphllm | Alternative |
|----------|-------------|-------------|
| Replace all console.log → logger | ✅ | - |
| React class → hooks conversion | ✅ | - |
| Apply ESLint rules | ✅ | - |
| Unify code style | ✅ | - |
| Rename function (with references) | ❌ | Serena |
| Single-file small edits | ❌ | Native Edit tool |
| Symbol-based refactoring | ❌ | Serena |

### Decision Criteria

- **Over Serena**: For pattern-based edits, not symbol operations
- **For bulk operations**: Style enforcement, framework updates, text replacements
- **When token efficiency matters**: Fast Apply scenarios
- **For simple to medium complexity**: <10 files, straightforward transformations
- **Not for semantic operations**: Symbol renames, dependency tracking, LSP integration

## Works Best With

```
Serena + Morphllm
├── Serena: Analyzes semantic context (identifies what to change)
└── Morphllm: Executes precise edits (actual code modifications)

Sequential + Morphllm
├── Sequential: Plans edit strategy and sequence
└── Morphllm: Executes systematic changes following the plan

Context7 + Morphllm
├── Context7: Identifies migration patterns (e.g., Vue 2 → Vue 3)
└── Morphllm: Executes bulk transformations following patterns
```

## Available Tools

- `mcp__morphllm-fast-apply__edit_file`: Edit file with pattern-based transformation

### Parameters

```typescript
{
  path: string;        // File path to edit
  code_edit: string;   // Code changes with context
  instruction: string; // Edit instructions (what and why)
  dryRun?: boolean;    // Preview without applying (default: false)
}
```

## Examples

```
"update all React class components to hooks" → Morphllm (pattern transformation)
"enforce ESLint rules across project" → Morphllm (style guide application)
"replace all console.log with logger calls" → Morphllm (bulk text replacement)
"rename getUserData function everywhere" → Serena (symbol operation)
"analyze code architecture" → Sequential (complex analysis)
"explain this algorithm" → Native Claude (simple explanation)
```

### Detailed Usage

```
User: "Convert all var declarations to const"

→ mcp__morphllm-fast-apply__edit_file({
    path: "src/utils.js",
    code_edit: `
// ... existing code ...
const data = fetchData();
const config = loadConfig();
// ... existing code ...
`,
    instruction: "Convert var to const for immutable bindings"
  })
```

```
User: "Convert React class component to functional with hooks"

→ mcp__morphllm-fast-apply__edit_file({
    path: "src/components/UserProfile.jsx",
    code_edit: `
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      // ... existing render logic ...
    </div>
  );
}

export default UserProfile;
`,
    instruction: "Convert class component to functional component with hooks"
  })
```

## Token Efficiency

- **Efficient editing**: Only sends changed portions, not entire file
- **`// ... existing code ...`**: Indicates unchanged sections
- **Optimal for bulk operations**: Significant benefit for multi-file edits

## When NOT to Use

- Symbol-based refactoring → Serena
- Complex analysis before editing → Sequential + Serena
- API key not configured (won't work)
- Single-line modifications → Native Edit tool (faster)

## Best Practices

1. **Use dryRun first**: Preview large changes before applying
2. **Provide context**: Use `// ... existing code ...` to indicate position
3. **Clear instructions**: Explain what and why in `instruction` field
4. **Batch processing**: For multiple files, make parallel calls
5. **Combine with Serena**: Use Serena to identify targets, Morphllm to edit
