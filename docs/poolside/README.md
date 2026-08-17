# Poolside CLI ACP (Agent Client Protocol) 문서

Poolside CLI의 ACP (Agent Client Protocol) 브릿지 기능에 대한 문서입니다.
ACP는 IDE와 같은 클라이언트가 표준화된 프로토콜을 통해 AI 에이전트 세션을 조작할 수 있게 해주는 프로토콜입니다.

## 목차

- [개요](./overview.md) — ACP의 역할, 아키텍처, 핵심 개념
- [CLI 명령어](./cli-commands.md) — `openclaw acp` 명령어와 옵션
- [프로토콜 상세](./protocol.md) — ACP API 엔드포인트, 메시지 구조, 세션 관리
- [Gateway 연동](./gateway-integration.md) — Gateway와의 통신, 세션 매핑, 이벤트 변환
- [IDE 설정](./ide-setup.md) — Zed, Claude Code 등 IDE 통합 방법

## 빠른 시작

1. Gateway 실행 (로컬 또는 원격)
2. Gateway 타겟 설정 (URL + 인증)
3. IDE에서 `openclaw acp`를 stdio로 연결

```bash
# Gateway 설정
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>

# ACP 브릿지 실행
openclaw acp --url wss://gateway-host:18789 --token <token>
```

## 핵심 개념

| 개념 | 설명 |
|---|---|
| **ACP 브릿지** | IDE의 ACP 프로토콜을 Gateway WebSocket으로 변환하는 레이어 |
| **세션 키** | Gateway 세션을 식별하는 키 (`acp:<uuid>`, `agent:main:main` 등) |
| **메시지 형식** | NDJSON (Newline-Delimited JSON) over stdio |
| **에이전트 정보** | `openclaw-acp` / `OpenClaw ACP Gateway` / 버전 |
