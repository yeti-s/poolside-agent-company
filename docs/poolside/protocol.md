# 프로토콜 상세 (Protocol Details)

## ACP 프로토콜 버전

`@agentclientprotocol/sdk` (현재 0.13.x)를 사용합니다.

## 통신 방식

- **전송 계층**: stdio (표준 입출력)
- **인코딩**: NDJSON (Newline-Delimited JSON)
- **스트리밍**: 각 이벤트는 새 줄로 분리된 JSON 객체

---

## API 엔드포인트

### `initialize` — 초기화

클라이언트가 에이전트와 통신하기 전에 먼저 호출해야 합니다.

**요청:**
```json
{
  "protocolVersion": "0.13.x",
  "clientCapabilities": {
    "fs": { "readTextFile": true, "writeTextFile": true },
    "terminal": true
  },
  "clientInfo": {
    "name": "openclaw-acp-client",
    "version": "1.0.0"
  }
}
```

**응답:**
```json
{
  "protocolVersion": "0.13.x",
  "agentCapabilities": {
    "loadSession": true,
    "promptCapabilities": {
      "image": true,
      "audio": false,
      "embeddedContext": true
    },
    "mcpCapabilities": {
      "http": false,
      "sse": false
    },
    "sessionCapabilities": {
      "list": {}
    }
  },
  "agentInfo": {
    "name": "openclaw-acp",
    "title": "OpenClaw ACP Gateway",
    "version": "<VERSION>"
  },
  "authMethods": []
}
```

**에이전트 능력:**
| 능력 | 값 | 설명 |
|---|---|---|
| `loadSession` | `true` | 기존 세션 로드 지원 |
| `promptCapabilities.image` | `true` | 이미지 프롬프트 지원 |
| `promptCapabilities.audio` | `false` | 오디오 프롬프트 미지원 |
| `promptCapabilities.embeddedContext` | `true` | 임베디드 컨텍스트 지원 |
| `mcpCapabilities.http` | `false` | HTTP MCP 미지원 |
| `mcpCapabilities.sse` | `false` | SSE MCP 미지원 |

---

### `newSession` — 새 세션 생성

새 ACP 세션을 생성합니다.

**요청:**
```json
{
  "sessionId": null,
  "cwd": "/path/to/project",
  "mcpServers": [],
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true,
    "requireExisting": false
  }
}
```

**응답:**
```json
{
  "sessionId": "uuid-string"
}
```

**_meta 필드:**
| 필드 | 설명 |
|---|---|
| `sessionKey` | Gateway 세션 키 직접 지정 |
| `sessionLabel` | 기존 세션 라벨로 해결 |
| `resetSession` | 첫 사용 전에 새 대화 기록 생성 |
| `requireExisting` | 키/라벨이 존재하지 않으면 실패 |

---

### `loadSession` — 세션 로드

기존 세션을 로드합니다.

**요청:**
```json
{
  "sessionId": "existing-session-id",
  "cwd": "/path/to/project",
  "mcpServers": [],
  "_meta": {
    "sessionKey": "agent:main:main"
  }
}
```

**응답:**
```json
{}
```

---

### `prompt` — 프롬프트 전송

에이전트에 프롬프트를 전송합니다.

**요청:**
```json
{
  "sessionId": "session-uuid",
  "prompt": [
    { "type": "text", "text": "Hello, how are you?" },
    { "type": "resource", "resource": { "text": "Some context..." } },
    { "type": "resource_link", "uri": "file:///path/to/image.png", "title": "Screenshot" },
    { "type": "image", "data": "base64data", "mimeType": "image/png" }
  ],
  "_meta": {
    "thinking": { "thinkingLevel": "high" },
    "deliver": true,
    "timeoutMs": 30000
  }
}
```

**응답:**
```json
{
  "stopReason": "end_turn"
}
```

**프롬프트 콘텐츠 블록:**
| 타입 | 설명 |
|---|---|
| `text` | 일반 텍스트 프롬프트 |
| `resource` | 리소스 텍스트 (임베디드 컨텍스트) |
| `resource_link` | URI 링크 (이미지 mime 타입인 경우 첨부 파일로 처리) |
| `image` | 이미지 첨부 파일 (base64 인코딩) |

**_meta 필드:**
| 필드 | 설명 |
|---|---|
| `thinking.thinkingLevel` | 추론 레벨 |
| `deliver` | 즉시 전달 |
| `timeoutMs` | 타임아웃 (밀리초) |

**스트리밍 이벤트:**
```json
{ "sessionId": "...", "update": { "sessionUpdate": "agent_message_chunk", "content": { "type": "text", "text": "Hello" } } }
{ "sessionId": "...", "update": { "sessionUpdate": "tool_call", "toolCallId": "...", "title": "Bash: ls", "status": "in_progress", "rawInput": {}, "kind": "execute" } }
{ "sessionId": "...", "update": { "sessionUpdate": "tool_call_update", "toolCallId": "...", "status": "completed", "rawOutput": {} } }
{ "sessionId": "...", "update": { "sessionUpdate": "available_commands_update", "availableCommands": [{ "name": "config" }] } }
```

---

### `cancel` — 프롬프트 취소

활성 프롬프트 실행을 취소합니다.

**요청:**
```json
{
  "sessionId": "session-uuid"
}
```

**응답:** 없음 (알림)

---

### `listSessions` — 세션 목록

Gateway 세션 목록을 반환합니다 (IDE 세션 피커용).

**요청:**
```json
{
  "cwd": "/path/to/project",
  "_meta": { "limit": 100 }
}
```

**응답:**
```json
{
  "sessions": [
    {
      "sessionId": "agent:main:main",
      "cwd": "/path/to/project",
      "title": "Support Inbox",
      "updatedAt": "2026-08-17T07:00:00.000Z",
      "_meta": {
        "sessionKey": "agent:main:main",
        "kind": "agent",
        "channel": "chat"
      }
    }
  ],
  "nextCursor": null
}
```

---

### `authenticate` — 인증

```json
{}
```

응답: `{}` (OpenClaw ACP는 현재 인증 방법을 사용하지 않음)

---

### `setSessionMode` — 세션 모드 설정

세션의 추론 레벨을 설정합니다.

**요청:**
```json
{
  "sessionId": "session-uuid",
  "modeId": "high"
}
```

**응답:** `{}`

---

## 메시지 형식

모든 메시지는 NDJSON (Newline-Delimited JSON) 형식입니다.
각 JSON 객체는 새 줄로 분리됩니다.

에러 처리:
- Gateway 연결 끊김: 모든 보류 중인 프롬프트가 에러로.reject
- 세션 없음: `Session <id> not found` 에러
- 도구 실패: `tool_call_update`에서 `status: "failed"`
