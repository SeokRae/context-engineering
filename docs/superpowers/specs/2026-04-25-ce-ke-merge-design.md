# CE + KE 통합 설계: 단일 7-Phase Context Engineering 파이프라인

**날짜**: 2026-04-25
**목적**: 두 개의 독립 서브시스템(CE 4-phase, KE 7-phase)을 단일 7-phase 파이프라인으로 통합

---

## 배경

### 현재 문제

플러그인에 두 개의 독립 서브시스템이 공존:
- **CE (4-phase)**: 소프트웨어 개발 워크플로우 (knowledge-base.md, CLAUDE.md, spec.md, 코드)
- **KE (7-phase)**: 범용 지식 관리 엔진 (KB 엔트리, 인덱스, 답변)

두 시스템의 핵심 개념(수집, 선택, 구조화, 압축, 검증)이 중복되어, 한 쪽을 수정하면 다른 쪽도 동기화해야 하는 유지보수 부담이 증가하는 구조.

### 핵심 원칙

> **컨텍스트 엔지니어링 = AI에게 보낼 컨텍스트를 잘 만드는 것**

두 서브시스템 모두 이 목적을 다른 레이어에서 수행하고 있으며, 단일 파이프라인으로 통합 가능.

---

## 아키텍처

### 파일 구조

**현재 (8개):**
```
skills/
  context-engineering/SKILL.md   ← CE 오케스트레이터
  gather/SKILL.md                 ← CE Step 0 + Phase 1-2
  spec/SKILL.md                   ← CE Phase 3
  valid/SKILL.md                  ← CE Readiness Gate
  impl/SKILL.md                   ← CE Phase 4
  ke/SKILL.md                     ← KE 7-phase 엔진
  ingest/SKILL.md                 ← KE ingest 직접 진입
  query/SKILL.md                  ← KE query 직접 진입
```

**신규 (5개):**
```
skills/
  context-engineering/SKILL.md   ← 오케스트레이터 (전체 파이프라인)
  gather/SKILL.md                 ← Phase 1-3 + 게이트 G1-G3
  build/SKILL.md                  ← Phase 4-5 + 게이트 G4-G5  [신규]
  compose/SKILL.md                ← Phase 6 + 게이트 G6        [신규]
  verify/SKILL.md                 ← Phase 7 holistic 검증      [재작성]
```

### 파일 변경

| 파일 | 처리 |
|------|------|
| `context-engineering/SKILL.md` | 오케스트레이터로 재작성 |
| `gather/SKILL.md` | Phase 1-3 + G1-G3으로 재작성 |
| `build/SKILL.md` | **신규** — Phase 4-5 + G4-G5 |
| `compose/SKILL.md` | **신규** — Phase 6 + G6 |
| `verify/SKILL.md` | 재작성 — holistic 검증 |
| `spec/SKILL.md` | **삭제** (build로 흡수) |
| `valid/SKILL.md` | **삭제** (verify로 흡수) |
| `impl/SKILL.md` | **삭제** (compose로 흡수) |
| `ke/SKILL.md` | **삭제** (파이프라인 전체로 통합) |
| `ingest/SKILL.md` | **삭제** (gather로 흡수) |
| `query/SKILL.md` | **삭제** (gather + compose로 흡수) |

---

## 7-Phase 파이프라인

### 전체 흐름

```
[입력]
  ↓
Phase 1 →[G1]→ Phase 2 →[G2]→ Phase 3 →[G3]
                                          ↓
                         Phase 4 →[G4]→ Phase 5 →[G5]
                                                   ↓
                                         Phase 6 →[G6]→ [출력]
                                                          ↓ (G6 실패 시)
                                                        Phase 7
```

---

### Phase 1. 문제 정의

**목적**: 사용자 의도 명확화 → 이후 전체 Phase의 기준 수립

**방식**: 브레인스토밍 스킬 질의 패턴 채택 (purpose → constraints → success criteria)
- 질문은 하나씩
- 가능하면 객관식
- 최대 3개 + 모호 시 1개 추가 허용

**3개 핵심 질문:**
1. "지금 해결하려는 문제가 무엇인가요?" → **Task** 확정
2. "AI가 이 작업을 잘 하려면 어떤 배경 정보가 필요할 것 같나요?" → **Constraints + Phase 2 소스** 결정
3. "결과물이 어떤 형태여야 하나요?" → **Output Format** 확정

**Role**: 세 답변에서 자동 추론

**G1 기준**: purpose / constraints / success criteria 3개 모두 명확 + Role 추론 가능

---

### Phase 2. 컨텍스트 후보 수집

**목적**: Phase 1 답변 기반으로 소스 결정 + 후보 수집

**소스 자동 결정:**

| Phase 1 신호 | 수집 소스 |
|-------------|----------|
| 문서/URL 첨부 | 해당 파일/URL 전문 |
| 프로젝트/코드베이스 언급 | 코드베이스, 기존 spec, CLAUDE.md |
| "저장해줘" / 데이터 입력 | 입력 텍스트 전문 |
| 질문 / 태스크 요청 | KB index.md → 매칭 엔트리 |

**G2 기준**: Phase 1 purpose에 필요한 소스가 모두 수집됐는가? 명백한 누락 없음

---

### Phase 3. 컨텍스트 선택

**목적**: 수집된 후보에서 필요한 것만 남기기

**3개 기준으로 필터링:**
- **관련성**: Phase 1 purpose와 일치도
- **최신성**: 날짜 기준
- **신뢰성**: 소스 품질

**분류**: Keep / Skip / Merge

**G3 기준**: 3개 기준이 일관되게 적용됐는가? Skip 이유가 명확한가?

---

### Phase 4. 컨텍스트 구조화

**목적**: 모델이 읽기 좋은 형태로 정리

**구조:**
```
## Key Facts
## Constraints
## Decisions
## Notes
```

**G4 기준**: 모호한 항목 없음, 섹션 간 중복 없음

---

### Phase 5. 컨텍스트 압축

**목적**: 핵심 사실·제약·결정사항만 남기기

**규칙:**
- 중복 제거
- 접선 정보 → 한 줄 요약
- 고관련 내용 → 원문 유지

**G5 기준**: Constraints / Decisions 항목이 압축 후에도 살아있는가?

---

### Phase 6. 실행 지시 생성

**목적**: Phase 1 success criteria에 맞는 최종 출력물 생성

**출력 형식 (Phase 1 신호 기반 자동 결정):**

| Phase 1 success criteria 신호 | 출력 형식 |
|------------------------------|---------|
| "지시해줘" / "프롬프트 만들어줘" | **Role + Task + Constraints + Output Format** 지시문 |
| "저장해줘" / "기억해줘" | **KB 엔트리** (frontmatter + Key Facts/Constraints/Decisions/Notes) |
| "프로젝트 시작" / "개발하고 싶어" | **CLAUDE.md + spec + 구현 계획** |

**G6 기준**: 출력이 Phase 1 success criteria와 일치하는가? 누락·충돌·추측 없는가?
→ G6 통과: 완료 / G6 실패: Phase 7 자동 호출

---

### Phase 7. 최종 검증

**트리거**: G6 실패 시 자동 호출 OR `/context-engineering:verify` 수동 실행

**4가지 점검:**

| 항목 | 기준 |
|------|------|
| **누락** | Phase 1 purpose/constraints가 출력에 모두 반영됐는가 |
| **충돌** | 컨텍스트 내 모순이 없는가 |
| **추측** | 근거 없는 주장이 없는가 |
| **일관성** | 출력 형식이 Phase 1 success criteria와 일치하는가 |

**신뢰도 표시:**
- H (High): 3개 이상 근거 소스, 충돌 없음
- M (Medium): 1-2개 소스, 경미한 갭
- L (Low): 소스 부족 또는 충돌 존재

**피드백 루프**: 문제 유형에 따라 해당 Phase로 복귀

---

## 게이트 동작 방식

```
[자동 통과]  기준 명확히 충족 → 사용자 표시 없이 다음 Phase 진행

[사용자 확인] 모호하거나 기준 미충족 →
  "Phase N 결과: {요약}
   미충족 항목: {내용}
   수정하거나 승인 후 진행해주세요."
```

---

## 서브 스킬 진입점

```
/context-engineering           전체 파이프라인 (Phase 1 → 7)
/context-engineering:gather    Phase 1-3 (문제 정의 → 선택)
/context-engineering:build     Phase 4-5 (구조화 → 압축)
/context-engineering:compose   Phase 6   (실행 지시 생성)
/context-engineering:verify    Phase 7   (최종 검증, 재실행 가능)
```

중간 진입 조건: 이전 서브 스킬 산출물이 존재할 때만 허용
→ 산출물 없으면: "먼저 `/context-engineering:gather`를 실행해주세요" 안내 후 종료

---

## references/ 디렉토리 정리

현재 CE references:
- `phase-checklist.md` → 새 5-phase 체크리스트로 재작성
- `knowledge-base-template.md` → `build` Phase 4 템플릿으로 흡수
- `claude-md-policy-template.md` → `compose` Phase 6 출력 템플릿으로 유지
- `spec-template.md` → `compose` Phase 6 출력 템플릿으로 유지
- `architecture-principles.md` → 유지 (선택적 참조)

KE references:
- `entry-template.md` → `compose` Phase 6 KB 엔트리 템플릿으로 흡수
- `index-template.md` → `compose` Phase 6 인덱스 템플릿으로 흡수
