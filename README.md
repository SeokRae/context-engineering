# context-engineering

AI 도구로 복잡한 서비스를 개발할 때 **"무엇을 알고, 어떻게 행동하고, 무엇을 만들어야 하는가"** 를 단계별로 정의하는 Claude Code 플러그인.

## 개요

복잡한 서비스를 AI와 함께 개발할 때 세 가지 질문에 답해야 한다:

- **무엇을 알아야 하는가** → Knowledge Base (도메인 지식, 제약)
- **어떻게 행동해야 하는가** → Policy (CLAUDE.md)
- **무엇을 만들어야 하는가** → Spec (PRD + 기술 결정 + 구현 계획)

이 플러그인은 두 가지 방식으로 사용할 수 있다:

- **`/context-engineering`** — Step 0부터 Phase 4까지 순차 진행 (전체 워크플로우)
- **Sub-skills** — 특정 Phase에 직접 진입 (재작업·피드백 루프용)

## 설치

```bash
# 1. 마켓플레이스 등록
claude plugin marketplace add https://github.com/SeokRae/context-engineering.git

# 2. 플러그인 설치
claude plugin install context-engineering
```

설치 후 Claude Code를 재시작해야 적용된다.

설치 확인:
```bash
claude plugin list
# context-engineering@context-engineering  Version: 1.0.0  Status: ✔ enabled
```

로컬 디렉토리에서 바로 사용할 때:
```bash
claude --plugin-dir /path/to/context-engineering
```

## 스킬

### `/context-engineering` — 전체 워크플로우

Step 0부터 Phase 4까지 순차 진행한다.

```
/context-engineering @<코드경로> [--output <출력경로>] [--specs <스펙문서경로>]
```

**예시:**
```
/context-engineering @/path/to/project
/context-engineering @/path/to/project --specs docs/references/
/context-engineering @/path/to/project --output /tmp/context-docs --specs docs/references/
```

### Sub-skills — Phase별 직접 진입

이미 진행 중인 프로젝트에서 특정 Phase만 재작업하거나, 피드백 루프로 특정 Phase로 돌아가는 경우 사용한다.

| 명령 | 담당 | 사용 시점 |
|------|------|---------|
| `/context-engineering:gather` | Step 0 + Phase 1 (KB) + Phase 2 (CLAUDE.md) | 새 프로젝트 시작, KB·CLAUDE.md 재작업·갱신 |
| `/context-engineering:spec` | Phase 3 — PRD + SPEC | 요구사항·기술 명세 작성·갱신 |
| `/context-engineering:impl` | Phase 4 — 구현 | 구현 시작, 피드백 루프 진입 |
| `/context-engineering:valid` | Readiness Gate | Phase 4 진입 전 검증, 탐색 모드 종료 체크 |

각 sub-skill은 기존 artifacts(knowledge-base.md, CLAUDE.md, prd.md, spec.md)가 있으면 자동 탐지해 재작업/갱신을 선택할 수 있다.

```
/context-engineering:gather @/path/to/project
/context-engineering:spec @/path/to/project
/context-engineering:valid
/context-engineering:impl @/path/to/project
```

## 워크플로우

> **[📊 인터랙티브 다이어그램](https://seokrae.github.io/context-engineering/context-engineering-cycle.html)** — 각 Phase를 클릭하면 단계별 작업 내용을 확인할 수 있다.

```
Step 0: Context Gathering
  ├─ 요구사항 명확  ─────────────────────────────────────┐
  └─ 요구사항 불명확 (탐색 모드) → 빈 초안 생성 ─────────┤
                                                         ↓
Phase 1: Knowledge Base
  1-1. Context Assessment — 시나리오 A/B/C/D 판별 후 사용자 확인
       A: 그린필드  B: 스펙 우선  C: 코드 우선  D: 풀 컨텍스트
       * 혼합 상태는 가장 가까운 시나리오 선택 후 사용자 확인
  1-2. Source Scan (시나리오별 조건부 실행)
  1-3. Glossary + 제약 (TC/BL/OC) + 매니페스트 → knowledge-base.md
  ↓  [사용자 확인]
Phase 2: Policy (CLAUDE.md)
  * knowledge-base.md 링크 포함 필수
  * 초안 성격 — Non-obvious Gotchas는 Phase 4에서 갱신
  ↓  [사용자 확인]
Phase 3: PRD + SPEC
  3-1. PRD — 기능 요구사항 + 검증 가능한 성공 기준
  3-2. SPEC — 아키텍처 결정 + 패키지 구조 + 구현 계획 (병렬 가능 표시)
  3-3. Readiness Gate — AI가 현재 상태 요약, 사용자가 통과 여부 결정
  ↑___________________↓ 미충족 시 해당 Phase 재순환
Phase 4: Implementation
  * 완료 기준: 빌드 + 테스트 + PRD 성공 기준 달성 확인
  * 피드백 루프: 발견 내용 → 해당 Phase 문서 즉시 갱신
```

**탐색 모드**: 요구사항 불명확 시 Phase 1~3을 빈 초안으로 생성하고 Phase 4로 바로 진입. 구현하며 채워나가다가 사용자가 직접 Readiness Gate를 요청해 본궤도로 전환한다.

## 참고 문서

각 Phase 산출물 템플릿은 `skills/context-engineering/references/` 에 있다:

| 파일 | 용도 |
|------|------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | 프로젝트별 진행 체크리스트 |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | Phase 1 산출물 템플릿 |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | Phase 2 CLAUDE.md 템플릿 |
| [`prd-template.md`](skills/context-engineering/references/prd-template.md) | Phase 3 PRD 템플릿 |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | Phase 3 SPEC 템플릿 (아키텍처 결정 + 구현 계획) |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | Hexagonal + DDD 원칙 (복잡한 서비스용, 선택 적용) |

## License

MIT
