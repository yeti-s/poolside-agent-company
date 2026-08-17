# 프로토콜 상세 (Protocol Details)

## ACP 프로토콜

Poolside CLI는 Agent Client Protocol (ACP)을 통해 IDE와 통신합니다.
기본 통신 방식은 stdio이며, 실험적으로 Streamable HTTP도 지원합니다.

## 통신 방식

### stdio 모드 (기본)

- **전송 계층**: stdio (표준 입출력)
- **인코딩**: NDJSON (Newline-Delimited JSON)
- **스트리밍**: 각 이벤트는 새 줄로 분리된 JSON 객체

### Streamable HTTP 모드 (실험적)

- **전송 계층**: HTTP POST `/acp`
- **포트**: 기본 3284
- **바인딩**: 기본 127.0.0.1

---

## 메시지 구조

### 프롬프트 전송

IDE에서 에이전트에 프롬프트를 전송합니다:

```json
{
  "sessionId": "session-uuid",
  "prompt": [
    { "type": "text", "text": "Hello, how are you?" }
  ]
}
```

### 에이전트 응답

에이전트의 응답은 스트리밍 이벤트로 반환됩니다:

```json
{ "sessionId": "...", "update": { "sessionUpdate": "agent_message_chunk", "content": { "type": "text", "text": "Hello" } } }
{ "sessionId": "...", "update": { "sessionUpdate": "tool_call", "toolCallId": "...", "title": "Bash: ls", "status": "in_progress" } }
{ "sessionId": "...", "update": { "sessionUpdate": "tool_call_update", "toolCallId": "...", "status": "completed" } }
```

---

## 세션 관리

### 세션 생성

새 ACP 세션이 생성됩니다. `pool acp serve`는 각 연결마다 새 에이전트 인스턴스를 생성합니다.

### 세션 기본값

`settings.yaml`의 `default_config_options`에서 세션 기본값을 구성할 수 있습니다:

```yaml
agent_servers:
  Poolside:
    type: custom
    command: pool
    args:
      - acp
    default_config_options:
      mode: always-allow
      model: Qwen/Qwen3.6-35B-A3B-FP8
```

### 세션 모드

- `always-allow`: 모든 작업에 승인 불필요
- 다른 모드: `settings.yaml` 또는 CLI 플래그로 구성

---

## 에러 처리

### 인증 에러

```
403 Forbidden: please check the api-key you provided
```

- `POOLSIDE_API_KEY` 또는 `POOLSIDE_TOKEN` 환경 변수 확인
- `pool logout` 후 `pool login`으로 재인증

### 세션/프롬프트 에러

```
Error during ACP method session/prompt
```

- 짧은 텍스트 메시지로 테스트
- `/logs`로 진단 아카이브 생성 (`logs.zip`)

### 연결 에러

에디터 연결 문제 진단:

```bash
# 로그 확인
pool acp logs -f
pool acp logs --pretty

# 환경 변수 확인
env | grep -E '^POOLSIDE_(API_KEY|API_URL|TOKEN|STANDALONE_BASE_URL|STANDALONE_CONTEXT_LENGTH|STANDALONE_MODEL)=' | sed 's/=.*/=<set>/'
```

---

## 샌드박스

### 샌드박스 모드

`--sandbox` 플래그로 샌드박스 동작을 제어합니다:

| 모드 | 설명 |
|---|---|
| `required` | 샌드박스 필수 |
| `disabled` | 샌드박스 비활성화 |

### 예시

```bash
pool acp --sandbox required
pool acp --sandbox disabled
```

---

## 기술 동작

### 외부 에디터 연결

에디터는 `/acp` 엔드포인트를 통해 Streamable HTTP로 연결할 수 있습니다.

### Thought Depth 제어

세션 구성 옵션을 통해 thought depth를 제어합니다:

- Poolside 호스팅: `max` 또는 `none`
- OpenRouter: 모델 메타데이터에 따름
- 자체 호스팅: 제공자별

### Slash Commands

에디터는 ACP를 통해 다음 슬래시 명령어를 라우팅할 수 있습니다:

- `/plan`, `/clear`, `/compact`, `/share`, `/rename`
- `/mcp`, `/sandbox`, `/sandbox-apply-to-host`, `/usage`, `/skills`

호환되는 클라이언트는 스킬을 슬래시 명령어로 노출할 수도 있습니다.
