---
name: gather
description: >
  context-engineering Step 0 + Phase 1 (Knowledge Base) + Phase 2 (CLAUDE.md).
  새 프로젝트 시작 또는 기존 KB·CLAUDE.md 재작업·갱신 시 사용.
  Keywords: context gathering, knowledge base, CLAUDE.md, 요구사항 탐색, 도메인 지식, gather
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# context-engineering:gather

프로젝트 컨텍스트 수집(Step 0) → Knowledge Base(Phase 1) → CLAUDE.md(Phase 2)를 진행한다.

## 사용법

```
/context-engineering:gather [@<code-path>] [--output <output-path>]
```

## Context Resolution

경로를 다음 순서로 결정한다:

1. `@<code-path>` 인자 있음 → `{CODE_PATH}` = 해당 경로
2. 없음 → 현재 디렉토리에서 `docs/knowledge-base.md` 탐색
3. 탐색 실패 → "프로젝트 경로를 알려주세요 (예: `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/`

## 기존 Artifacts 확인

`{OUTPUT_PATH}/knowledge-base.md` 또는 `{CODE_PATH}/CLAUDE.md` 존재 시:

```
[현재 상태]
- knowledge-base.md: {존재하면 용어 N개·제약 N개, 없으면 "없음"}
- CLAUDE.md: {존재 여부}

재작업(처음부터) 또는 갱신(기존 내용 유지·수정)?
```

- **재작업** → Step 0부터 시작
- **갱신** → 파일 로드 후 수정할 부분만 진행

## Step 0: Context Gathering

질문은 한 번에 하나씩 던진다. 사용자 답변 후 다음 질문으로 넘어간다.

1. "이번에 만들려는 것이 무엇인지 한 문장으로 설명해 주세요."
2. "이것을 왜 만들어야 하나요? 해결하려는 문제나 달성하려는 목표가 무엇인가요?"
3. "지금 알고 있는 제약 조건이 있나요? (기술 스택 고정, 마감일, 연동 시스템, 성능 요구사항 등)"

3가지 답변 후 요약 출력 + 사용자 확인:

```
[요구사항 요약]
- 무엇: {요약}
- 왜: {요약}
- 제약: {요약 또는 "없음"}

이 내용이 맞나요?
```

`@<code-path>` 있으면 빌드 파일에서 기술 스택 자동 감지:
`build.gradle` / `pom.xml` / `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod`

## Phase 1: Knowledge Base

### 시나리오 판별

| 시나리오 | 조건 | 설명 |
|---------|------|------|
| A. 그린필드 | 스펙 없음 + 코드 없음 | 새 프로젝트 |
| B. 스펙 우선 | 스펙 있음 + 코드 없음 | 문서만 있음 |
| C. 코드 우선 | 스펙 없음 + 코드 있음 | 코드만 있음 |
| D. 풀 컨텍스트 | 스펙 있음 + 코드 있음 | 둘 다 있음 |

혼합 상태(스펙이 구식, 코드 일부만 문서화 등)는 가장 가까운 시나리오를 선택하고
불확실 사항을 명시한 후 사용자 확인을 받는다:

```
[Context Assessment] 시나리오 {A/B/C/D} ({시나리오명}) — {판별 근거}
불확실 사항: {있으면 명시}

이 판별이 맞나요?
```

### Knowledge Base 작성

도메인 용어, 제약, 참조 문서를 수집해 `{OUTPUT_PATH}/knowledge-base.md`를 작성한다.

**도메인 용어 (Glossary)**:
```
| 용어 | 정의 | 출처 |
|------|------|------|
| {용어} | {한 줄 정의} | {문서명 또는 "대화"} |
```

**제약 목록**:
```
| ID | 분류 | 설명 | 영향 | 우선순위 |
|----|------|------|------|---------|
| TC-1 | 기술 | {제약} | {영향} | High |
| BL-1 | 비즈니스 | {규칙} | {영향} | Medium |
| OC-1 | 조직 | {일정·접근 제한} | {영향} | Low |
```

분류: `TC` 기술 제약, `BL` 비즈니스 규칙, `OC` 조직 제약

**산출물**: `{OUTPUT_PATH}/knowledge-base.md`

> **게이트**: "Phase 1 완료 — `knowledge-base.md` 저장했습니다. 검토 후 Phase 2 진행할까요?"

## Phase 2: CLAUDE.md 작성

> **아키텍처 섹션**: 복잡한 도메인 서비스 → Hexagonal + DDD. 단순 스크립트·파이프라인 → 모듈 구조만 기술.

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {한 줄 설명}

## 아키텍처

{핵심 패턴 3~5가지 — WHY 포함 / 단순 프로젝트는 모듈 구조만 기술}

## 빌드 & 테스트

\`\`\`bash
{BUILD_CMD}
{TEST_CMD}
\`\`\`

## 핵심 제약

{Phase 1 TC/BL 중 개발에 직접 영향을 주는 항목 — 최대 5개}

## Non-obvious Gotchas

{구현하며 발견 예정 — Phase 4 피드백 루프에서 갱신}

## 참고 문서

- [Knowledge Base]({OUTPUT_PATH}/knowledge-base.md): Phase 1 도메인 지식 베이스
```

> **필수**: `knowledge-base.md` 링크를 참고 문서 섹션에 반드시 포함한다.
> **초안 성격**: Non-obvious Gotchas는 구현해봐야 알 수 있으므로 Phase 4에서 갱신한다.

**산출물**: `{CODE_PATH}/CLAUDE.md`

> **완료**: "gather 완료 — `/context-engineering:spec`으로 PRD·SPEC을 작성하거나 `/context-engineering:valid`로 Readiness Gate를 체크할 수 있습니다."
