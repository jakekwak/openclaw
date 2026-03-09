---
summary: "ACP 코딩 에이전트를 스레드 가능한 채널에 1급 ACP 제어 평면으로 통합하는 계획"
owner: "onutc"
status: "draft"
last_updated: "2026-02-25"
title: "ACP Thread Bound Agents"
---

# ACP Thread Bound Agents

## 개요

이 계획은 OpenClaw가 ACP 코딩 에이전트를 스레드 가능한 채널(우선 Discord)에서 프로덕션 수준의 생명주기와 복구 의미 체계로 지원하는 방식을 정의합니다.

관련 문서:

- [Unified Runtime Streaming Refactor Plan](/ko-KR/experiments/plans/acp-unified-streaming-refactor)

목표 UX:

- 사용자가 ACP 세션을 스레드에 생성하거나 포커스
- 해당 스레드의 후속 메시지가 바인딩된 ACP 세션으로 라우팅
- 에이전트 출력이 같은 스레드 페르소나로 다시 스트리밍
- 세션은 영속형 또는 일회성으로 운용 가능

## 결정 요약

장기적으로는 하이브리드 아키텍처를 권장합니다.

- OpenClaw 코어가 ACP 제어 평면을 소유
  - 세션 ID와 메타데이터
  - 스레드 바인딩과 라우팅 결정
  - 전달 불변식과 중복 억제
  - 정리 및 복구 의미 체계
- ACP 런타임 백엔드는 플러그형
  - 첫 백엔드는 `acpx`
  - 런타임은 ACP 전송, 큐잉, 취소, 재연결 담당

## 북극성 아키텍처

ACP를 OpenClaw의 1급 제어 평면으로 다루고, 런타임 어댑터는 교체 가능하게 둡니다.

핵심 불변식:

- 모든 ACP 스레드 바인딩은 유효한 ACP 세션 레코드를 가리켜야 함
- 모든 ACP 세션은 명시적 상태(`creating`, `idle`, `running`, `cancelling`, `closed`, `error`)를 가짐
- 모든 ACP 실행은 명시적 run 상태(`queued`, `running`, `completed`, `failed`, `cancelled`)를 가짐
- 생성, 바인딩, 최초 enqueue는 원자적으로 처리
- 명령 재시도는 idemopotent 해야 함
- 바인딩된 스레드 출력은 ACP 이벤트의 투영이어야 하며 임의 부작용이 아니어야 함

## 왜 플러그인 전용으로는 안 되는가

현재 플러그인 훅만으로는 종단간 ACP 세션 라우팅을 구현하기 어렵습니다.

- 스레드 바인딩 기반 인바운드 라우팅은 코어 디스패치에서 먼저 해석됩니다.
- 메시지 훅은 fire-and-forget이며 메인 응답 경로를 단락시키지 못합니다.
- 플러그인 명령은 제어 작업에는 적합하지만 코어 턴 디스패치를 대체하기엔 부족합니다.

결론:

- ACP 런타임은 플러그인화 가능
- ACP 라우팅 분기는 코어에 있어야 함

## 아키텍처 요약

코어가 담당:

- ACP 세션 모드 디스패치 분기
- 전달 중복 방지
- ACP 제어 평면 영속화
- 세션 reset/delete와 연결된 정리

플러그인 백엔드가 담당:

- ACP 워커 관리
- `acpx` 프로세스 호출 및 이벤트 파싱
- `/acp ...` 명령과 운영자 UX
- 백엔드별 진단 및 기본값

## 장기 영속화 모델

장기적인 단일 소스는 WAL 모드의 전용 ACP SQLite 데이터베이스입니다.

- `acp_sessions`
- `acp_runs`
- `acp_bindings`
- `acp_events`
- `acp_delivery_checkpoint`
- `acp_idempotency`

이 구조는 트랜잭션 업데이트, 크래시 복구, 재생성 가능한 이벤트 전달을 목표로 합니다.
