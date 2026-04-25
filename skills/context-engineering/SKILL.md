---
name: context-engineering
description: >
  7-Phase Context Engineering 파이프라인 — AI에게 보낼 컨텍스트를 잘 만든다.
  문제 정의 → 후보 수집 → 선택 → 구조화 → 압축 → 실행 지시 생성 → 검증.
  Keywords: context engineering, ai context, prompt, knowledge base, CLAUDE.md, spec, 7-phase
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# Context Engineering

AI에게 보낼 컨텍스트를 체계적으로 만드는 7-Phase 파이프라인. 단일 흐름으로 지식 축적, 프롬프트 조립, 프로젝트 문서 생성을 모두 처리한다.

---

## 전체 흐름

```
[입력]
  ↓
Phase 1 →[G1]→ Phase 2 →[G2]→ Phase 3 →[G3]
                                          ↓
                         Phase 4 →[G4]→ Phase 5 →[G5]
                                                   ↓
                                         Phase 6 →[G6]→ [출력]
                                                          ↓ (G6 실패)
                                                        Phase 7
```

---

## 게이트 동작 방식

각 Phase 완료 후 AI가 기준을 자동 평가:

```
[자동 통과]  기준 명확히 충족 → 사용자 표시 없이 다음 Phase 진행

[사용자 확인] 모호하거나 기준 미충족 →
  "Phase N 결과: {요약}
   미충족 항목: {내용}
   수정하거나 승인 후 진행해주세요."
```

---

## 서브 스킬 진입점

각 단계를 개별적으로 실행할 수 있다:

| 명령 | 담당 Phase | 용도 |
|------|-----------|------|
| `/context-engineering:gather` | Phase 1-3 | 문제 정의 → 후보 수집 → 선택 |
| `/context-engineering:build` | Phase 4-5 | 구조화 → 압축 |
| `/context-engineering:compose` | Phase 6 | 실행 지시 생성 |
| `/context-engineering:verify` | Phase 7 | 최종 검증 (재실행 가능) |

**중간 진입 조건**: 이전 서브 스킬 산출물이 존재할 때만 허용.
산출물 없으면 먼저 실행할 서브 스킬을 안내한다.

---

## 전체 실행 (`/context-engineering`)

서브 스킬을 순서대로 호출한다:

### 1단계: gather 실행
`/context-engineering:gather` — Phase 1-3 + G1-G3

완료 기준:
- Phase 1 분석 결과 (purpose / constraints / success criteria / Role / scope-declaration)
- Phase 3 선택 결과 (Keep 항목 목록)

### 2단계: build 실행
`/context-engineering:build` — Phase 4-5 + G4-G5

완료 기준:
- 구조화·압축된 컨텍스트 블록 (Key Facts / Constraints / Decisions / Notes)

### 3단계: compose 실행
`/context-engineering:compose` — Phase 6 + G6

완료 기준:
- G6 통과: 최종 출력물 (지시문 / KB 엔트리 / 프로젝트 산출물)
- G6 실패: Phase 7로 자동 이동

### 4단계: verify 실행 (G6 실패 시)
`/context-engineering:verify` — Phase 7

완료 기준:
- 신뢰도 표시 (H/M/L)
- 문제 유형별 피드백 루프 안내
- 자가 채점 루브릭 점수 (0-100)
- `_verify-log.md`에 결과 기록

---

## 출력물 유형

Phase 1 success criteria에 따라 자동 결정:

| 목적 | 출력물 |
|------|--------|
| 실행 지시문 | Role + Task + Constraints + Output Format |
| 지식 저장 | KB 엔트리 + index.md 업데이트 |
| 프로젝트 개발 | CLAUDE.md + spec.md + 구현 계획 |

---

## 스킬 수정 가이드라인

1. **Phase 일관성**: gather → build → compose → verify 흐름을 유지할 것
2. **게이트 형식**: `"Phase N 결과: {요약}\n 미충족 항목: {내용}\n 수정하거나 승인 후 진행해주세요."` 형식 준수
3. **템플릿 동기화**: `references/` 템플릿과 SKILL.md 인라인 구조를 항상 일치시킬 것
4. **컴팩션 생존**: 각 SKILL.md는 첫 5,000토큰 내에 핵심 행동 계약을 완결할 것. 보조 정보는 뒤에 배치하여 컴팩션 시 잘려도 핵심 동작에 영향 없게 한다.

## 검증

스킬 수정 후 실제 프로젝트에서 `/context-engineering @<path>` 실행하여 Phase 1 질의가 올바르게 시작되는지 확인.
