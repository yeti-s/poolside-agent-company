# ACP 개요 (Overview)

## ACP란?

ACP(Agent Client Protocol)는 IDE와 같은 클라이언트가 AI 에이전트 세션을 조작하기 위한 표준 프로토콜입니다.
Poolside CLI는 ACP 클라이언트로서 외부 에이전트 서버(Claude Agent, Codex, Gemini 등)에 연결하거나,
내장 에이전트를 ACP 서버로 실행합니다.

## 아키텍처

```
┌─────────────┐     ACP over stdio      ┌──────────────┐     HTTP/WebSocket    ┌─────────────┐
│   IDE       │ ◄─────────────────────── ►│  pool acp     │ ◄──────────────────► │  에이전트    │
│ (Zed, etc.) │     Agent Client Protocol │  (브릿지)     │     (agent server)  │  (백엔드)    │
└─────────────┘                          └──────────────┘                      └─────────────┘
```

1. **IDE 클라이언트**: ACP 프로토콜을 지원하는 IDE (Zed, JetBrains, Neovim 등)
2. **ACP 브릿지**: `pool acp` 명령어. stdio로 ACP 메시지를 받고 에이전트 서버로 변환
3. **에이전트 서버**: Poolside 내장 에이전트 또는 외부 에이전트 (Claude Agent, Codex, Gemini 등)

## 핵심 목표

- **최소한의 ACP 표면**: stdio 기본, Streamable HTTP 실험적 지원
- **다양한 에이전트 지원**: Poolside 내장 에이전트 + 외부 ACP 호환 에이전트
- **설정 기반 통합**: `settings.yaml`의 `agent_servers`로 에이전트 구성
- **샌드박스 지원**: `--sandbox`로 격리된 에이전트 실행

## 구성 요소

### 1. ACP 서버 (브릿지)

`pool acp` 명령어에서 구현됩니다.

- 기본: stdio를 통해 ACP NDJSON 스트림 처리
- 실험적: `pool acp serve`로 Streamable HTTP 지원 (기본 포트 3284)

### 2. 에이전트 서버 구성

`settings.yaml`의 `agent_servers` 섹션에서 구성됩니다.

- `type`: `custom`, `registry`, `remote`
- `command`: 로컬 stdio 서버 실행 파일
- `url`: 원격 ACP 서버 URL (Streamable HTTP)
- `args`: 명령어 인수
- `env`: 환경 변수
- `default_config_options`: 세션 기본값

### 3. 세션 관리

- `pool acp serve`는 각 연결마다 새 에이전트 인스턴스 생성
- 세션 기본값은 `default_config_options`에서 구성
- thought depth는 제공자에 따라 `max`, `none`, 또는 모델 메타데이터에 따름

## 데이터 흐름

### stdio 모드 (기본)

```
IDE → prompt() → pool acp → 에이전트 서버 → 에이전트 실행
                                                ↓
IDE ← agent_message_chunk ← pool acp ← 에이전트 스트리밍 이벤트
```

### Streamable HTTP 모드 (실험적)

```
IDE → POST /acp → pool acp serve → 에이전트 서버 → 에이전트 실행
                                                  ↓
IDE ← Streamable HTTP ← pool acp serve ← 에이전트 스트리밍 이벤트
```

## 에이전트 서버 연결

`pool`은 ACP 클라이언트로서 외부 에이전트 서버에 연결할 수 있습니다.

```bash
# Claude Agent 연결
pool --agent-server claude-agent-acp

# Codex 연결
pool --agent-server codex-acp

# Gemini 연결
pool --agent-server "gemini --acp"

# 원격 서버 연결
pool --agent-server http://localhost:3284/acp
```

`pool -s`로 대화식 메뉴에서 사용 가능한 설정을 선택할 수 있습니다.

## 인증

Poolside 인증은 다음 순서로 해결됩니다:

1. 환경 변수 (`POOLSIDE_API_KEY`, `POOLSIDE_TOKEN`)
2. 설정 파일 (`~/.config/poolside/settings.yaml`)
3. `pool login`으로 저장된 자격 증명
