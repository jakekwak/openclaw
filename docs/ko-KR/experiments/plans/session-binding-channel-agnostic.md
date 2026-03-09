---
summary: "채널 비종속 세션 바인딩 아키텍처와 1차 구현 범위"
read_when:
  - 채널 비종속 세션 라우팅과 바인딩을 리팩터링할 때
  - 채널 간 중복/누락/오래된 세션 전달을 조사할 때
owner: "onutc"
status: "in-progress"
last_updated: "2026-02-21"
title: "Session Binding Channel Agnostic Plan"
---

# Session Binding Channel Agnostic Plan

## 개요

이 문서는 장기적인 채널 비종속 세션 바인딩 모델과 다음 구현 반복에서 다룰 구체적인 범위를 정의합니다.

목표:

- 서브에이전트 바인딩 세션 라우팅을 코어 기능으로 만들기
- 채널별 동작은 어댑터에 남기기
- 일반 Discord 동작 회귀 방지

## 왜 필요한가

현재 동작은 다음을 뒤섞고 있습니다.

- completion 콘텐츠 정책
- 목적지 라우팅 정책
- Discord 전용 세부사항

그 결과:

- 동시 실행 시 메인 채널과 스레드 중복 전달
- 재사용된 바인딩 매니저의 오래된 토큰 사용
- webhook 전송의 활동 기록 누락

## 1차 범위

### 1. 채널 비종속 코어 인터페이스 추가

- 바인딩과 라우팅을 위한 코어 타입/서비스 추가
- `BindingTargetKind = "subagent" | "session"`
- `BindingStatus = "active" | "ending" | "ended"`

### 2. 서브에이전트 completion용 코어 전달 라우터 추가

- completion 이벤트에 대한 단일 목적지 해석 경로 도입
- 이번 단계에서는 `task_completion`만 새 경로를 사용

### 3. Discord는 어댑터로 유지

- 스레드 생성/재사용
- webhook 또는 채널 전송
- 스레드 상태 검증
- Discord 메타데이터 매핑

### 4. 알려진 정확성 문제 수정

- 기존 thread binding manager 재사용 시 최신 토큰 사용
- Discord webhook 전송의 활동 타임스탬프 기록
- 바인딩된 목적지를 선택했을 때 메인 채널로의 암묵적 fallback 제거

### 5. 현재 안전 기본값 유지

- `channels.discord.threadBindings.spawnSubagentSessions = false` 기본값 유지

## 이번 단계에 포함되지 않는 것

- ACP 바인딩 타깃(`targetKind: "acp"`)
- Discord 외 새 채널 어댑터
- 모든 전달 경로 전체 교체
- 프로토콜 수준 변경
- 바인딩 저장소 전체 마이그레이션 재설계
