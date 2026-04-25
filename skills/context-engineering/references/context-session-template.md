---
name: context-session-template
description: 다단계 작업 중 컨텍스트 유지를 위한 세션 스크래치 파일 템플릿
type: reference
---

# Context Session Template

> 다단계 작업(3단계 이상) 중 컨텍스트를 유지하기 위한 임시 스크래치 파일.
> compose Phase 6에서 실행 지시문(A 형식)이 Context Checkpoints 블록을 포함할 때 사용된다.
> **작업 완료 후 반드시 삭제할 것.**

## 사용 방법

1. 작업 시작 시 이 템플릿을 복사하여 `_context-session.md`로 저장
2. 각 주요 단계 완료 후 해당 섹션을 업데이트
3. 다음 단계 시작 전 파일 전체를 다시 읽기
4. 모든 단계 완료 후 파일 삭제

---

# Context Session — {Task Name}

> 시작: {YYYY-MM-DD HH:mm}
> 목적: {Phase 1 purpose 한 줄 요약}

## 완료된 단계

| # | 단계 | 핵심 발견 | 완료 시각 |
|---|------|---------|---------|
| | | | |

## 현재 컨텍스트

{현재 단계에 필요한 핵심 정보}

## 미처리 항목

{나중에 처리할 항목 또는 다음 단계 의존성}
