# IDE 설정 (IDE Setup)

## Zed 에디터 설정

Zed에서 ACP 에이전트를 사용하려면 `~/.config/zed/settings.json`에 설정을 추가합니다.

### 기본 설정

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Zed의 Agent 패널을 열고 "OpenClaw ACP"를 선택하면 스레드를 시작할 수 있습니다.

### 원격 Gateway + 에이전트 타겟팅

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

### 환경 변수 설정

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {
        "OPENCLAW_GATEWAY_TOKEN": "<token>"
      }
    }
  }
}
```

---

## Claude Code 설정

Claude Code는 ACP 프로토콜을 직접 지원하므로 별도의 IDE 설정 없이 `openclaw acp`를 사용할 수 있습니다.

### 기본 사용

```bash
claude --server openclaw acp
```

### 원격 Gateway

```bash
claude --server "openclaw acp --url wss://gateway-host:18789 --token <token>"
```

---

## 커스텀 ACP 클라이언트

내장 ACP 클라이언트를 사용하여 브릿지를 디버깅할 수 있습니다.

```bash
# 기본 브릿지 연결
openclaw acp client

# 원격 Gateway 지정
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# 서버 명령어 오버라이드
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

클라이언트는 대화식 프롬프트 인터페이스를 제공하여:
- 프롬프트 입력
- 에이전트 응답 확인
- 세션 ID 확인
- `exit` 또는 `quit`로 종료

---

## ACP 클라이언트 요구사항

ACP 클라이언트는 다음 기능을 구현해야 합니다:

| 기능 | 설명 |
|---|---|
| `initialize` | 에이전트 초기화 |
| `newSession` | 새 세션 생성 |
| `loadSession` | 기존 세션 로드 |
| `prompt` | 프롬프트 전송 |
| `cancel` | 프롬프트 취소 |
| `listSessions` | 세션 목록 |

### 클라이언트 능력

```json
{
  "clientCapabilities": {
    "fs": { "readTextFile": true, "writeTextFile": true },
    "terminal": true
  }
}
```

- **fs**: 파일 읽기/쓰기 지원 여부
- **terminal**: 터미널 기능 지원 여부

---

## 문제 해결

### 연결 실패
- Gateway가 실행 중인지 확인
- URL과 인증 정보 확인
- `--verbose` 플래그로 상세 로그 확인

### 세션 매핑 문제
- `--session` 플래그로 명시적 세션 키 지정
- `--reset-session`으로 새 대화 기록 생성

### 인증 문제
- CLI 플래그, 설정 파일, 환경 변수 순서로 인증 확인
- Gateway 토큰이 유효한지 확인
