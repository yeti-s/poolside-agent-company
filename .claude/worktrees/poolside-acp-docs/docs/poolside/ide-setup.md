# IDE 설정 (IDE Setup)

## Zed 에디터 설정

Zed에서 ACP 에이전트를 사용하려면 `~/.config/zed/settings.json`에 설정을 추가합니다.

### 자동 설정

```bash
pool acp setup --editor zed
```

### 수동 설정

```json
{
  "agent_servers": {
    "Poolside": {
      "type": "custom",
      "command": "pool",
      "args": ["acp"],
      "default_config_options": {
        "mode": "always-allow",
        "model": "Qwen/Qwen3.6-35B-A3B-FP8"
      }
    }
  }
}
```

Zed의 Agent 패널에서 Poolside Agent를 선택하면 시작할 수 있습니다.

---

## JetBrains 설정

JetBrains IDE에서 ACP 에이전트를 사용하려면:

### 자동 설정

```bash
pool acp setup --editor jetbrains
```

### 수동 설정

JetBrains IDE의 설정에서 ACP 에이전트를 추가합니다:

- 명령어: `pool`
- 인수: `acp`

---

## Neovim 설정

Neovim에서 ACP 에이전트를 사용하려면 ACP 호환 클라이언트를 설정합니다.

### 설정 예시

```lua
-- Neovim ACP 클라이언트 설정
-- pool acp를 stdio 서버로 연결
```

---

## 데스크톱 앱

Poolside Assistant는 ACP 호환 에이전트를 실행합니다.
데스크톱 앱에서 Poolside 에이전트를 선택하면 됩니다.

---

## 코딩 에이전트 통합

ACP 호환 클라이언트는 Poolside 에이전트를 코딩 에이전트로 사용할 수 있습니다.

### 지원되는 에이전트

- Claude Agent (`claude-agent-acp`)
- Codex (`codex-acp`)
- Gemini (`gemini --acp`)
- 원격 서버 (`http://localhost:3284/acp`)

---

## 로컬 모델 런타임

자체 호스팅 에이전트는 제공자의 reasoning controls를 사용합니다.

### 설정 예시

```yaml
agent_servers:
  my-local-model:
    type: custom
    command: my-local-agent
    args: ["--acp"]
    env:
      API_KEY: "your-api-key"
```

---

## 문제 해결

### 연결 문제

1. Pool CLI가 시스템 PATH에 있는지 확인
2. 선택한 inference endpoint의 인증 불일치 확인
3. 에디터 클라이언트가 특정 ACP 기능을 표시할 수 있는 UI 제한 확인

### 설정/인증 문제 해결

```bash
# 로그 확인
pool acp logs -f
pool acp logs --pretty

# 재인증
pool logout
pool login

# 환경 변수 확인
env | grep -E '^POOLSIDE_(API_KEY|API_URL|TOKEN|STANDALONE_BASE_URL|STANDALONE_CONTEXT_LENGTH|STANDALONE_MODEL)=' | sed 's/=.*/=<set>/'
```

### 설정 디렉토리 확인

```bash
pool config
```

활성 디렉토리와 자격 증명 경로를 표시합니다.

### 환경 변수 간섭 제거

```bash
env -u POOLSIDE_API_KEY -u POOLSIDE_API_URL -u POOLSIDE_TOKEN -u POOLSIDE_STANDALONE_BASE_URL -u POOLSIDE_STANDALONE_CONTEXT_LENGTH -u POOLSIDE_STANDALONE_MODEL pool
```

### 클립보드 (Linux)

- Wayland: `wl-clipboard`
- X11: `xclip` 또는 `xsel`
