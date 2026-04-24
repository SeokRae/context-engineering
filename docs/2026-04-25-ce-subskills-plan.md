# ce sub-skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `ce` 플러그인 생성 — `/ce:gather`, `/ce:spec`, `/ce:impl`, `/ce:valid` 4개 sub-skill로 context-engineering 워크플로우의 특정 Phase에 직접 진입

**Architecture:** 기존 `/context-engineering` 스킬과 완전히 독립된 새 플러그인. 각 SKILL.md는 자체 완결적으로 작성하며 공통 로직(Context Resolution)을 인라인으로 포함. 템플릿은 별도 references/ 없이 SKILL.md에 직접 기술.

**Tech Stack:** Markdown only (Claude Code plugin — no build tools, no tests in the traditional sense)

---

## Context Resolution 공통 블록

모든 SKILL.md에 동일하게 포함:

```markdown
## Context Resolution

경로를 다음 순서로 결정한다:

1. `@<code-path>` 인자 있음 → `{CODE_PATH}` = 해당 경로
2. 없음 → 현재 디렉토리에서 `docs/knowledge-base.md` 탐색
3. 탐색 실패 → "프로젝트 경로를 알려주세요 (예: `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/`
```

---

### Task 1: 레포 초기화 + plugin.json

**Files:**
- Create: `/Users/sr/IdeaProjects/claude/ce/.claude-plugin/plugin.json`
- Create: `/Users/sr/IdeaProjects/claude/ce/.gitignore`

- [ ] **Step 1: 디렉토리 생성**

```bash
mkdir -p /Users/sr/IdeaProjects/claude/ce/.claude-plugin
mkdir -p /Users/sr/IdeaProjects/claude/ce/skills/gather
mkdir -p /Users/sr/IdeaProjects/claude/ce/skills/spec
mkdir -p /Users/sr/IdeaProjects/claude/ce/skills/impl
mkdir -p /Users/sr/IdeaProjects/claude/ce/skills/valid
```

- [ ] **Step 2: plugin.json 작성**

`/Users/sr/IdeaProjects/claude/ce/.claude-plugin/plugin.json`:
```json
{
  "name": "ce",
  "version": "1.0.0",
  "description": "context-engineering sub-skills — Phase별 직접 진입: /ce:gather, /ce:spec, /ce:impl, /ce:valid",
  "author": {
    "name": "SeokRae",
    "email": "kslbsh@gmail.com"
  },
  "homepage": "https://github.com/SeokRae/ce",
  "repository": "https://github.com/SeokRae/ce",
  "license": "MIT",
  "keywords": [
    "context-engineering",
    "ce",
    "gather",
    "spec",
    "impl",
    "valid"
  ]
}
```

- [ ] **Step 3: git 초기화 + .gitignore**

```bash
cd /Users/sr/IdeaProjects/claude/ce
git init
echo ".DS_Store" > .gitignore
```

- [ ] **Step 4: GitHub 레포 생성 + 초기 커밋**

```bash
gh repo create SeokRae/ce --public --description "context-engineering Phase별 직접 진입 sub-skills"
git add .
git commit -m "chore: init ce plugin"
git branch -M main
git remote add origin https://github.com/SeokRae/ce.git
git push -u origin main
```

---

### Task 2: /ce:gather SKILL.md

**Files:**
- Create: `/Users/sr/IdeaProjects/claude/ce/skills/gather/SKILL.md`

- [ ] **Step 1: Issue 생성**

```bash
cd /Users/sr/IdeaProjects/claude/ce
gh issue create \
  --title "feat: /ce:gather — Step 0 + Phase 1 (KB) + Phase 2 (CLAUDE.md)" \
  --body "Step 0 요구사항 탐색부터 knowledge-base.md, CLAUDE.md 작성까지 진행하는 sub-skill"
```

- [ ] **Step 2: 브랜치 생성**

```bash
git checkout -b feature/{issue-번호}-gather
```

- [ ] **Step 3: SKILL.md 작성**

`/Users/sr/IdeaProjects/claude/ce/skills/gather/SKILL.md`:
```markdown
---
name: gather
description: >
  context-engineering Step 0 + Phase 1 (Knowledge Base) + Phase 2 (CLAUDE.md).
  새 프로젝트 시작 또는 기존 KB·CLAUDE.md 재작업·갱신 시 사용.
  Keywords: context gathering, knowledge base, CLAUDE.md, 요구사항 탐색, 도메인 지식, gather
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# ce:gather

프로젝트 컨텍스트 수집(Step 0) → Knowledge Base(Phase 1) → CLAUDE.md(Phase 2)를 진행한다.

## 사용법

```
/ce:gather [@<code-path>] [--output <output-path>]
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

> **완료**: "gather 완료 — `/ce:spec`으로 PRD·SPEC을 작성하거나 `/ce:valid`로 Readiness Gate를 체크할 수 있습니다."
```

- [ ] **Step 4: 커밋 + PR**

```bash
git add skills/gather/SKILL.md
git commit -m "feat: /ce:gather — Step 0 + Phase 1 + Phase 2 (#이슈번호)"
git push origin feature/{이슈번호}-gather
gh pr create --title "feat: /ce:gather" --body "Closes #{이슈번호}"
gh pr merge {PR번호} --merge --delete-branch
git checkout main && git pull origin main
```

---

### Task 3: /ce:spec SKILL.md

**Files:**
- Create: `/Users/sr/IdeaProjects/claude/ce/skills/spec/SKILL.md`

- [ ] **Step 1: Issue 생성 + 브랜치**

```bash
gh issue create --title "feat: /ce:spec — Phase 3 PRD + SPEC" \
  --body "PRD와 SPEC을 작성하는 sub-skill. knowledge-base.md 있으면 로드해서 컨텍스트 활용."
git checkout -b feature/{이슈번호}-spec
```

- [ ] **Step 2: SKILL.md 작성**

`/Users/sr/IdeaProjects/claude/ce/skills/spec/SKILL.md`:
```markdown
---
name: spec
description: >
  context-engineering Phase 3 — PRD(제품 요구사항) + SPEC(기술 명세) 작성.
  새로 작성하거나 기존 PRD·SPEC 재작업·갱신 시 사용.
  Keywords: PRD, SPEC, 요구사항 명세, 기술 명세, 구현 계획, spec
allowed-tools: Read, Write, Bash, Grep, Glob
---

# ce:spec

PRD(제품 요구사항)와 SPEC(기술 명세)를 작성한다.

## 사용법

```
/ce:spec [@<code-path>] [--output <output-path>]
```

## Context Resolution

경로를 다음 순서로 결정한다:

1. `@<code-path>` 인자 있음 → `{CODE_PATH}` = 해당 경로
2. 없음 → 현재 디렉토리에서 `docs/prd.md` 또는 `docs/spec.md` 탐색
3. 탐색 실패 → "프로젝트 경로를 알려주세요 (예: `@/path/to/project`)"

`{OUTPUT_PATH}` = `--output` 지정 시 해당 경로, 없으면 `{CODE_PATH}/docs/`

## 기존 Artifacts 확인

**Knowledge Base 로드**: `{OUTPUT_PATH}/knowledge-base.md` 있으면 도메인 용어·제약을 컨텍스트로 활용한다.

**PRD·SPEC 탐지**: `{OUTPUT_PATH}/prd.md`, `{OUTPUT_PATH}/spec.md` 존재 시:

```
[현재 상태]
- prd.md: {존재하면 기능 N개, 없으면 "없음"}
- spec.md: {존재하면 아키텍처 결정 N개, 없으면 "없음"}

재작업(처음부터) 또는 갱신(기존 내용 유지·수정)?
```

## Phase 3-1: PRD

```markdown
# PRD — {PROJECT_NAME}

## 목적
{이 제품/기능이 해결하는 문제 — 한 문단}

## 사용 시나리오
{누가, 어떤 상황에서 사용하는가}

## 기능 요구사항
| 기능 | 성공 기준 | 우선순위 |
|------|---------|---------|
| {기능명}: {한 줄 설명} | {검증 가능한 기준} | High/Medium/Low |

## 제약
| 제약 | 영향 | 우선순위 |
|------|------|---------|
| {제약 설명} | {영향} | High/Medium/Low |
```

**산출물**: `{OUTPUT_PATH}/prd.md`

## Phase 3-2: SPEC

```markdown
# SPEC — {PROJECT_NAME}

## 아키텍처 결정
| 결정 사항 | 고려한 대안 | 선택 | 근거 |
|---------|-----------|------|------|
| {무엇을 결정했나} | {대안1}, {대안2} | {선택한 것} | {이유} |

## 패키지 구조
의존 방향: `진입점 → 애플리케이션 → 도메인 ← 인프라` (역방향 금지)
* 단순 프로젝트(CLI, 스크립트 등)는 이 구조 대신 프로젝트에 맞는 모듈 구조를 정의한다.

\`\`\`
{레이어}: {폴더 경로}  ← {담당 역할}
\`\`\`

## 데이터 흐름
\`\`\`
{입력} → {컴포넌트A} → {컴포넌트B} → {출력}
\`\`\`

## 구현 계획
| 순서 | 구현 대상 | 의존 | 병렬 가능 | 완료 기준 |
|------|---------|------|:-------:|---------|
| 1 | {모듈/기능} | — | — | {검증 가능한 기준} |
| 2 | {모듈/기능} | 1 | — | {검증 가능한 기준} |
```

**산출물**: `{OUTPUT_PATH}/spec.md`

> **완료**: "`prd.md`·`spec.md` 저장했습니다. `/ce:valid`로 Readiness Gate를 체크하거나 `/ce:impl`로 구현을 시작할 수 있습니다."
```

- [ ] **Step 3: 커밋 + PR**

```bash
git add skills/spec/SKILL.md
git commit -m "feat: /ce:spec — Phase 3 PRD + SPEC (#이슈번호)"
git push origin feature/{이슈번호}-spec
gh pr create --title "feat: /ce:spec" --body "Closes #{이슈번호}"
gh pr merge {PR번호} --merge --delete-branch
git checkout main && git pull origin main
```

---

### Task 4: /ce:impl SKILL.md

**Files:**
- Create: `/Users/sr/IdeaProjects/claude/ce/skills/impl/SKILL.md`

- [ ] **Step 1: Issue 생성 + 브랜치**

```bash
gh issue create --title "feat: /ce:impl — Phase 4 구현" \
  --body "SPEC 구현 계획 기반 단계별 구현 + 피드백 루프 sub-skill"
git checkout -b feature/{이슈번호}-impl
```

- [ ] **Step 2: SKILL.md 작성**

`/Users/sr/IdeaProjects/claude/ce/skills/impl/SKILL.md`:
```markdown
---
name: impl
description: >
  context-engineering Phase 4 — 단계별 구현.
  SPEC 구현 계획 기반으로 모듈 단위 구현 + 완료 기준 검증 + 피드백 루프 반복.
  Keywords: 구현, implementation, Phase 4, 빌드, 테스트, impl
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# ce:impl

SPEC 구현 계획을 기반으로 단계별 구현을 진행한다.

## 사용법

```
/ce:impl [@<code-path>] [--output <output-path>]
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
| 도메인 용어 오류, 새 제약 발견 | `knowledge-base.md` | `/ce:gather` |
| 새 gotcha 발견, AI 행동 규칙 변경 | `CLAUDE.md` | `/ce:gather` |
| 기능 요구사항 변경 | `prd.md` | `/ce:spec` |
| 아키텍처 결정 번복, 패키지 구조 변경 | `spec.md` | `/ce:spec` |

> **원칙**: 구현을 문서에 맞추지 말고, 문서를 현실에 맞춰 갱신한다.
```

- [ ] **Step 3: 커밋 + PR**

```bash
git add skills/impl/SKILL.md
git commit -m "feat: /ce:impl — Phase 4 구현 + 피드백 루프 (#이슈번호)"
git push origin feature/{이슈번호}-impl
gh pr create --title "feat: /ce:impl" --body "Closes #{이슈번호}"
gh pr merge {PR번호} --merge --delete-branch
git checkout main && git pull origin main
```

---

### Task 5: /ce:valid SKILL.md

**Files:**
- Create: `/Users/sr/IdeaProjects/claude/ce/skills/valid/SKILL.md`

- [ ] **Step 1: Issue 생성 + 브랜치**

```bash
gh issue create --title "feat: /ce:valid — Readiness Gate 체크" \
  --body "PRD·SPEC·KB·CLAUDE.md 상태를 점검하고 사용자가 Phase 4 진입 여부를 결정하는 sub-skill"
git checkout -b feature/{이슈번호}-valid
```

- [ ] **Step 2: SKILL.md 작성**

`/Users/sr/IdeaProjects/claude/ce/skills/valid/SKILL.md`:
```markdown
---
name: valid
description: >
  context-engineering Readiness Gate 체크.
  Phase 3 완료 후 Phase 4 진입 전, 또는 탐색 모드 종료 시점에 사용.
  AI는 현재 상태 요약만 제시하고 사용자가 통과 여부를 결정한다.
  Keywords: readiness gate, 검증, valid, gate, 스파이럴 종료, 체크포인트
allowed-tools: Read, Bash, Glob
---

# ce:valid

현재 프로젝트 artifacts 상태를 점검하고 Readiness Gate 결과를 사용자에게 제시한다.

## 사용법

```
/ce:valid [@<code-path>] [--output <output-path>]
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
모두 통과라면 "Phase 4 진행" 또는 "/ce:impl" 이라고 알려 주세요.
```

| 기준 | 미충족 시 |
|------|---------|
| ① PRD 성공 기준 누락 | `/ce:spec` 으로 PRD 보완 |
| ② SPEC 근거 누락 | `/ce:spec` 으로 SPEC 보완 |
| ③ 도메인 용어 미확정 | `/ce:gather` 로 KB 보완 |
| ④ CLAUDE.md 제약 미반영 | `/ce:gather` 로 CLAUDE.md 보완 |
| ⑤ 구현 계획 착수 불가 | `/ce:spec` 으로 SPEC 보완 |
```

- [ ] **Step 3: 커밋 + PR**

```bash
git add skills/valid/SKILL.md
git commit -m "feat: /ce:valid — Readiness Gate 체크 (#이슈번호)"
git push origin feature/{이슈번호}-valid
gh pr create --title "feat: /ce:valid" --body "Closes #{이슈번호}"
gh pr merge {PR번호} --merge --delete-branch
git checkout main && git pull origin main
```

---

### Task 6: README.md

**Files:**
- Create: `/Users/sr/IdeaProjects/claude/ce/README.md`

- [ ] **Step 1: Issue 생성 + 브랜치**

```bash
gh issue create --title "docs: README" --body "플러그인 설치·사용법·각 sub-skill 설명"
git checkout -b feature/{이슈번호}-readme
```

- [ ] **Step 2: README 작성**

`/Users/sr/IdeaProjects/claude/ce/README.md`:
```markdown
# ce

context-engineering 워크플로우의 특정 Phase에 직접 진입하는 Claude Code 플러그인.

## Sub-skills

| 명령 | 담당 | 사용 시점 |
|------|------|---------|
| `/ce:gather` | Step 0 + KB + CLAUDE.md | 새 프로젝트 시작, KB·CLAUDE.md 재작업 |
| `/ce:spec` | PRD + SPEC | 요구사항·기술 명세 작성·갱신 |
| `/ce:impl` | Phase 4 구현 | 구현 시작, 피드백 루프 진입 |
| `/ce:valid` | Readiness Gate | Phase 4 진입 전 검증, 탐색 모드 종료 체크 |

## 설치

```bash
claude plugin marketplace add https://github.com/SeokRae/ce.git
claude plugin install ce
```

로컬에서 바로 사용:

```bash
claude --plugin-dir /path/to/ce
```

## 사용법

```bash
# 새 프로젝트 시작 — 경로 직접 지정
/ce:gather @/path/to/project

# 현재 디렉토리에서 자동 탐색
/ce:spec

# Readiness Gate 체크
/ce:valid @/path/to/project

# 구현 시작
/ce:impl @/path/to/project
```

## 기존 스킬과의 관계

`/context-engineering` — 전체 워크플로우를 Step 0부터 Phase 4까지 순차 진행.  
`/ce:*` — 특정 Phase에 직접 진입. 이미 진행 중인 프로젝트의 재작업·피드백 루프에 사용.

## License

MIT
```

- [ ] **Step 3: 커밋 + PR**

```bash
git add README.md
git commit -m "docs: README — sub-skills 설명·설치·사용법 (#이슈번호)"
git push origin feature/{이슈번호}-readme
gh pr create --title "docs: README" --body "Closes #{이슈번호}"
gh pr merge {PR번호} --merge --delete-branch
git checkout main && git pull origin main
```

---

### Task 7: 설치 검증

- [ ] **Step 1: 로컬 플러그인 등록**

```bash
claude plugin marketplace add /Users/sr/IdeaProjects/claude/ce
claude plugin install ce
```

- [ ] **Step 2: 스킬 목록 확인**

Claude Code를 재시작 후 available skills 목록에 아래 4개가 나타나는지 확인:
```
ce:gather
ce:spec
ce:impl
ce:valid
```

- [ ] **Step 3: 동작 확인**

테스트 프로젝트로 각 스킬 기본 진입 확인:
```
/ce:gather                     → "프로젝트 경로를 알려주세요" 프롬프트 출력 확인
/ce:gather @/tmp/test-project  → Step 0 질문 1번 출력 확인
/ce:valid @/tmp/test-project   → Readiness Gate 출력 확인 (artifacts 없음 상태)
```
