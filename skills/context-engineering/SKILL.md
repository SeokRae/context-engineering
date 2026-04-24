---
name: context-engineering
description: >
  AI 도구(Claude Code)로 복잡한 서비스를 개발할 때 따르는 4단계 Context Engineering 워크플로우.
  도메인 지식 수집 → Policy(CLAUDE.md) 작성 → 구현 Spec 설계 → 단계별 구현 검증.
  새 프로젝트 시작, 대형 기능 개발, 외부 API 연동 서비스 구축 시 사용.
  Keywords: context engineering, AI 개발, 도메인 지식, CLAUDE.md, 서비스 개발, 컨텍스트 엔지니어링,
  knowledge base, policy, spec, implementation, 구현 명세, 새 프로젝트, service dev
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# context-engineering

AI 도구로 복잡한 서비스를 개발할 때 **"무엇을 알고, 어떻게 행동하고, 무엇을 만들어야 하는가"** 를 단계별로 정의하는 4단계 워크플로우.

```
Step 0: Context Gathering  ←  요구사항 탐색 + 파라미터 수집
  ↓  [HARD-GATE: 요구사항 확인 완료 시만 진입]
┌──────────────────────────────────────────┐
│  Phase 1: Knowledge Base                 │
│    ↓                                     │
│  Phase 2: Policy (CLAUDE.md)             │
│    ↓                                     │
│  Phase 3: Spec  →  [Readiness Gate]      │
│    ↑_________________↓ 미충족 시 재순환  │
└──────────────────────────────────────────┘
  ↓  [HARD-GATE: 계획 준비 완료]
Phase 4: Implementation
```

## 사용법

```
/context-engineering @<코드경로> [--output <출력경로>] [--specs <스펙문서경로>]
```

**예시:**
```
/context-engineering @/path/to/project
/context-engineering @/path/to/project --specs docs/references/
/context-engineering @/path/to/project --output /tmp/context-docs --specs docs/references/
```

---

## Step 0: Context Gathering

스킬 시작 시 **두 가지를 순서대로** 수행한다: ① 요구사항 탐색 대화, ② 기술 파라미터 수집.

### Step 0-1. 요구사항 탐색 (HARD-GATE)

**질문은 한 번에 하나씩** 던진다. 사용자 답변을 받은 후 다음 질문으로 넘어간다.

최소 3가지를 확인한다:

1. **무엇을 만드는가?**
   > "이번에 만들려는 것이 무엇인지 한 문장으로 설명해 주세요."

2. **왜 만드는가? (목적·동기)**
   > "이것을 왜 만들어야 하나요? 해결하려는 문제나 달성하려는 목표가 무엇인가요?"

3. **알려진 제약이 있는가?**
   > "지금 알고 있는 제약 조건이 있나요? (기술 스택 고정, 마감일, 연동 시스템, 성능 요구사항 등)"

3가지 답변이 모이면 요약을 출력하고 사용자에게 확인 요청:

```
[요구사항 요약]
- 무엇: {답변 요약}
- 왜: {답변 요약}
- 제약: {답변 요약 또는 "없음"}

이 내용이 맞나요? 확인되면 기술 파라미터를 수집하고 Phase 1을 시작합니다.
```

> **HARD-GATE**: 사용자가 요약을 확인하기 전까지 Phase 1로 진입하지 않는다.

### Step 0-2. 기술 파라미터 수집

요구사항 확인 후, `@<코드경로>`가 주어지면 빌드 파일에서 자동 감지한다.

| 파라미터 | 설명 | 자동 감지 |
|---------|------|---------|
| `{PROJECT_NAME}` | 프로젝트 이름 | — (사용자 입력 또는 디렉토리명) |
| `{CODE_PATH}` | 코드 저장소 절대 경로 | `@` 인자 |
| `{TECH_STACK}` | 기술 스택 | `build.gradle` / `pom.xml` / `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` |
| `{BUILD_CMD}` | 빌드 명령어 | 빌드 파일 기반 추론 |
| `{TEST_CMD}` | 테스트 명령어 | 빌드 파일 기반 추론 |
| `{SPEC_PATHS}` | 외부 스펙 문서 경로 | `--specs` 인자 |
| `{OUTPUT_PATH}` | 산출물 저장 경로 | `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/` |

파라미터 수집 후 최종 요약 출력하고 사용자 확인 요청.

---

## Phase 1: Knowledge Base (도메인 지식 준비)

**목적**: AI가 이 프로젝트의 도메인을 이해하기 위한 지식 베이스를 구축한다.

### Step 1-1. Context Assessment (시나리오 판별)

프로젝트 상황을 분석해 아래 4가지 시나리오 중 하나를 판별한다:

| 시나리오 | 조건 | 설명 |
|---------|------|------|
| **A. 그린필드** | 스펙 문서 없음 + 코드 없음 | 아무것도 없는 새 프로젝트 |
| **B. 스펙 우선** | 스펙 문서 있음 + 코드 없음 | 문서는 있지만 구현 전 |
| **C. 코드 우선** | 스펙 문서 없음 + 코드 있음 | 기존 코드가 있지만 문서 없음 |
| **D. 풀 컨텍스트** | 스펙 문서 있음 + 코드 있음 | 문서와 코드 모두 있음 |

판별 기준:
- `{SPEC_PATHS}` 유무 → 스펙 문서 존재 여부
- `{CODE_PATH}` 존재 시 소스 파일 유무 → 코드 존재 여부

판별 결과를 한 줄로 출력하고 바로 Source Scan으로 진행한다:

```
[Context Assessment] 시나리오 {A/B/C/D} ({시나리오명}) — {판별 근거 한 줄}
```

### Step 1-2. Source Scan (조건부 실행)

시나리오에 따라 다른 방식으로 지식을 수집한다:

**시나리오 A (그린필드)**:
- Step 0 대화 내용에서 도메인 개념, 제약, 목적 추출
- 외부 참조 없이 대화 기반으로 초안 작성
- 산출물에 "초안 (검증 필요)" 표시

**시나리오 B (스펙 우선)**:
- `{SPEC_PATHS}` 문서를 Explore 서브에이전트로 병렬 탐색
- 프로토콜 스펙, API 가이드, 벤더 문서 수집
- 각 문서를 한 줄 요약으로 압축

**시나리오 C (코드 우선)**:
- `{CODE_PATH}` 코드베이스 스캔
- 패키지 구조, 주요 클래스/모듈 패턴
- 이미 적용된 설계 패턴, 기존 테스트 전략

**시나리오 D (풀 컨텍스트)**:
- 시나리오 B + C 병렬 실행 (스펙 탐색과 코드 스캔을 동시에)

### Step 1-3. Knowledge Base 조립

시나리오와 무관하게 항상 실행. 수집된 정보를 아래 구조로 정리한다.

**도메인 용어 (Glossary)**:

```markdown
| 용어 | 정의 | 출처 |
|------|------|------|
| {용어} | {한 줄 정의} | {문서명 또는 "대화"} |
```

**제약 목록**:

```markdown
| 제약 | 영향 | 우선순위 |
|------|------|---------|
| {제약 설명} | {영향} | High/Medium/Low |
```

**참조 문서 매니페스트** (시나리오 A는 생략):

```markdown
| 경로 | 유형 | 요약 | 갱신일 |
|------|------|------|--------|
| {경로} | spec/code/api | {한 줄} | {날짜} |
```

**산출물**: `{OUTPUT_PATH}/knowledge-base.md`
- 시나리오 A: 파일 상단에 `> ⚠️ 초안 — Step 0 대화 기반, 구현 진행 중 검증 필요` 표시

> **게이트**: "Phase 1 완료 — `knowledge-base.md` 초안을 저장했습니다. 검토 후 Phase 2 진행할까요?"

---

## Phase 2: Policy (CLAUDE.md 작성)

**목적**: AI가 이 프로젝트에서 어떻게 행동해야 하는지 정책을 정의한다.

### Step 2-1. CLAUDE.md 레벨 결정

| 레벨 | 위치 | 줄 수 제한 | 용도 |
|------|------|----------|------|
| 프로젝트 | `{CODE_PATH}/CLAUDE.md` | 100~150줄 | 전체 프로젝트 공통 |
| 모듈 | `{CODE_PATH}/{모듈}/CLAUDE.md` | 60~100줄 | 모듈별 특화 |

### Step 2-2. CLAUDE.md 작성

WHAT/WHY/HOW 3섹션 구조:

```markdown
# {PROJECT_NAME}

> {한 줄 설명}

## 아키텍처

{핵심 패턴 3~5가지 — WHY 포함}

## 빌드 & 테스트

\`\`\`bash
{BUILD_CMD}
{TEST_CMD}
\`\`\`

## 포트 / 엔드포인트

| 포트 | 용도 |
|------|------|

## 핵심 제약

{Phase 1에서 도출한 TC/BL 제약 중 개발에 직접 영향을 주는 것}

## Non-obvious Gotchas

- {함정 1}
- {함정 2}
```

150줄 초과 항목은 별도 문서로 분리 후 링크.

**산출물**: `{CODE_PATH}/CLAUDE.md` (+ 모듈별 CLAUDE.md)

> **게이트**: "Phase 2 완료 — `CLAUDE.md` 초안을 저장했습니다. 검토 후 Phase 3 진행할까요?"

---

## Phase 3: Spec (구현 명세)

**목적**: 무엇을 어떤 순서로 만들지 명세한다.

### Step 3-1. 기능 요구사항 (FR-*)

```markdown
| ID | 요구사항 | 수용 기준 | 우선순위 |
|----|---------|---------|---------|
| FR-1 | {기능} | {검증 가능한 기준} | High |
```

### Step 3-2. 설계 결정 (D-*)

```markdown
| ID | 결정 사항 | 대안 | 선택 | 근거 |
|----|---------|------|------|------|
| D-1 | {무엇을 결정했나} | {고려한 대안들} | {선택한 것} | {이유} |
```

### Step 3-3. 패키지/모듈 구조

Ports & Adapters + DDD 기반으로 레이어를 설계한다.
의존 방향: `진입점 → 애플리케이션 → 도메인 ← 인프라` (역방향 금지)

원칙·레이어 정의·언어별 폴더명 → [references/architecture-principles.md](references/architecture-principles.md)

### Step 3-4. WBS

```markdown
| Phase | 작업 | 의존성 | 예상 산출물 |
|-------|------|--------|-----------|
| P1 | 공통 모듈 | — | {모듈명} |
| P2 | 핵심 서버 | P1 | {모듈명} |
```

### Step 3-5. 구현 순서

의존성 그래프 기반으로 모듈 구현 순서 결정. 공유 공통 모듈 → 핵심 서버 → 클라이언트 → 통합 테스트.

**산출물**: `{OUTPUT_PATH}/spec/`

### Step 3-6. Readiness Gate (반복 여부 판단)

아래 기준을 모두 충족해야 Phase 4로 진행한다.

| 기준 | 미충족 시 돌아갈 Phase |
|------|----------------------|
| 모든 FR-*에 검증 가능한 수용 기준이 있는가? | Phase 3 |
| 설계 결정(D-*)에 대안과 근거가 있는가? | Phase 3 |
| 도메인 용어 미확정 항목이 없는가? | Phase 1 |
| CLAUDE.md가 실제 제약을 반영하는가? | Phase 2 |
| WBS 첫 번째 작업을 지금 바로 시작할 수 있는가? | Phase 3 |

하나라도 미충족이면 해당 Phase로 돌아가 보강 후 Phase 3까지 재순환한다.

> **HARD-GATE**: 모두 충족 시 "계획 준비 완료 — Phase 4로 진행합니다." 출력 후 Phase 4 진입.

---

## Phase 4: Implementation (단계별 구현)

**목적**: Phase 3 WBS 순서대로 구현하고 각 단계에서 검증한다.

### 각 WBS 단계마다

**진입 기준**: Phase 3 해당 WBS 항목의 FR-*, D-* 확인 완료

**완료 기준**: 아래 체크리스트 전부 통과 시만 다음 단계로 이동

- [ ] `{BUILD_CMD}` 성공
- [ ] `{TEST_CMD}` 통과 (신규 + 기존 회귀 없음)
- [ ] Phase 3 설계 결정(D-*) 준수
- [ ] 발견 사항 해당 단계 문서에 반영 완료

### 피드백 루프 — 구현 중 발견 시 해당 단계로 돌아간다

구현 중 현실과 맞지 않는 내용이 발견되면, Spec만 고치는 것이 아니라 **발견된 내용의 성격에 따라 해당 단계로 돌아가** 갱신한다.

| 발견 내용 | 갱신 대상 |
|---------|---------|
| 도메인 개념 오류, 용어 불일치, 새 제약 발견 | Phase 1 `knowledge-base.md` |
| AI 행동 규칙 변경 필요, 새 gotcha 발견 | Phase 2 `CLAUDE.md` |
| 기능 요구사항 변경, 설계 결정 번복, WBS 재조정 | Phase 3 `spec/` |

갱신 절차:
1. 발견 내용과 이유를 한 줄로 기록
2. 해당 단계 문서의 항목 갱신
3. 갱신 후 구현 계속

> **원칙**: 구현을 문서에 맞추지 말고, 문서를 현실에 맞춰 갱신한다. 발견은 전 단계 어디든 반영될 수 있다.


---

## 참조 문서

- [4단계 체크리스트](references/phase-checklist.md)
- [Knowledge Base 템플릿](references/knowledge-base-template.md)
- [CLAUDE.md 정책 템플릿](references/claude-md-policy-template.md)
- [구현 Spec 템플릿](references/spec-template.md)
- [아키텍처 원칙](references/architecture-principles.md)
