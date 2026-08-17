# CLI 명령어 (CLI Commands)

## `openclaw acp` — ACP 브릿지 실행

IDE 통합을 위한 ACP 브릿지를 실행합니다. Gateway-backed ACP 서버로, IDE 통합을 지원합니다.

### 기본 사용법

```bash
openclaw acp
```

### 옵션

| 옵션 | 설명 |
|---|---|
| `--url <url>` | Gateway WebSocket URL (설정 시 `gateway.remote.url` 기본값 사용) |
| `--token <token>` | Gateway 인증 토큰 |
| `--password <password>` | Gateway 인증 비밀번호 |
| `--session <key>` | 기본 세션 키 (예: `"agent:main:main"`) |
| `--session-label <label>` | 기본 세션 라벨로 해결 |
| `--require-existing` | 세션 키/라벨이 없으면 실패 |
| `--reset-session` | 첫 사용 전에 세션 키 리셋 |
| `--no-prefix-cwd` | 프롬프트에 작업 디렉토리 접두어 붙이지 않음 |
| `--verbose, -v` | stderr로 상세 로그 |
| `--help, -h` | 도움말 표시 |

### 원격 Gateway 연결

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

### 기존 세션 키에 연결

```bash
openclaw acp --session agent:main:main
```

### 세션 라벨로 연결

```bash
openclaw acp --session-label "support inbox"
```

### 세션 리셋

```bash
openclaw acp --session agent:main:main --reset-session
```

---

## `openclaw acp client` — ACP 클라이언트 (디버그용)

IDE 없이 브릿지의 설치를 검증하기 위한 내장 ACP 클라이언트입니다.
ACP 브릿지를 스폰하고 대화식으로 프롬프트를 입력할 수 있습니다.

### 기본 사용법

```bash
openclaw acp client
```

### 원격 Gateway 연결

```bash
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>
```

### 서버 명령어 오버라이드

```bash
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

### `acp client` 옵션

| 옵션 | 설명 |
|---|---|
| `--cwd <dir>` | ACP 세션의 작업 디렉토리 |
| `--server <command>` | ACP 서버 명령어 (기본값: `openclaw`) |
| `--server-args <args...>` | ACP 서버에 전달할 추가 인수 |
| `--server-verbose` | ACP 서버에서 상세 로그 활성화 |
| `--verbose, -v` | 클라이언트 상세 로그 |

---

## 에이전트 선택 (Agent Selection)

ACP는 에이전트를 직접 선택하지 않습니다. **Gateway 세션 키**로 라우팅합니다.

에이전트별 세션 키를 사용하여 특정 에이전트 타겟팅:

```bash
# 메인 에이전트
openclaw acp --session agent:main:main

# 디자인 에이전트
openclaw acp --session agent:design:main

# QA 버그 트래커
openclaw acp --session agent:qa:bug-123
```

각 ACP 세션은 단일 Gateway 세션 키에 매핑됩니다. 하나의 에이전트는 여러 세션을 가질 수 있으며,
ACP는 키나 라벨을 오버라이드하지 않는 한 격리된 `acp:<uuid>` 세션을 기본값으로 사용합니다.

---

## 설정 예시

### 설정 파일에 영구 저장

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

### 플래그로 직접 실행 (설정 파일에 기록하지 않음)

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```
