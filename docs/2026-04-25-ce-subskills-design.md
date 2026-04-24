# ce sub-skills 설계

**작성일**: 2026-04-25  
**목적**: context-engineering 워크플로우의 특정 Phase를 직접 호출할 수 있는 `ce` 플러그인 설계

---

## 배경

기존 `/context-engineering` 스킬은 Step 0부터 Phase 4까지 순차 진행을 전제로 설계되었다.  
실제 작업에서는 이미 진행 중인 프로젝트에서 특정 단계만 재작업하거나, 피드백 루프로 특정 Phase로 돌아가는 경우가 많다.  
이를 위해 Phase 단위로 직접 진입할 수 있는 `ce` 플러그인을 별도로 만든다.

---

## 명령어

| 명령 | 담당 Phase | 전제 artifacts |
|------|-----------|---------------|
| `/ce:gather` | Step 0 + Phase 1 (KB) + Phase 2 (CLAUDE.md) | 없어도 시작 가능 |
| `/ce:spec`   | Phase 3 — PRD + SPEC | KB 있으면 로드 |
| `/ce:impl`   | Phase 4 — 구현 | SPEC 있으면 로드 |
| `/ce:valid`  | Readiness Gate 체크 | PRD·SPEC·KB·CLAUDE.md 확인 |

---

## 플러그인 구조

```
ce/
├── .claude-plugin/
│   └── plugin.json          (name: "ce")
├── skills/
│   ├── gather/SKILL.md
│   ├── spec/SKILL.md
│   ├── impl/SKILL.md
│   └── valid/SKILL.md
└── README.md
```

기존 `context-engineering` 레포와 별도 레포로 관리한다.  
설치 후 `/ce:gather` 형태로 호출된다.

---

## Context 해석 규칙

모든 sub-skill에 공통 적용:

```
1. 인자 있음:  /ce:spec @/path/to/project  →  해당 경로를 CODE_PATH로 사용
2. 인자 없음:  현재 디렉토리부터 docs/ 탐색 (knowledge-base.md, prd.md, spec.md 존재 여부)
3. 탐색 실패:  "프로젝트 경로를 알려주세요" 질문
```

---

## 각 스킬 상세 동작

### `/ce:gather`

**진입 시:**
- 기존 `knowledge-base.md`, `CLAUDE.md` 탐지
- 있으면: 현재 상태 요약 출력 → "재작업(처음부터) / 갱신(추가·수정)" 선택
- 없으면: Step 0 질문 3개부터 시작

**진행:**
```
Step 0 → Phase 1 (KB 조립) → Phase 2 (CLAUDE.md 작성)
```

**종료:** "gather 완료 — /ce:spec 으로 PRD·SPEC을 작성하거나 /ce:valid 로 체크할 수 있습니다."

---

### `/ce:spec`

**진입 시:**
- 기존 `knowledge-base.md` 로드 (있으면 도메인 컨텍스트 자동 반영)
- 기존 `prd.md`, `spec.md` 탐지
- 있으면: 현재 상태 요약 → "재작업 / 갱신" 선택
- 없으면: Phase 3 처음부터

**진행:**
```
Phase 3-1 (PRD) → Phase 3-2 (SPEC) → /ce:valid 제안
```

---

### `/ce:impl`

**진입 시:**
- `spec.md` 로드 (있으면 구현 계획 테이블 확인)
- 없으면: "SPEC 없이 진행하면 탐색 모드로 시작됩니다" 안내

**진행:**
```
Phase 4 구현 단위 반복 (빌드 → 테스트 → PRD 기준 확인 → 피드백 루프)
```

---

### `/ce:valid`

**진입 시:**
- PRD·SPEC·KB·CLAUDE.md 현재 상태 탐지

**출력:**
```
[Readiness Gate]
① PRD 성공 기준 → {기능 N개 중 M개 기재}
② SPEC 아키텍처 결정 근거 → {결정 N개 중 M개}
③ 도메인 용어 미확정 → {N개}
④ CLAUDE.md 제약 반영 → {Phase 1 제약 M개 반영}
⑤ SPEC 첫 항목 착수 가능 → {상태}

미충족 항목 지정 또는 "Phase 4 진행" 확인 요청
```

---

## 기존 스킬과의 관계

- `/context-engineering` 전체 워크플로우는 그대로 유지
- `ce` sub-skills는 독립 SKILL.md로 작성 (context-engineering SKILL.md에 의존하지 않음)
- 공통 references(KB 템플릿 등)는 각 sub-skill SKILL.md에 필요한 부분만 인라인으로 포함

---

## 설치

```bash
cd /Users/sr/IdeaProjects/claude/ce
# 플러그인 등록 (omx 또는 claude plugin 방식)
```
