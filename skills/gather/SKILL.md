---
name: gather
description: >
  Phase 1-3: 문제를 정의하고, 컨텍스트 후보를 수집하고, 필요한 것만 선택한다.
  7-Phase Context Engineering 파이프라인의 첫 단계.
  Keywords: problem definition, context collection, selection, relevance, recency, reliability
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# Gather (Phase 1-3)

컨텍스트 엔지니어링의 출발점. 사용자 의도를 명확히 하고, 관련 소스에서 후보를 수집하고, 필요한 것만 선택한다.

---

## Phase 1. 문제 정의

**목적**: 사용자가 AI에게 원하는 것을 명확히 한다. 이후 모든 Phase의 기준이 된다.

### 질의 방식

**purpose → constraints → success criteria** 순서로, 하나씩, 가능하면 객관식으로 질문한다.

**질문 1** (purpose — Task 확정):
> "지금 해결하려는 문제가 무엇인가요?"

**질문 2** (constraints — Phase 2 소스 + Constraints 확정):
> "AI가 이 작업을 잘 하려면 어떤 배경 정보가 필요할 것 같나요? (예: 기존 코드, 문서, 이전 결정사항)"

**질문 3** (success criteria — Output Format 확정):
> "결과물이 어떤 형태여야 하나요? (예: 실행 지시문, 저장용 노트, 프로젝트 문서)"

**추가 질문**: 3개로 해소되지 않는 모호함이 남아있을 때만 1회 허용. 총 4개 이상 질문 금지.

### Scope Reduction

3개 질문 답변 수집 후 AI가 자동 수행하는 처리 단계 (추가 질문 아님):

**A. 문제 범위 축소**
- purpose 답변을 분석하여 "이 세션에서 풀 최소 문제"를 확인
- 이미 충분히 작으면 → 자동 통과
- 분해 가능하면 → 2-3개 선택지 제시, 사용자 선택

**B. 소스 범위 선언**
- constraints 답변에서 예상 소스를 분류:
  - **Must-have**: 직접 언급되었거나 명백히 필요한 소스 — Phase 2에서 우선 수집
  - **Nice-to-have**: 간접적으로 도움될 수 있는 소스 — Must-have 완료 후 여유 시 수집

**C. 세션 범위 선언 블록 출력**
```
--- 세션 범위 ---
최소 문제: {축소된 purpose}
Must-have 소스: {목록}
Nice-to-have 소스: {목록}
제외: {명시적으로 다루지 않을 것}
```

### Role 자동 추론

세 답변에서 AI가 담당해야 할 역할을 자동 추론한다. 사용자에게 역할을 직접 묻지 않는다.

### G1 게이트

다음 기준을 AI가 자동 평가:

| 기준 | 평가 |
|------|------|
| purpose 명확 | 해결하려는 문제가 구체적으로 정의됐는가 |
| constraints 명확 | 필요한 배경 정보의 범위가 파악됐는가 |
| success criteria 명확 | 결과물의 형태가 확정됐는가 |
| Role 추론 가능 | 세 답변에서 AI 역할을 추론할 수 있는가 |
| scope 축소 완료 | 세션 범위 선언 블록이 생성됐는가 |

**자동 통과**: 기준 명확히 충족 → Phase 2로 진행

**사용자 확인** (모호하거나 미충족 시):
```
Phase 1 결과:
  미충족 항목: {항목명} — {이유}
  수정하거나 승인 후 진행해주세요.
```

---

### 컴팩션 대비 Phase 1 결과 보존

G1 통과 후, 컨텍스트 컴팩션 시 유실 방지를 위해 `_phase1-result.md`에 기록한다:

    --- Phase 1 결과 ---
    purpose: {확정된 purpose}
    constraints: {확정된 constraints}
    success criteria: {확정된 success criteria}
    Role: {추론된 역할}
    scope-declaration: {세션 범위 선언 블록}

Phase 2 시작 시 대화 컨텍스트에 Phase 1 결과가 없으면 `_phase1-result.md`에서 복원한다.
파이프라인 완료(G6 통과 또는 G7 통과) 후 삭제한다.

---

## Phase 2. 컨텍스트 후보 수집

**목적**: Phase 1 답변을 바탕으로 소스를 결정하고 후보 컨텍스트를 수집한다.

### 소스 자동 결정

Phase 1 세션 범위 선언의 Must-have 소스를 우선 수집하고, 예산 여유가 있을 때 Nice-to-have 소스를 추가한다.

Phase 1의 constraints 답변 신호에서 소스를 결정한다:

| Phase 1 신호 | 수집 소스 |
|-------------|----------|
| 문서 / 파일 / URL 제공 | 해당 파일 전문 읽기 / URL 페치 |
| 프로젝트 / 코드베이스 언급 | 코드베이스 탐색, 기존 spec, CLAUDE.md |
| "저장해줘" / 데이터 입력 | 입력 텍스트 전문 |
| 질문 / 태스크 요청 | KB `index.md` 스캔 → 매칭 엔트리 읽기 |
| 복합 (여러 신호 혼재) | 위 소스 조합 — 서브 에이전트 병렬 실행 |

> 전략 분류 (RAG / Memory / Tool Result) → [Context Source Strategy](../context-engineering/references/context-source-strategy.md)

### KB 검색 강화

**KB 존재 여부 확인**:
1. `index.md` 파일 경로를 탐색한다 (KB 설정이 있으면 해당 경로, 없으면 일반적 위치 스캔)
2. `index.md`가 존재하지 않으면:
   - 아래 KB 검색 4단계를 **전체 건너뜀**
   - "KB가 설정되지 않았습니다. KB 없이 진행합니다." 안내 후 Phase 1 신호 테이블의 다른 소스로만 수집 진행
   - 참조: [Failure Cases — 시나리오 7](../context-engineering/references/failure-cases.md)
3. `index.md`가 존재하면 → 아래 KB 검색 4단계를 실행:

KB `index.md`에서 후보를 찾을 때 다음 순서로 검색한다:

1. **키워드 매칭**: Phase 1 purpose의 핵심 단어로 index.md 테이블 스캔
2. **태그 확장**: 매칭된 엔트리의 `tags` 필드를 읽고, 같은 태그를 가진 다른 엔트리도 후보에 추가
3. **관련 엔트리 추적**: 매칭된 엔트리의 `related` 필드를 따라가며 연결된 엔트리를 후보에 추가
4. **신뢰도 우선**: 동일 주제에 여러 엔트리가 있으면 `reliability: high` 우선

검색 결과가 5개 이상이면 Phase 3에서 필터링한다.

### 동적 컨텍스트 주입

프로젝트/코드베이스 소스의 경우, 수집 시점의 최신 상태를 동적으로 확인한다:

- 최근 변경: `git log --oneline -5`
- 프로젝트 구조: `find . -name "*.md" -maxdepth 2 -not -path "./.git/*"`
- 빌드 파일 상태: `head -20 package.json` 또는 `head -20 build.gradle` (해당 파일이 있으면)

결과를 후보 컨텍스트에 포함하여 Phase 3 Keep/Skip 판단의 근거로 사용한다.

### 수집 시 주의

- 소스가 10,000단어 이상이면 섹션 분할 여부 확인
- KB에서 후보가 0건이면 즉시 알림: "저장된 지식에서 관련 항목을 찾지 못했습니다. 일반 지식으로 진행합니다."
- 소스가 대형일 때 점진적 로딩: 목차/헤더 먼저 스캔 → purpose 관련 섹션만 선택적 수집
- 수집한 각 소스의 크기(단어 수)를 기록 — build Phase 5 예산 추정에 사용

### G2 게이트

다음 기준을 AI가 자동 평가:

| 기준 | 평가 |
|------|------|
| 소스 완전성 | Phase 1 purpose 달성에 필요한 소스가 모두 수집됐는가 |
| 명백한 누락 없음 | constraints 답변에서 언급된 자료가 모두 포함됐는가 |

**자동 통과**: 기준 명확히 충족 → Phase 3으로 진행

**사용자 확인** (누락이 의심될 때):
```
Phase 2 결과:
  수집된 소스: {소스 목록}
  의심 누락: {항목} — 이 소스도 필요한가요?
```

---

## Phase 3. 컨텍스트 선택

**목적**: 수집된 후보에서 Phase 1 purpose에 실제로 필요한 것만 남긴다.

### 3개 기준으로 평가

| 기준 | 설명 |
|------|------|
| **관련성** | Phase 1 purpose와 직접 연결되는가 |
| **최신성** | 더 최근의 정보가 있다면 오래된 것을 대체 |
| **신뢰성** | 소스의 품질 (직접 진술 > 공식 문서 > 간접 참조) |

### 분류

각 후보 항목을 분류한다:

- **Keep**: 관련성·최신성·신뢰성 기준 충족 → 다음 단계로 전달
- **Skip**: 기준 미충족 — 이유 명시
- **Merge**: 기존 Keep 항목과 70% 이상 의미 중복 → 더 신뢰도 높은 항목으로 통합

### G3 게이트

다음 기준을 AI가 자동 평가:

| 기준 | 평가 |
|------|------|
| 기준 일관 적용 | 관련성·최신성·신뢰성이 모든 항목에 동일하게 적용됐는가 |
| Skip 이유 명확 | Skip된 항목마다 이유가 있는가 |
| Keep 항목 충분 | Phase 1 purpose 달성에 필요한 정보가 Keep에 포함됐는가 |

**자동 통과**: 기준 명확히 충족 → build로 전달

**사용자 확인** (모호하거나 미충족 시):
```
Phase 3 결과:
  Keep: {N}개  Skip: {N}개  Merge: {N}개
  미충족 항목: {항목명} — {이유}
  수정하거나 승인 후 진행해주세요.
```

---

## 완료 후

gather 산출물 (Phase 1 분석 + 선택된 컨텍스트 세트)을 가지고 다음 단계로:

```
/context-engineering:build   — Phase 4-5 구조화·압축
```
