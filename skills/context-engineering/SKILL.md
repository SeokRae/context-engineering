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
Step 0: Context Gathering
  ↓
Phase 1: Knowledge Base  →  확인
  ↓
Phase 2: Policy (CLAUDE.md)  →  확인
  ↓
Phase 3: Spec (구현 명세)  →  확인
  ↓
Phase 4: Implementation
```

## 사용법

```
/context-engineering @<코드경로> [--obsidian <옵시디언경로>] [--specs <스펙문서경로>]
```

**예시:**
```
/context-engineering @/Users/sr/IdeaProjects/payment/fx --obsidian 20-areas/payment/financial-information-exchange --specs docs/references/
```

---

## Step 0: Context Gathering

스킬 시작 시 아래 파라미터를 수집한다. `@<코드경로>`가 주어지면 빌드 파일에서 자동 감지.

| 파라미터 | 설명 | 자동 감지 |
|---------|------|---------|
| `{PROJECT_NAME}` | 프로젝트 이름 | — (사용자 입력) |
| `{CODE_PATH}` | 코드 저장소 절대 경로 | `@` 인자 |
| `{OBSIDIAN_AREA}` | Obsidian 노트 영역 (선택) | `--obsidian` 인자 (vault 루트 기준 상대경로) |
| `{TECH_STACK}` | 기술 스택 | `build.gradle` / `pom.xml` / `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` |
| `{BUILD_CMD}` | 빌드 명령어 | 빌드 파일 기반 추론 |
| `{TEST_CMD}` | 테스트 명령어 | 빌드 파일 기반 추론 |
| `{SPEC_PATHS}` | 외부 스펙 문서 경로 | `--specs` 인자 |
| `{OUTPUT_PATH}` | 산출물 저장 경로 | `--obsidian` 지정 시 vault `{OBSIDIAN_AREA}/docs/`, 없으면 `{CODE_PATH}/docs/` |

파라미터 수집 후 요약 출력하고 사용자 확인 요청.

---

## Phase 1: Knowledge Base (도메인 지식 준비)

**목적**: AI가 이 프로젝트의 도메인을 이해하기 위한 지식 베이스를 구축한다.

### Step 1-1. 외부 스펙/API 문서 수집

`{SPEC_PATHS}`의 문서를 Explore 서브에이전트로 병렬 탐색:
- 프로토콜 스펙, API 가이드, 벤더 문서
- 각 문서를 한 줄 요약으로 압축

### Step 1-2. 도메인 용어 정의

핵심 개념을 Domain Glossary 테이블로 정리:

```markdown
| 용어 | 정의 | 출처 |
|------|------|------|
| {용어} | {한 줄 정의} | {문서명} |
```

### Step 1-3. 제약조건 레지스트리

기술·비즈니스·조직 제약을 분류해 등록:

```markdown
| ID | 분류 | 설명 | 영향 | 우선순위 |
|----|------|------|------|---------|
| TC-1 | 기술 | {제약} | {영향} | High |
| BL-1 | 비즈니스 | {규칙} | {영향} | Medium |
| OC-1 | 조직 | {제약} | {영향} | Low |
```

### Step 1-4. 기존 코드 스캔 (있을 경우)

`{CODE_PATH}`가 있으면:
- 패키지 구조, 주요 클래스 패턴
- 이미 적용된 설계 패턴
- 기존 테스트 전략

### Step 1-5. 참조 문서 매니페스트

```markdown
| 경로 | 유형 | 요약 | 갱신일 |
|------|------|------|--------|
| {경로} | spec/meeting/api | {한 줄} | {날짜} |
```

**산출물**: `{출력경로}/knowledge-base.md`

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

### Step 3-3. 패키지 구조

```
{루트 패키지}
├── config/       ← Bean 조립, 설정 바인딩
├── adapter/      ← 외부 경계 (in/out)
├── application/  ← 비즈니스 로직
└── infrastructure/ ← 부가 기능
```

### Step 3-4. WBS

```markdown
| Phase | 작업 | 의존성 | 예상 산출물 |
|-------|------|--------|-----------|
| P1 | 공통 모듈 | — | {모듈명} |
| P2 | 핵심 서버 | P1 | {모듈명} |
```

### Step 3-5. 구현 순서

의존성 그래프 기반으로 모듈 구현 순서 결정. 공유 공통 모듈 → 핵심 서버 → 클라이언트 → 통합 테스트.

**산출물**: `{출력경로}/spec/` 또는 Obsidian `docs/`

> **게이트**: "Phase 3 완료 — `spec/` 초안을 저장했습니다. 검토 후 Phase 4 진행할까요?"

---

## Phase 4: Implementation (단계별 구현)

**목적**: Phase 3 WBS 순서대로 구현하고 각 단계에서 검증한다.

### 각 WBS 단계마다

**진입 기준**: Phase 3 해당 WBS 항목의 FR-*, D-* 확인 완료

1. 스켈레톤 생성 (패키지 구조, 빈 클래스)
2. 핵심 도메인 객체 구현
3. 어댑터 구현
4. 테스트 작성 + 통과 확인
5. CLAUDE.md 갱신 (구현 중 발견한 gotchas 추가)

**완료 기준**: 아래 체크리스트 전부 통과 시만 다음 단계로 이동

```bash
# 컴파일
{BUILD_CMD}

# 테스트
{TEST_CMD}
```

- [ ] 빌드 성공 (컴파일 에러 없음)
- [ ] 테스트 통과 (신규 + 기존 회귀 없음)
- [ ] Phase 3 설계 결정(D-*) 준수
- [ ] CLAUDE.md 최신 상태 반영

### Spec 변경이 필요한 경우 (피드백 루프)

구현 중 Phase 3 Spec이 현실과 맞지 않는 경우:

1. 변경 이유를 간단히 기록
2. Phase 3 `spec.md`의 해당 FR-* 또는 D-* 항목 갱신
3. 갱신 후 계속 구현

> **원칙**: 구현을 Spec에 맞추지 말고, Spec을 현실에 맞춰 갱신한다.

### 사후 문서화

구현 완료 후:
- 아키텍처 문서 (`docs/{번호}-{서비스}-architecture.md`)
- 개발자 온보딩 가이드
- Obsidian 다이어그램 (해당 시)

---

## 참조 문서

- [4단계 체크리스트](references/phase-checklist.md)
- [Knowledge Base 템플릿](references/knowledge-base-template.md)
- [CLAUDE.md 정책 템플릿](references/claude-md-policy-template.md)
- [구현 Spec 템플릿](references/spec-template.md)
