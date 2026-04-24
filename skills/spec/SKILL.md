---
name: spec
description: >
  context-engineering Phase 3 — PRD(제품 요구사항) + SPEC(기술 명세) 작성.
  새로 작성하거나 기존 PRD·SPEC 재작업·갱신 시 사용.
  Keywords: PRD, SPEC, 요구사항 명세, 기술 명세, 구현 계획, spec
allowed-tools: Read, Write, Bash, Grep, Glob
---

# context-engineering:spec

PRD(제품 요구사항)와 SPEC(기술 명세)를 작성한다.

## 사용법

```
/context-engineering:spec [@<code-path>] [--output <output-path>]
```

## Context Resolution

경로를 다음 순서로 결정한다:

1. `@<code-path>` 인자 있음 → `{CODE_PATH}` = 해당 경로
2. 없음 → 현재 디렉토리에서 `docs/prd.md` 또는 `docs/spec.md` 탐색
3. 탐색 실패 → "프로젝트 경로를 알려주세요 (예: `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/`

## 기존 Artifacts 확인

**Knowledge Base 로드**: `{OUTPUT_PATH}/knowledge-base.md` 있으면 도메인 용어·제약을 컨텍스트로 활용한다.

**PRD·SPEC 탐지**: `{OUTPUT_PATH}/prd.md`, `{OUTPUT_PATH}/spec.md` 존재 시:

```
[현재 상태]
- prd.md: {존재하면 기능 N개, 없으면 "없음"}
- spec.md: {존재하면 아키텍처 결정 N개, 없으면 "없음"}

재작업(처음부터) 또는 갱신(기존 내용 유지·수정)?
```

## Phase 3-1: PRD

```markdown
# PRD — {PROJECT_NAME}

## 목적
{이 제품/기능이 해결하는 문제 — 한 문단}

## 사용 시나리오
{누가, 어떤 상황에서 사용하는가}

## 기능 요구사항
| 기능 | 성공 기준 | 우선순위 |
|------|---------|---------|
| {기능명}: {한 줄 설명} | {검증 가능한 기준} | High/Medium/Low |

## 제약
| 제약 | 영향 | 우선순위 |
|------|------|---------|
| {제약 설명} | {영향} | High/Medium/Low |
```

**산출물**: `{OUTPUT_PATH}/prd.md`

## Phase 3-2: SPEC

```markdown
# SPEC — {PROJECT_NAME}

## 아키텍처 결정
| 결정 사항 | 고려한 대안 | 선택 | 근거 |
|---------|-----------|------|------|
| {무엇을 결정했나} | {대안1}, {대안2} | {선택한 것} | {이유} |

## 패키지 구조
의존 방향: `진입점 → 애플리케이션 → 도메인 ← 인프라` (역방향 금지)
* 단순 프로젝트(CLI, 스크립트 등)는 이 구조 대신 프로젝트에 맞는 모듈 구조를 정의한다.

\`\`\`
{레이어}: {폴더 경로}  ← {담당 역할}
\`\`\`

## 데이터 흐름
\`\`\`
{입력} → {컴포넌트A} → {컴포넌트B} → {출력}
\`\`\`

## 구현 계획
| 순서 | 구현 대상 | 의존 | 병렬 가능 | 완료 기준 |
|------|---------|------|:-------:|---------|
| 1 | {모듈/기능} | — | — | {검증 가능한 기준} |
| 2 | {모듈/기능} | 1 | — | {검증 가능한 기준} |
```

**산출물**: `{OUTPUT_PATH}/spec.md`

> **완료**: "`prd.md`·`spec.md` 저장했습니다. `/context-engineering:valid`로 Readiness Gate를 체크하거나 `/context-engineering:impl`로 구현을 시작할 수 있습니다."
