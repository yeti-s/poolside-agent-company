# CLI 명령어 (CLI Commands)

## `pool acp` — ACP 서버 실행

IDE 통합을 위한 ACP 서버를 실행합니다. 기본값은 stdio를 통해 ACP 메시지를 처리합니다.

### 기본 사용법

```bash
pool acp
```

### 옵션

| 옵션 | 설명 |
|---|---|
| `--sandbox <mode>` | 샌드박스 동작 재정의 (`required` 또는 `disabled`) |
| `--settings <settings>` | YAML 파일 경로 또는 인라인 YAML 문자열에서 추가 구성 로드 |
| `--version, -v` | 버전 번호 출력 후 종료 |

### 환경 변수 주입

```bash
KEY=VALUE pool acp
```

---

## `pool acp serve` — Streamable HTTP 서버 (실험적)

에이전트를 Streamable HTTP를 통해 라우팅합니다. 각 연결마다 새 에이전트 인스턴스가 생성됩니다.

### 기본 사용법

```bash
pool acp serve
```

### 옵션

| 옵션 | 설명 |
|---|---|
| `--host <host>` | 바인딩할 네트워크 인터페이스 (기본값: `127.0.0.1`) |
| `--port <port>` | TCP 포트 (기본값: `3284`) |
| `--access-log <destination>` | 로그 대상 (`stderr`, `stdout`, `-`, `off`, 또는 파일 경로) |

### 예시

```bash
pool acp serve --host 127.0.0.1 --port 3284
```

---

## `pool acp setup` — IDE 설정 자동화

에디터 설정을 자동으로 구성합니다.

### 기본 사용법

```bash
pool acp setup --editor <editor>
```

### 옵션

| 옵션 | 설명 |
|---|---|
| `--editor <editor>` | 필수 인수. `zed` 또는 `jetbrains` |

---

## `pool acp logs` — 디버그 로그 조회

Poolside 로그 폴더에서 디버그 로그를 가져옵니다.

### 기본 사용법

```bash
pool acp logs
```

### 옵션

| 옵션 | 설명 |
|---|---|
| `--follow, -f` | 새 항목을 계속 스트림 |
| `--pretty, -p` | 출력에 서식과 색상 적용 |
| `--session` | 특정 세션 ID로 결과 필터링 |

### 예시

```bash
# 로그 스트리밍
pool acp logs -f

# 예쁘게 출력
pool acp logs --pretty

# 특정 세션 필터링
pool acp logs --session <session-id>
```

---

## `pool --agent-server` — 에이전트 서버 지정

`pool`을 ACP 클라이언트로 실행하여 외부 에이전트 서버에 연결합니다.

### 기본 사용법

```bash
pool --agent-server <server>
```

### 지원되는 에이전트

| 에이전트 | 명령어 |
|---|---|
| Claude Agent | `claude-agent-acp` |
| Codex | `codex-acp` |
| Gemini | `"gemini --acp"` |
| 원격 서버 | `http://localhost:3284/acp` |

### 대화식 선택

```bash
pool -s
```

사용 가능한 설정 목록에서 선택할 수 있는 대화식 메뉴가 열립니다.

---

## Slash Commands

에디터는 ACP를 통해 다음 슬래시 명령어를 라우팅할 수 있습니다:

- `/plan` — 계획 모드
- `/clear` — 대화 기록 지우기
- `/compact` — 대화 기록 압축
- `/share` — 대화 공유
- `/rename` — 세션 이름 변경
- `/mcp` — MCP 서버 관리
- `/sandbox` — 샌드박스 설정
- `/sandbox-apply-to-host` — 호스트에 샌드박스 적용
- `/usage` — 사용량 정보
- `/skills` — 스킬 관리

호환되는 클라이언트는 스킬을 슬래시 명령어로 노출할 수도 있습니다.

---

## Thought Depth (사유 깊이)

세션 구성 옵션을 통해 thought depth를 제어합니다. 제공자별로 다릅니다:

| 제공자 | 옵션 | 설명 |
|---|---|---|
| Poolside 호스팅 | `max`, `none` | Poolside가 제어 |
| OpenRouter | 모델 메타데이터에 따름 | 모델이 결정 |
| 자체 호스팅 | 제공자별 | 제공자가 제어 |
