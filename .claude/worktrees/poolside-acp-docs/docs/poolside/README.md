# Poolside CLI ACP (Agent Client Protocol) 문서

Poolside CLI (`pool`)의 ACP (Agent Client Protocol) 기능에 대한 문서입니다.
ACP는 IDE와 같은 클라이언트가 표준화된 프로토콜을 통해 AI 에이전트 세션을 조작할 수 있게 해주는 프로토콜입니다.

## 목차

- [개요](./overview.md) — ACP의 역할, 아키텍처, 핵심 개념
- [CLI 명령어](./cli-commands.md) — `pool acp` 명령어와 옵션
- [프로토콜 상세](./protocol.md) — ACP 메시지 구조, 세션 관리, 에러 처리
- [IDE 설정](./ide-setup.md) — Zed, JetBrains, Neovim 등 IDE 통합 방법
- [설정](./configuration.md) — `settings.yaml` 설정, `agent_servers`, 환경 변수

## 빠른 시작

1. Poolside CLI 설치 및 로그인
2. `settings.yaml`에 에이전트 서버 설정
3. IDE에서 `pool acp` 연결

```bash
# 로그인
pool login

# ACP 서버 실행
pool acp

# 또는 settings.yaml에 설정 후 pool 실행
```

## 핵심 개념

| 개념 | 설명 |
|---|---|
| **ACP 브릿지** | IDE의 ACP 프로토콜을 Poolside Gateway로 변환하는 레이어 |
| **에이전트 서버** | `settings.yaml`의 `agent_servers`에 정의된 외부 에이전트 |
| **메시지 형식** | stdio 또는 Streamable HTTP (실험적) |
| **샌드박스** | `--sandbox`로 동작 제어 (`required` 또는 `disabled`) |

## 공식 문서

- [Poolside 공식 ACP 문서](https://docs.poolside.ai/tools.md)
- [ACP 클라이언트 — 타 에이전트 서버 연결](https://docs.poolside.ai/cli/other-agent-servers.md)
- [설정 파일 레퍼런스](https://docs.poolside.ai/settings-file-reference.md)
- [문제 해결](https://docs.poolside.ai/cli/troubleshooting.md)
