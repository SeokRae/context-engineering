---
name: compose
description: >
  Phase 6: Phase 1 success criteria에 맞는 최종 출력물을 생성한다.
  build가 압축한 컨텍스트를 받아 지시문 / KB 엔트리 / 프로젝트 산출물 중 하나로 조립한다.
  Keywords: execution instruction, prompt, role task constraints output format, knowledge entry, CLAUDE.md, spec
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Compose (Phase 6)

최종 출력물 생성 단계. build의 구조화·압축된 컨텍스트와 Phase 1의 success criteria를 결합하여 목적에 맞는 출력물을 조립한다.

---

## 전제조건

실행 전 build 산출물 확인:

```
Phase 1 분석 결과 (purpose / constraints / success criteria / Role)
구조화·압축된 컨텍스트 블록 (Key Facts / Constraints / Decisions / Notes)
```

산출물이 없으면:
> "먼저 `/context-engineering:build`를 실행해주세요."
종료.

---

## Phase 6. 실행 지시 생성

**목적**: Phase 1 success criteria 신호에 맞는 출력 형식을 자동 결정하고 최종 출력물을 생성한다.

### 출력 형식 자동 결정

Phase 1의 success criteria 답변에서 신호를 읽어 형식을 결정한다:

| success criteria 신호 | 출력 형식 |
|-----------------------|----------|
| "지시해줘", "프롬프트 만들어줘", "어떻게 써야 해" | **A. 실행 지시문** |
| "저장해줘", "기억해줘", "노트해줘" | **B. KB 엔트리** |
| "프로젝트 시작", "개발하고 싶어", "코드베이스 분석" | **C. 프로젝트 산출물** |

신호가 모호하면 1회만 질문:
> "출력 형식을 선택해주세요: (A) 실행 지시문  (B) KB 엔트리  (C) 프로젝트 산출물"

---

### A. 실행 지시문

Role + Task + Constraints + Output Format 구조로 조립:

```markdown
## Role
{Phase 1 답변과 컨텍스트에서 추론한 AI의 역할}

## Task
{Phase 1 purpose — 해결하려는 문제를 구체적으로 서술}

## Constraints
{Phase 5의 Constraints 항목 전체}
{Phase 1 constraints에서 도출된 추가 제약}

## Output Format
{Phase 1 success criteria에서 정의한 결과물의 형태}
```

작성 원칙:
- Role: 한 문장, 구체적인 전문 역할로 기술
- Task: 모호함 없이 행동 가능한 수준으로 기술
- Constraints: 목록 형태, 각 항목은 단독으로 이해 가능
- Output Format: 형식·분량·구조를 명시

### Tool Context (실행 지시문 전용)

Phase 1 purpose에서 도구 사용이 필요한 작업이면 지시문에 `## Tools` 섹션을 추가한다:

```markdown
## Tools

| Tool | 용도 | 사용 예시 |
|------|------|---------|
| {tool name} | {이 작업에서의 용도} | {호출 예시 또는 패턴} |
```

MCP Tools (해당 시):
- context7: 라이브러리 문서 조회 — `resolve-library-id` → `get-library-docs`
- playwright: 웹 페이지 검증 — `browser_navigate` → `browser_snapshot`

**포함 기준** (Phase 1 purpose 신호 기반):
- "코드", "구현", "테스트" 신호 → Bash, Read, Write, Agent 포함
- "문서", "라이브러리" 신호 → context7 MCP 포함
- "웹", "브라우저", "UI" 신호 → playwright MCP 포함
- 도구 필요 신호 없으면 → `## Tools` 섹션 생략

### 컨텍스트 유지 지시 (실행 지시문 전용)

실행 지시문(A 형식)이 **3단계 이상** 작업을 지시하는 경우, 지시문 하단에 다음 블록을 추가한다:

    ## Context Checkpoints

    이 작업이 3단계 이상이면:
    1. 각 주요 단계 완료 후 `_context-session.md`에 기록:
       - 현재까지 완료한 것
       - 발견한 핵심 사실
       - 다음 단계에 필요한 정보
    2. 다음 단계 시작 전 `_context-session.md`를 다시 읽는다
    3. 모든 단계 완료 후 `_context-session.md`를 삭제한다

참조: [Context Session Template](../context-engineering/references/context-session-template.md)

---

### B. KB 엔트리

표준 지식 엔트리 형식으로 저장:

```markdown
---
title: {descriptive title}
domain: {domain-slug}
type: fact | decision | constraint | procedure | reference
source: user | document:{filename} | url:{https://...}
date: {YYYY-MM-DD}
reliability: high | medium | low
tags: [tag1, tag2]
related: []
---

# {Title}

> {one-line summary}

## Key Facts
{Phase 4의 Key Facts 항목}

## Constraints
{Phase 4의 Constraints 항목}

## Decisions
{Phase 4의 Decisions 항목}

## Notes
{Phase 4의 Notes 항목}
```

저장 경로: `{KNOWLEDGE_PATH}/{domain}/{slugified-title}.md`

저장 후 `index.md` 업데이트:
- 해당 domain 테이블에 행 추가
- domain 섹션 없으면 알파벳 순서로 신규 섹션 생성
- 헤더 카운트 재계산 (Total entries, Domains)
- 날짜 내림차순 정렬 유지

참조: [Entry Template](../context-engineering/references/entry-template.md)

---

### C. 프로젝트 산출물

프로젝트 개발을 위한 3가지 문서를 생성:

**CLAUDE.md** (AI 행동 정책):

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {한줄 프로젝트 설명}

## Architecture
{Phase 4 Key Facts + Decisions 기반 아키텍처 설명}
{레이어 구조 테이블}

## Build & Test
{빌드·테스트 명령어}

## Core Constraints
{Phase 4 Constraints 항목 — 최대 5개, TC/BL 우선}

## Non-obvious Gotchas
{Phase 4 Notes 중 놓치기 쉬운 함정}

## References
- [Knowledge Base]({OUTPUT_PATH}/knowledge-base.md)
```

참조: [CLAUDE.md Template](../context-engineering/references/claude-md-policy-template.md)

**spec.md** (기술 명세):

```markdown
# {PROJECT_NAME} Spec

## Architecture Decisions
| Decision | Alternatives | Choice | Rationale |

## Package Structure
{레이어별 패키지 구조}

## Implementation Plan
| # | Item | REQ | Depends | Done Criteria |

## Requirements Traceability
| REQ-ID | Feature | Implementation | Status |
```

참조: [Spec Template](../context-engineering/references/spec-template.md)

**구현 계획**: 첫 번째 구현 항목을 즉시 시작할 수 있는 수준으로 기술.

---

## G6 게이트

compose 완료 후 AI가 자동 평가:

| 기준 | 평가 |
|------|------|
| success criteria 일치 | 출력물이 Phase 1에서 정의한 결과물의 형태와 일치하는가 |
| 누락 없음 | Phase 1 purpose/constraints가 출력에 모두 반영됐는가 |
| 충돌 없음 | 출력 내 모순이 없는가 |
| 추측 없음 | 근거 없는 주장이 포함되지 않았는가 |

**G6 통과**: 완료 — 출력물 제시

**G6 실패**: Phase 7 자동 호출
```
G6 미충족: {기준} — {이유}
verify를 실행하여 상세 검증을 진행합니다.
```
