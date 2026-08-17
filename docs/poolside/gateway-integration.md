# Gateway 연동 (Gateway Integration)

## Gateway 연결

ACP 브릿지는 기존 인증 설정을 사용하여 Gateway URL과 인증을 해결합니다.

### 인증 우선순위

1. CLI 플래그 (`--url`, `--token`, `--password`)
2. 설정 파일 (`gateway.remote.*`)
3. 환경 변수 (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`)

### 연결 상세 빌드

```typescript
const connection = buildGatewayConnectionDetails({
  config: cfg,
  url: opts.gatewayUrl,
});
```

### Gateway 클라이언트 설정

```typescript
const gateway = new GatewayClient({
  url: connection.url,
  token: token || undefined,
  password: password || undefined,
  clientName: GATEWAY_CLIENT_NAMES.CLI,
  clientDisplayName: "ACP",
  clientVersion: "acp",
  mode: GATEWAY_CLIENT_MODES.CLI,
  onEvent: (evt) => agent?.handleGatewayEvent(evt),
  onHelloOk: () => agent?.handleGatewayReconnect(),
  onClose: (code, reason) => agent?.handleGatewayDisconnect(`${code}: ${reason}`),
});
```

---

## 세션 매핑 (Session Mapping)

### 기본 매핑

각 ACP 세션은 기본적으로 고유한 Gateway 세션 키를 받습니다:

- 기본값: `acp:<uuid>` (고유 UUID)
- 오버라이드: `--session` 플래그 또는 `_meta.sessionKey`

### 세션 키 해결 로직

```
1. _meta.sessionKey 설정? → 직접 사용
2. _meta.sessionLabel 설정? → sessions.resolve로 라벨 해결
3. CLI defaultSessionKey 설정? → 직접 사용
4. CLI defaultSessionLabel 설정? → sessions.resolve로 라벨 해결
5. 위 모두 없음 → acp:<uuid> fallback
```

### 세션 라벨 해결

```typescript
const resolved = await gateway.request<{ ok: true; key: string }>(
  "sessions.resolve",
  { label: params.meta.sessionLabel }
);
```

### 세션 리셋

```typescript
await gateway.request("sessions.reset", { key: params.sessionKey });
```

`--reset-session` 플래그 또는 `_meta.resetSession: true`로 활성화됩니다.

---

## 이벤트 변환 (Event Translation)

### 프롬프트 → Gateway 메시지

ACP 프롬프트 입력은 Gateway `chat.send`로 변환됩니다:

| ACP 프롬프트 | Gateway 메시지 |
|---|---|
| `text` 블록 | 프롬프트 텍스트 |
| `resource` 블록 | 프롬프트 텍스트 |
| `resource_link` (이미지 mime) | 첨부 파일 |
| `image` 블록 | 첨부 파일 |
| 작업 디렉토리 | 프롬프트 접두어 (기본값 on, `--no-prefix-cwd`로 끄기) |

### Gateway 이벤트 → ACP 이벤트

Gateway 스트리밍 이벤트는 ACP 이벤트로 변환됩니다:

| Gateway 상태 | ACP stopReason |
|---|---|
| `complete` | `stop` |
| `aborted` | `cancel` |
| `error` | `refusal` |

### 이벤트 처리 흐름

```
Gateway 이벤트 → handleGatewayEvent()
    ├── event === "chat" → handleChatEvent()
    │       ├── state === "delta" → handleDeltaEvent() → agent_message_chunk
    │       ├── state === "final" → finish_prompt("end_turn")
    │       ├── state === "aborted" → finish_prompt("cancelled")
    │       └── state === "error" → finish_prompt("refusal")
    └── event === "agent" → handleAgentEvent()
            ├── phase === "start" → tool_call 이벤트
            └── phase === "result" → tool_call_update 이벤트
```

### 도구 호출 변환

```typescript
// 도구 호출 시작
await connection.sessionUpdate({
  sessionId: pending.sessionId,
  update: {
    sessionUpdate: "tool_call",
    toolCallId,
    title: formatToolTitle(name, args),
    status: "in_progress",
    rawInput: args,
    kind: inferToolKind(name),
  },
});

// 도구 호출 결과
await connection.sessionUpdate({
  sessionId: pending.sessionId,
  update: {
    sessionUpdate: "tool_call_update",
    toolCallId,
    status: isError ? "failed" : "completed",
    rawOutput: data.result,
  },
});
```

---

## 세션 스토어 (Session Store)

인메모리 세션 스토어는 다음을 관리합니다:

| 메서드 | 설명 |
|---|---|
| `createSession(params)` | 새 세션 생성 (sessionId, sessionKey, cwd) |
| `getSession(sessionId)` | 세션 ID로 세션 조회 |
| `getSessionByRunId(runId)` | 실행 ID로 세션 조회 |
| `setActiveRun(sessionId, runId, abortController)` | 활성 실행 설정 |
| `clearActiveRun(sessionId)` | 활성 실행 지우기 |
| `cancelActiveRun(sessionId)` | 활성 실행 취소 |
| `clearAllSessionsForTest()` | 테스트용 전체 세션 지우기 |

### AcpSession 구조

```typescript
type AcpSession = {
  sessionId: SessionId;        // ACP 세션 ID (UUID)
  sessionKey: string;           // Gateway 세션 키
  cwd: string;                  // 작업 디렉토리
  createdAt: number;            // 생성 시간
  abortController: AbortController | null;  // 활성 실행 취소용
  activeRunId: string | null;   // 활성 실행 ID
};
```

---

## 운영 관련 (Operational Notes)

- ACP 세션은 브릿지 프로세스 수명 동안 메모리에 저장됩니다.
- Gateway 세션 상태는 Gateway 자체에서 영구 저장됩니다.
- `--verbose`는 ACP/Gateway 브릿지 이벤트를 stderr에 로깅합니다 (stdout 아님).
- ACP 실행은 취소할 수 있으며, 활성 실행 ID는 세션별로 추적됩니다.
- Gateway 재연결 시 `handleGatewayReconnect()`가 호출됩니다.
- Gateway 끊김 시 모든 보류 중인 프롬프트가 에러로 reject됩니다.
