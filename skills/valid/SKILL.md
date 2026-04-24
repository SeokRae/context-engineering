---
name: valid
description: >
  context-engineering Readiness Gate 체크.
  Phase 3 완료 후 Phase 4 진입 전, 또는 탐색 모드 종료 시점에 사용.
  AI는 현재 상태 요약만 제시하고 사용자가 통과 여부를 결정한다.
  Keywords: readiness gate, 검증, valid, gate, 스파이럴 종료, 체크포인트
allowed-tools: Read, Bash, Glob
---

# context-engineering:valid

현재 프로젝트 artifacts 상태를 점검하고 Readiness Gate 결과를 사용자에게 제시한다.

## 사용법

```
/context-engineering:valid [@<code-path>] [--output <output-path>]
```

## Context Resolution

경로를 다음 순서로 결정한다:

1. `@<code-path>` 인자 있음 → `{CODE_PATH}` = 해당 경로
2. 없음 → 현재 디렉토리에서 `docs/prd.md` 또는 `docs/spec.md` 탐색
3. 탐색 실패 → "프로젝트 경로를 알려주세요 (예: `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/`

## Artifacts 탐색

다음 파일을 읽어 현재 상태를 파악한다:
- `{OUTPUT_PATH}/knowledge-base.md` — 도메인 용어·제약
- `{CODE_PATH}/CLAUDE.md` — 정책
- `{OUTPUT_PATH}/prd.md` — 기능 요구사항·성공 기준
- `{OUTPUT_PATH}/spec.md` — 아키텍처 결정·구현 계획

## Readiness Gate 출력

**HARD-GATE (사용자 확인 필수)**: AI는 각 기준의 현재 상태만 요약한다. 통과 여부는 사용자가 결정한다. AI가 스스로 "통과"를 선언하지 않는다.

```
[Readiness Gate]
① PRD의 모든 기능에 검증 가능한 성공 기준이 있는가?
   → {기능 N개 중 M개 성공 기준 기재됨 / 미기재 항목: ...}
② SPEC의 아키텍처 결정에 대안과 근거가 있는가?
   → {결정 N개 중 M개 근거 있음 / 근거 없는 항목: ...}
③ 도메인 용어 미확정 항목이 없는가?
   → {미확정 N개 — 항목명 나열 / 없으면 "없음"}
④ CLAUDE.md가 실제 제약을 반영하는가?
   → {Phase 1 제약 N개 중 CLAUDE.md 반영 M개}
⑤ SPEC 구현 계획의 첫 항목을 지금 바로 시작할 수 있는가?
   → {첫 항목명 + 의존 관계}

미충족 항목이 있다면 해당 명령을 사용해 주세요.
모두 통과라면 "Phase 4 진행" 또는 "/context-engineering:impl" 이라고 알려 주세요.
```

| 기준 | 미충족 시 |
|------|---------|
| ① PRD 성공 기준 누락 | `/context-engineering:spec` 으로 PRD 보완 |
| ② SPEC 근거 누락 | `/context-engineering:spec` 으로 SPEC 보완 |
| ③ 도메인 용어 미확정 | `/context-engineering:gather` 로 KB 보완 |
| ④ CLAUDE.md 제약 미반영 | `/context-engineering:gather` 로 CLAUDE.md 보완 |
| ⑤ 구현 계획 착수 불가 | `/context-engineering:spec` 으로 SPEC 보완 |
