---
summary: "main, subagent, ACP 전반에 하나의 공통 런타임 스트리밍 파이프라인을 도입하는 계획"
owner: "onutc"
status: "draft"
last_updated: "2026-02-25"
title: "Unified Runtime Streaming Refactor Plan"
---

# Unified Runtime Streaming Refactor Plan

## 목표

`main`, `subagent`, `acp`가 모두 동일한 코얼레싱, 청킹, 전달 순서, 크래시 복구 동작을 공유하도록 하나의 스트리밍 파이프라인을 제공합니다.

## 필요성

- 현재 동작이 런타임별 경로로 갈라져 있습니다.
- 한 경로에서 고친 포맷/코얼레싱 버그가 다른 경로에는 남을 수 있습니다.
- 전달 일관성, 중복 억제, 복구 의미 체계를 이해하기 어렵습니다.

## 목표 아키텍처

하나의 공통 파이프라인 + 런타임별 어댑터:

1. 런타임 어댑터는 표준 이벤트만 방출
2. 공통 스트림 어셈블러가 텍스트/도구/상태 이벤트를 합치고 마무리
3. 공통 채널 프로젝터가 채널별 청킹/포맷팅을 한 번 적용
4. 공통 전달 ledger가 idemopotent send/replay를 보장
5. 아웃바운드 채널 어댑터가 실제 전송과 체크포인트 기록 수행

표준 이벤트 계약:

- `turn_started`
- `text_delta`
- `block_final`
- `tool_started`
- `tool_finished`
- `status`
- `turn_completed`
- `turn_failed`
- `turn_cancelled`

## 작업 흐름

### 1. 표준 스트리밍 계약

- 엄격한 이벤트 스키마와 검증 도입
- 어댑터 계약 테스트 추가
- 잘못된 런타임 이벤트를 조기에 거부하고 구조화된 진단 제공

### 2. 공통 스트림 처리기

- 런타임별 coalescer/projector를 하나의 프로세서로 대체
- 텍스트 버퍼링, idle flush, max-chunk 분할, completion flush를 일원화

### 3. 공통 채널 프로젝션

- 채널 어댑터는 최종 블록만 받아 전송
- Discord 특수 청킹은 채널 프로젝터로만 이동

### 4. 전달 ledger + replay

- 턴/청크별 전달 ID 추가
- 전송 전후 체크포인트 기록
- 재시작 시 보류 중인 청크를 중복 없이 재생

### 5. 마이그레이션과 전환

- 1단계: shadow mode
- 2단계: 런타임별 점진 전환
- 3단계: 기존 런타임별 스트리밍 코드 제거

## 비목표

- ACP 권한/정책 모델 변경 없음
- 채널별 기능 확대 없음
- 이벤트 호환성 외의 ACP 전송/백엔드 재설계 없음
