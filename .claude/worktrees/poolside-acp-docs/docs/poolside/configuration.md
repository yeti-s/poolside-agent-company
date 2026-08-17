# 설정 (Configuration)

## `settings.yaml` 구성

Poolside CLI의 설정은 `~/.config/poolside/settings.yaml`에 있습니다.
프로젝트 수준 파일의 `agent_servers` 키는 무시됩니다.

### 기본 위치

```
~/.config/poolside/settings.yaml
```

### 자동 마이그레이션

시작 시 구형 `pool.json` 항목이 자동으로 `settings.yaml`으로 마이그레이션됩니다.

---

## `agent_servers` 구성

에이전트 서버는 `agent_servers` 섹션에 정의합니다.

### 필수 규칙

각 서버 항목은 `command` (로컬 stdio 서버) 또는 `url` (원격 네트워크 서버) 중 하나를 설정해야 합니다.

### 핵심 설정 키

| 키 | 설명 |
|---|---|
| `type` | 서버 타입. 기본값: `custom`. 예약값: `registry`, `remote` |
| `command` | 로컬 stdio 서버의 실행 파일 |
| `url` | 원격 ACP 서버 URL (Streamable HTTP) |
| `headers` | `url` 사용 시 필요한 HTTP 헤더 |
| `args` | `command` 실행 파일에 전달할 인수 |
| `env` | 명령어 프로세스의 환경 변수 |
| `default_config_options` | 저장된 ACP 세션 기본값 |

### 기본 설정 예시

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

### 로컬 stdio 서버

```yaml
agent_servers:
  my-local-agent:
    type: custom
    command: my-local-agent
    args: ["--acp"]
    env:
      API_KEY: "your-api-key"
```

### 원격 ACP 서버

```yaml
agent_servers:
  remote-agent:
    type: remote
    url: https://agent.example.com/acp
    headers:
      Authorization: "Bearer your-token"
```

---

## `pool.default_agent_server`

서버를 CLI에 연결하려면 `pool.default_agent_server`에 해당 서버 이름을 할당합니다.

```yaml
pool:
  default_agent_server: Poolside
```

이 설정은 `--agent-server` 플래그를 생략할 때 사용되는 서버를 결정합니다.

---

## 환경 변수

### 인증 환경 변수

| 변수 | 설명 |
|---|---|
| `POOLSIDE_API_KEY` | API 키 |
| `POOLSIDE_API_URL` | API URL |
| `POOLSIDE_TOKEN` | 인증 토큰 |
| `POOLSIDE_STANDALONE_BASE_URL` | 독립 실행형 기본 URL |
| `POOLSIDE_STANDALONE_CONTEXT_LENGTH` | 독립 실행형 컨텍스트 길이 |
| `POOLSIDE_STANDALONE_MODEL` | 독립 실행형 모델 |

### 확인 방법

```bash
env | grep -E '^POOLSIDE_(API_KEY|API_URL|TOKEN|STANDALONE_BASE_URL|STANDALONE_CONTEXT_LENGTH|STANDALONE_MODEL)=' | sed 's/=.*/=<set>/'
```

---

## 설정 파일 레퍼런스

더 많은 설정 옵션은 [Poolside 공식 설정 파일 레퍼런스](https://docs.poolside.ai/settings-file-reference.md)를 참조하세요.
