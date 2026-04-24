---
name: impl
description: >
  context-engineering Phase 4 — 단계별 구현.
  SPEC 구현 계획 기반으로 모듈 단위 구현 + 완료 기준 검증 + 피드백 루프 반복.
  Keywords: 구현, implementation, Phase 4, 빌드, 테스트, impl
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# context-engineering:impl

SPEC 구현 계획을 기반으로 단계별 구현을 진행한다.

## 사용법

```
/context-engineering:impl [@<code-path>] [--output <output-path>]
```

## Context Resolution

경로를 다음 순서로 결정한다:

1. `@<code-path>` 인자 있음 → `{CODE_PATH}` = 해당 경로
2. 없음 → 현재 디렉토리에서 `docs/spec.md` 탐색
3. 탐색 실패 → "프로젝트 경로를 알려주세요 (예: `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/`

## SPEC 확인

`{OUTPUT_PATH}/spec.md` 있으면 구현 계획 테이블을 출력하고 시작할 항목을 확인한다:

```
[구현 계획]
| 순서 | 구현 대상 | 의존 | 병렬 가능 | 완료 기준 |
...

어느 항목부터 시작할까요? (번호 또는 "처음부터")
```

없으면: "SPEC 없이 진행하면 탐색 모드로 시작됩니다 — 계속할까요?"

## 구현 단위별 반복

각 구현 단위 완료 시 체크리스트 확인:

- [ ] `{BUILD_CMD}` 성공
- [ ] `{TEST_CMD}` 통과 (신규 + 기존 회귀 없음)
- [ ] SPEC 아키텍처 결정 준수 (SPEC 비어있으면 생략)
- [ ] PRD 기능별 성공 기준 달성 확인 (빌드 성공과 별개로 PRD 기준 직접 확인)
- [ ] 발견 사항 해당 단계 문서 반영 완료

## 피드백 루프

구현 중 현실과 맞지 않는 내용 발견 시:

| 발견 내용 | 갱신 대상 | 명령 |
|---------|---------|------|
| 도메인 용어 오류, 새 제약 발견 | `knowledge-base.md` | `/context-engineering:gather` |
| 새 gotcha 발견, AI 행동 규칙 변경 | `CLAUDE.md` | `/context-engineering:gather` |
| 기능 요구사항 변경 | `prd.md` | `/context-engineering:spec` |
| 아키텍처 결정 번복, 패키지 구조 변경 | `spec.md` | `/context-engineering:spec` |

> **원칙**: 구현을 문서에 맞추지 말고, 문서를 현실에 맞춰 갱신한다.
