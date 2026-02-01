# omc-mcp-extension 설계 문서

> oh-my-claudecode 위에 SuperClaude의 MCP 통합 기능을 추가하는 경량 확장 플러그인

---

## 1. 개요

### 1.1 목적
- **OMC의 강점**: 발전된 스킬/훅/에이전트 시스템, 매직 키워드, 실행 모드
- **SuperClaude의 강점**: MCP 서버 통합 및 행동 지침
- **이 플러그인**: 두 프로젝트의 장점을 결합

### 1.2 설계 원칙
| 원칙 | 설명 |
|------|------|
| **독립성** | OMC 코드 수정 없이 동작 |
| **최소 침습** | 필요한 MCP 설정과 가이드만 포함 |
| **업스트림 호환** | OMC 업데이트와 충돌 없음 |
| **선택적 활성화** | 각 MCP 서버를 개별적으로 활성화/비활성화 가능 |

---

## 2. 아키텍처

### 2.1 OMC의 MCP 디스커버리 시스템 분석

OMC는 다중 소스 디스커버리 시스템을 구현합니다:

```
레지스트리 초기화
├─ ~/.claude/plugins/ 및 ~/.claude/installed-plugins/에서 플러그인 탐색
├─ ~/.claude/settings.json에서 MCP 서버 탐색 (우선순위 1)
├─ ~/.claude/claude_desktop_config.json에서 MCP 서버 탐색 (우선순위 2)
├─ 플러그인 매니페스트에서 MCP 서버 탐색 (우선순위 3)
└─ 활성화된 모든 서버 자동 연결
```

핵심 파일 (`/oh-my-claudecode/src/compatibility/`):
- `discovery.ts` - 설정 파일 탐색 및 파싱
- `registry.ts` - 도구 등록 및 상태 관리
- `mcp-bridge.ts` - MCP 서버 스폰 및 통신

### 2.2 ⚠️ 중요: 호환성 요구사항

**OMC의 플러그인 매니페스트 스키마 분석 결과:**

`PluginManifest` TypeScript 인터페이스 (types.ts, line 69):
```typescript
mcpServers?: Record<string, McpServerEntry>;
```

`discovery.ts` (line 600)의 처리 코드:
```typescript
for (const [name, serverConfig] of Object.entries(plugin.manifest.mcpServers)) {
  servers.push({
    name: `${plugin.name}:${name}`,  // 서버 네이밍 패턴
    config: serverConfig,
    source: plugin.name,
    connected: false,
    tools: [],
  });
}
```

**결론:**
- ✅ `mcpServers`는 **인라인 객체**여야 함
- ❌ `"mcpServers": "../.mcp.json"` 형태의 파일 참조는 **미지원**
- ✅ 서버 이름은 `{plugin-name}:{server-name}` 패턴으로 자동 네이밍

### 2.3 보안 검증

OMC의 보안 화이트리스트 분석 (mcp-bridge.ts):

| 검증 항목 | 결과 | 비고 |
|-----------|------|------|
| `npx` 명령어 | ✅ 허용 | lines 38-52 ALLOWED_COMMANDS |
| `uvx` 명령어 | ✅ 허용 | lines 38-52 ALLOWED_COMMANDS |
| `MORPH_API_KEY` | ✅ 허용 | BLOCKED_ENV_VARS에 없음 |
| `ALL_TOOLS` | ✅ 허용 | BLOCKED_ENV_VARS에 없음 |
| `${HOME}` 패턴 | ✅ 지원 | 환경변수 치환 지원 |

### 2.4 통합 후 시스템 구조

```
┌─────────────────────────────────────────────────────────────────┐
│                      Claude Code Runtime                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MCP Servers                                                     │
│  ├── "t" (OMC Bridge) - OMC's own tools                         │
│  │   └─ mcp__t__lsp_*, mcp__t__ast_* (18 tools)                 │
│  │                                                              │
│  └── External MCP (omc-mcp-extension)                           │
│      ├─ omc-mcp-extension:context7      → Official docs lookup  │
│      ├─ omc-mcp-extension:serena        → Semantic analysis     │
│      ├─ omc-mcp-extension:sequential-thinking → Reasoning       │
│      └─ omc-mcp-extension:morphllm-fast-apply → Bulk editing    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.5 MCP 서버 충돌 분석

| 구분 | OMC (기존) | omc-mcp-extension (추가) | 충돌 여부 |
|------|-----------|-------------------------|----------|
| **서버 이름** | `t` | `context7`, `serena`, `sequential-thinking`, `morphllm-fast-apply` | ❌ 없음 |
| **도구 네임스페이스** | `mcp__t__*` | `mcp__context7__*`, `mcp__serena__*`, ... | ❌ 없음 |
| **역할** | 자체 커스텀 도구 (LSP, AST) | 외부 MCP 서버 (문서, 분석, 추론) | ❌ 상호 보완 |

**결론: 두 플러그인은 완전히 독립적으로 공존 가능**

---

## 3. 프로젝트 구조

```
omc-mcp-extension/
├── .claude-plugin/
│   └── plugin.json           # 플러그인 메타데이터 + 인라인 mcpServers
│
├── mcp/
│   ├── MCP_Context7.md       # Context7 사용 가이드
│   ├── MCP_Serena.md         # Serena 사용 가이드
│   ├── MCP_Sequential.md     # Sequential 사용 가이드
│   └── MCP_Morphllm.md       # Morphllm 사용 가이드
│
├── .mcp.json                  # (참조용 - OMC가 직접 사용하지 않음)
├── README.md                  # 설치 및 사용 가이드
├── LICENSE                    # MIT 라이선스
└── DESIGN.md                  # 이 문서
```

---

## 4. 컴포넌트 상세 설계

### 4.1 plugin.json (핵심 - 인라인 mcpServers)

```json
{
  "name": "omc-mcp-extension",
  "version": "1.0.0",
  "description": "MCP server integration for oh-my-claudecode",
  "author": {
    "name": "chlee1001"
  },
  "repository": "https://github.com/chlee1001/omc-mcp-extension",
  "license": "MIT",
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "serena": {
      "command": "uvx",
      "args": [
        "--from", "git+https://github.com/oraios/serena",
        "serena", "start-mcp-server",
        "--context", "ide-assistant"
      ]
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "morphllm-fast-apply": {
      "command": "npx",
      "args": ["@morph-llm/morph-fast-apply", "${HOME}"],
      "env": {
        "MORPH_API_KEY": "${MORPH_API_KEY}",
        "ALL_TOOLS": "true"
      }
    }
  }
}
```

### 4.2 .mcp.json (참조용 - 직접 사용되지 않음)

> ⚠️ 이 파일은 문서 및 독립 실행 참조용입니다. OMC는 plugin.json의 인라인 mcpServers만 사용합니다.

// ... existing code ...

---

## 5. 설치 및 사용

// ... existing code ...

---

## 6. MCP 서버 선택 매트릭스

// ... existing code ...

---

## 7. 테스트 계획

// ... existing code ...

---

## 8. 향후 확장

// ... existing code ...

---

## 9. 변경 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|----------|
| 1.0.0 | 2025-02-01 | 초기 설계 |
| 1.0.1 | 2025-02-01 | OMC MCP 구조 분석 반영, 아키텍처 다이어그램 상세화 |
| 1.0.2 | 2025-02-01 | OMC 디스커버리 시스템 심층 분석, 인라인 mcpServers 방식으로 변경 |