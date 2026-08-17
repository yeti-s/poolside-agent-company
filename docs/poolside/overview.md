# ACP 개요 (Overview)

## ACP란?

ACP(Agent Client Protocol)는 IDE와 같은 클라이언트가 AI 에이전트 세션을 조작하기 위한 표준 프로토콜입니다.
Poolside CLI는 ACP 브릿지를 통해 IDE 클라이언트와 Gateway(에이전트 백엔드) 사이에서 중재자 역할을 합니다.

## 아키텍처

```
┌─────────────┐     NDJSON over stdio      ┌──────────────────┐     WebSocket          ┌─────────────┐
│   IDE       │ ◄───────────────────────── ►│  ACP 브릿지       │ ◄────────────────────► │  Gateway    │
│ (Zed, etc.) │     Agent Client Protocol   │  (openclaw acp)   │     (chat.send, etc.)  │  (백엔드)    │
└─────────────┘                             └──────────────────┘                        └─────────────┘
```

1. **IDE 클라이언트**: ACP 프로토콜을 지원하는 IDE (Zed 등)
2. **ACP 브릿지**: `openclaw acp` 명령어. stdio로 ACP 메시지를 받고 Gateway WebSocket으로 변환
3. **Gateway**: 에이전트 세션을 관리하는 백엔드 서비스

## 핵심 목표

- **최소한의 ACP 표면**: stdio, NDJSON만 사용
- **재연결 시 안정적인 세션 매핑**: ACP 세션 ID를 Gateway 세션 키에 매핑
- **기존 Gateway 세션 스토어 활용**: list/resolve/reset 기능 사용
- **안전한 기본값**: 기본적으로 격리된 ACP 세션 키 사용

## 구성 요소

### 1. ACP 서버 (브릿지)

`src/acp/server.ts`에서 구현됩니다.

- `serveAcpGateway()`: ACP 브릿지 서버 시작
- stdio(표준 입출력)를 통해 ACP NDJSON 스트림 처리
- Gateway 클라이언트 연결 및 이벤트 라우팅

### 2. ACP 세션 관리

`src/acp/session.ts`에서 구현됩니다.

- `createInMemorySessionStore()`: 인메모리 세션 스토어 생성
- 세션 ID → 세션 키 매핑 관리
- 활성 실행(run) 추적 및 취소 지원

### 3. 세션 매핑

`src/acp/session-mapper.ts`에서 구현됩니다.

- 세션 키/라벨 해석 및 해결
- 세션 리셋 로직
- `_meta` 메타데이터 파싱

### 4. 이벤트 매핑

`src/acp/event-mapper.ts`에서 구현됩니다.

- ACP 프롬프트 → Gateway 메시지 변환
- 이미지 첨부 파일 추출
- 도구(tool) 호출 제목 포맷팅 및 종류 추론

### 5. 에이전트 구현

`src/acp/translator.ts`의 `AcpGatewayAgent` 클래스.

- ACP `Agent` 인터페이스 구현
- Gateway 이벤트 → ACP 이벤트 변환
- 프롬프트, 세션 관리, 취소 처리

## 데이터 흐름

### 프롬프트 실행 흐름

```
IDE → prompt() → ACP 브릿지 → Gateway chat.send → Gateway 에이전트 실행
                                                        ↓
IDE ← agent_message_chunk + tool_call ← ACP 브릿지 ← Gateway streaming events
```

1. IDE가 `prompt()` 호출
2. 브릿지가 프롬프트 텍스트/첨부파일을 추출하여 Gateway `chat.send`로 변환
3. Gateway가 에이전트를 실행하고 스트리밍 이벤트 반환
4. 브릿지가 Gateway 이벤트를 ACP `agent_message_chunk`, `tool_call` 이벤트로 변환
5. IDE로 스트리밍 반환

### 세션 생성 흐름

```
IDE → newSession() → ACP 브릿지 → Gateway 세션 키 생성/해결 → 세션 스토어 저장
       ↓
IDE ← sessionId ← 브릿지 ← Gateway ← 세션 키 결정
```

## 세션 모델

| 기본값 | 설명 |
|---|---|
| `acp:<uuid>` | 각 ACP 세션은 기본적으로 고유한 UUID 세션 키를 받음 |
| `agent:<scope>:<name>` | 에이전트별 세션 키로 특정 에이전트 타겟팅 가능 |
| 세션 라벨 | 기존 세션을 라벨로 해결 |
| 세션 리셋 | 동일한 키로 새 대화 기록 생성 |

## 인증

Gateway 인증은 다음 순서로 해결됩니다:

1. CLI 플래그 (`--url`, `--token`, `--password`)
2. 설정 (`gateway.remote.*`)
3. 환경 변수 (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`)

```
CLI 플래그 → 설정 파일 → 환경 변수 → Gateway 인증
    ↓
GatewayClient 생성 (token/password)
    ↓
WebSocket 연결 → auth 검증
    ↓
성공 → onHelloOk → handleGatewayReconnect()
실패 → onClose → handleGatewayDisconnect()
```

OpenClaw ACP는 현재 `authMethods: []` (인증 방법 없음)을 반환합니다.
Gateway 인증은 `GatewayClient`가 담당하며, ACP 브릿지는 Gateway의 인증 상태를 투명하게 전달합니다.

---

## 에러 처리

### Gateway 연결 끊김

```
Gateway disconnected: <reason>
```

- 모든 보류 중인 프롬프트가 에러로 reject됩니다.
- 세션 스토어의 활성 실행이 정리됩니다.
- `AcpGatewayAgent.handleGatewayDisconnect()`에서 처리됩니다.

### 세션 관련 에러

| 시나리오 | 에러 메시지 |
|---|---|
| 세션 없음 | `Session <id> not found` |
| 세션 키 없음 | `Session key not found: <key>` |
| 세션 라벨 해결 실패 | `Unable to resolve session label: <label>` |
| requireExisting + 세션 없음 | `Session key not found: <key>` |

### 도구 실패

`tool_call_update`에서 `status: "failed"`로 반환됩니다.

```json
{
  "sessionUpdate": "tool_call_update",
  "toolCallId": "...",
  "status": "failed",
  "rawOutput": { "error": "Command not found" }
}
```

### 프롬프트 실패

| Gateway 상태 | ACP stopReason |
|---|---|
| `complete` | `end_turn` |
| `aborted` | `cancelled` |
| `error` | `refusal` |

### 프롬프트 타임아웃

`_meta.timeoutMs`로 타임아웃을 설정할 수 있습니다. 타임아웃 초과 시 Gateway가 `aborted` 상태로 전환되고 ACP는 `cancelled`로 처리됩니다.

---

## Rate Limiting (속도 제한)

### Gateway 레벨

Gateway가 속도 제한을 적용합니다. ACP 브릿지는 Gateway의 응답을 통해 속도 제한을 감지합니다.

- Gateway가 429 Too Many Requests를 반환하면, ACP 브릿지는 에러를 프롬프트 reject로 전달합니다.
- 클라이언트는 `timeoutMs`를 설정하여 프롬프트 타임아웃을 관리할 수 있습니다.

### 세션 레벨

- 각 세션당 하나의 활성 실행만 허용됩니다.
- 새 프롬프트가 오면 기존 활성 실행이 자동으로 취소됩니다.
- `cancel()`을 명시적으로 호출하여 실행을 취소할 수 있습니다.

### 메모리 제한

- 세션 스토어는 인메모리입니다. 브릿지 재시작 시 모든 세션 정보가 손실됩니다.
- Gateway 세션은 영구 저장되므로, 재연결 시 `loadSession()`으로 복원할 수 있습니다.
