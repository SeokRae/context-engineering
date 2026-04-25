# context-engineering

> 한국어 | [English](README.md)

AI 도구로 복잡한 서비스를 개발할 때 **"무엇을 알아야 하고, 어떻게 행동해야 하고, 무엇을 만들어야 하는가"** 를 단계별로 정의하는 Claude Code 플러그인.

## Context Engineering이란?

Context Engineering은 **LLM의 컨텍스트 창에 무엇을 넣을지 설계하는 실천법**입니다. 단순히 좋은 프롬프트를 작성하는 것을 넘어, AI에게 어떤 정보를, 언제, 어떤 구조로 줄지를 결정합니다. RAG, 메모리 아키텍처, 토큰 예산 관리, 시스템 프롬프트 설계가 모두 여기에 해당합니다.

이 플러그인은 그 중 **지속적 컨텍스트 준비(persistent context preparation)** 라는 측면을 다룹니다 — AI가 매 개발 세션을 올바른 도메인 지식과 행동 규칙으로 시작할 수 있도록 `knowledge-base.md`와 `CLAUDE.md`를 체계적으로 작성·유지하는 워크플로우입니다.

## 개요

AI로 복잡한 서비스를 개발할 때 세 가지 질문에 답해야 합니다:

- **무엇을 알아야 하는가** → Knowledge Base (도메인 지식, 제약 사항)
- **어떻게 행동해야 하는가** → Policy (CLAUDE.md)
- **무엇을 만들어야 하는가** → Spec (PRD + 아키텍처 결정 + 구현 계획)

이 플러그인은 두 가지 방식으로 사용할 수 있습니다:

- **`/context-engineering`** — Step 0부터 Phase 4까지 순차적 전체 흐름
- **서브스킬** — 특정 Phase에 직접 진입 (재작업 또는 피드백 루프용)

## 설치

```bash
# 1. 마켓플레이스에 추가
claude plugin marketplace add https://github.com/SeokRae/context-engineering.git

# 2. 플러그인 설치
claude plugin install context-engineering
```

설치 후 Claude Code를 재시작해야 변경 사항이 적용됩니다.

설치 확인:
```bash
claude plugin list
#   ❯ context-engineering@context-engineering
#     Version: 1.0.0
#     Scope: user
#     Status: ✔ enabled
```

로컬 디렉토리에서 직접 사용하려면:
```bash
claude --plugin-dir /path/to/context-engineering
```

## 스킬

### `/context-engineering` — 전체 워크플로우

Step 0부터 Phase 4까지 순차적으로 실행합니다.

```
/context-engineering @<코드-경로> [--output <출력-경로>] [--specs <스펙-문서-경로>]
```

**예시:**
```
/context-engineering @/path/to/project
/context-engineering @/path/to/project --specs docs/references/
/context-engineering @/path/to/project --output /tmp/context-docs --specs docs/references/
```

### 서브스킬 — Phase별 직접 진입

진행 중인 프로젝트에서 특정 Phase를 재작업하거나, 피드백 루프를 통해 Phase로 돌아올 때 사용합니다.

| 커맨드 | 담당 | 사용 시점 |
|--------|------|----------|
| `/context-engineering:gather` | Step 0 + Phase 1 (KB) + Phase 2 (CLAUDE.md) | 새 프로젝트 시작, KB 또는 CLAUDE.md 재작업/업데이트 |
| `/context-engineering:spec` | Phase 3 — PRD + SPEC | 요구사항 및 기술 스펙 작성 또는 업데이트 |
| `/context-engineering:impl` | Phase 4 — 구현 | 구현 시작, 피드백 루프 진입 |
| `/context-engineering:valid` | Readiness Gate | Phase 4 사전 검증, 탐색 모드 종료 확인 |

각 서브스킬은 기존 아티팩트(knowledge-base.md, CLAUDE.md, prd.md, spec.md)를 자동 감지하고 재작업 또는 업데이트 옵션을 제공합니다.

```
/context-engineering:gather @/path/to/project
/context-engineering:spec @/path/to/project
/context-engineering:valid
/context-engineering:impl @/path/to/project
```

## 워크플로우

> **[📊 인터랙티브 다이어그램](https://seokrae.github.io/context-engineering/context-engineering-cycle.html)** — 각 Phase를 클릭하면 단계별 상세 내용을 확인할 수 있습니다.

```
Step 0 — 컨텍스트 수집
  · 요구사항 탐색 (What · Why · 제약 — 질문 하나씩)
  · 요약 확인 시 REQ-ID 부여 (REQ-1 · REQ-2 · REQ-3…)
  · 명확    → Phase 1~3을 충실히 작성한 후 Phase 4 진입
  · 불명확  → 탐색 모드: 빈 초안 생성 → Phase 4로 직행
                 ↓
Phase 1 — Knowledge Base           [자체 평가] [사용자 확인]
  · 컨텍스트 평가 — A/B/C/D 시나리오 감지
  · 소스 스캔  A: 대화  B: 스펙 문서  C: 코드  D: B+C
  · knowledge-base.md (용어집 · 제약 사항 · 매니페스트)
                 ↓
Phase 2 — Policy (CLAUDE.md)       [자체 평가] [사용자 확인]
  · knowledge-base.md 연결 필수
  · 비직관적 주의사항 초안 (Phase 4 피드백으로 업데이트)
                 ↓
Phase 3 — Spec              [자체 평가] [Readiness Gate ⑥REQ]
  · PRD 기능 요구사항 + SPEC 아키텍처 결정 + 패키지 구조
  · 요구사항 추적성 (REQ-ID → PRD 기능 → SPEC → 구현 계획)
  · Gate: AI가 현재 상태 요약 → 사용자가 통과/실패 결정
  ↑──────────────────── 미충족 시 해당 Phase로 재순환
                 ↓  Gate 통과
Phase 4 — 구현                                [회고]
  · 빌드 + 테스트 + PRD 성공 기준 달성
  · 완료마다 REQ-ID 상태 업데이트 (미구현 → 구현됨)
  · 피드백 루프: 발견 사항 → 즉시 해당 Phase 문서 업데이트
  · 전부 완료 시 구현 회고 출력 (달성 · 프로세스 · 문서)
```

**탐색 모드**: 요구사항이 불명확할 때 Phase 1~3을 빈 초안으로 생성하고 Phase 4로 직행합니다. 개발하면서 채워나가다가 사용자가 Readiness Gate를 요청하면 메인 트랙으로 전환합니다.

## 참고 문서

Phase 아티팩트 템플릿은 `skills/context-engineering/references/`에 있습니다:

| 파일 | 용도 |
|------|------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | 프로젝트별 진행 체크리스트 |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | Phase 1 아티팩트 템플릿 |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | Phase 2 CLAUDE.md 템플릿 |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | Phase 3 SPEC 템플릿 (아키텍처 결정 + 패키지 구조) |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | Hexagonal + DDD 원칙 (복잡한 서비스용, 선택 사항) |

## 라이선스

MIT
