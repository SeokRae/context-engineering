---
name: build
description: >
  Phase 4-5: 선택된 컨텍스트를 구조화하고 압축한다.
  gather 산출물을 받아 모델이 읽기 좋은 형태로 정리하고, 핵심만 남긴다.
  Keywords: context structuring, compression, key facts, constraints, decisions
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Build (Phase 4-5)

컨텍스트를 구조화하고 압축하는 단계. gather가 선택한 컨텍스트를 받아 모델이 읽기 좋은 형태로 정리한 후 핵심만 남긴다.

---

## 전제조건

실행 전 gather 산출물 확인:

```
Phase 1 분석 결과 (purpose / constraints / success criteria / Role)
Phase 3 선택 결과 (Keep 항목 목록)
```

산출물이 없으면:
> "먼저 `/context-engineering:gather`를 실행해주세요."
종료.

---

## Phase 4. 컨텍스트 구조화

**목적**: 선택된 컨텍스트를 모델이 읽기 좋은 섹션 구조로 정리한다.

### 구조

```markdown
## Key Facts
{Phase 1 purpose와 직접 관련된 핵심 사실들}

## Constraints
{Phase 1 constraints에서 도출된 제약 조건들}

## Decisions
{관련 결정사항 및 그 근거}

## Notes
{맥락 이해에 도움이 되는 보조 정보}
```

### 구조화 규칙

- 각 항목은 하나의 사실/제약/결정을 표현
- 빈 섹션은 완전히 생략
- 항목 간 중복 없음 — 같은 내용이 두 섹션에 등장하면 더 적합한 섹션으로 통합
- Key Facts는 Phase 1 purpose와 직접 관련된 것만 포함

### G4 게이트

다음 기준을 AI가 자동 평가:

| 기준 | 평가 |
|------|------|
| 모호한 항목 없음 | 각 항목이 명확한 하나의 사실/제약/결정을 표현하는가 |
| 섹션 간 중복 없음 | 동일 내용이 두 섹션에 존재하지 않는가 |
| Purpose 반영 | Key Facts가 Phase 1 purpose와 연결되는가 |

**자동 통과**: 기준 명확히 충족 → Phase 5로 진행

**사용자 확인** (모호하거나 미충족 시):
```
Phase 4 결과:
  미충족 항목: {항목명} — {이유}
  수정하거나 승인 후 진행해주세요.
```

---

## Phase 5. 컨텍스트 압축

**목적**: 핵심 사실·제약·결정사항만 남기고 불필요한 부분을 제거한다.

### 압축 규칙

- **중복 제거**: 같은 의미의 항목이 여러 번 등장하면 하나로 합침
- **접선 정보 요약**: Phase 1 purpose와 간접적으로 관련된 내용 → 한 줄 요약으로 대체
- **핵심 유지**: Constraints / Decisions 항목은 원문 유지 — 이 항목들은 압축 대상이 아님
- **Key Facts 우선**: purpose와 직접 관련된 사실은 원문 유지

### G5 게이트

다음 기준을 AI가 자동 평가:

| 기준 | 평가 |
|------|------|
| Constraints 보존 | 압축 후에도 제약 조건이 모두 살아있는가 |
| Decisions 보존 | 결정사항과 근거가 손실 없이 유지되는가 |
| 과압축 방지 | Phase 1 purpose 달성에 필요한 정보가 제거되지 않았는가 |

**자동 통과**: 기준 명확히 충족 → compose로 전달

**사용자 확인** (모호하거나 미충족 시):
```
Phase 5 결과:
  미충족 항목: {항목명} — {이유}
  수정하거나 승인 후 진행해주세요.
```

---

## 완료 후

build 산출물 (구조화·압축된 컨텍스트 블록)을 가지고 다음 단계로:

```
/context-engineering:compose   — Phase 6 실행 지시 생성
```
