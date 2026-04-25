# context-engineering

> [English](README.md) | 한국어

AI에게 컨텍스트를 전달하기 전에 체계적으로 준비하는 7-Phase 파이프라인 Claude Code 플러그인 — 어떤 정보를, 어떤 구조로, 어느 수준으로 압축해서 넣을지를 결정한다.

## Context Engineering이란?

Context Engineering은 **LLM의 컨텍스트 창에 무엇을 넣을지 설계하는 실천법**입니다. 단순히 좋은 프롬프트를 작성하는 것을 넘어, AI에게 어떤 정보를, 언제, 어떤 구조로 줄지를 결정합니다. RAG, 메모리 아키텍처, 토큰 예산 관리, 시스템 프롬프트 설계가 모두 여기에 해당합니다.

이 플러그인은 **지속적 컨텍스트 준비(persistent context preparation)** 를 다룹니다 — 컨텍스트를 수집하고, 선택하고, 구조화하고, 압축하고, 검증하는 반복 가능한 워크플로우로, 매 AI 세션이 정확히 필요한 정보로 시작할 수 있게 합니다.

## 파이프라인

```
Phase 1 --[G1]--> Phase 2 --[G2]--> Phase 3 --[G3]-->
Phase 4 --[G4]--> Phase 5 --[G5]--> Phase 6 --[G6]--> [출력]
                                                          |
                                                     (G6 실패)
                                                          v
                                                       Phase 7
```

| 서브스킬 | Phase | 역할 |
|---------|-------|------|
| `/context-engineering:gather`  | 1-3 | 문제 정의 → 후보 수집 → 선택 |
| `/context-engineering:build`   | 4-5 | 구조화 → 압축 |
| `/context-engineering:compose` | 6   | 실행 지시 생성 |
| `/context-engineering:verify`  | 7   | 최종 검증 |

각 게이트는 기준이 명확히 충족되면 자동 통과합니다. 모호한 경우 파이프라인이 멈추고 사용자 확인을 요청한 후 진행합니다.

## 출력 형식

Phase 6은 Phase 1 success criteria 신호에 따라 출력 형식을 자동 결정합니다:

| 형식 | 트리거 신호 | 출력 |
|------|-----------|------|
| **A — 실행 지시** | "프롬프트 만들어줘", "어떻게 써야 해", "지시해줘" | Role + Task + Constraints + Output Format |
| **B — KB 엔트리** | "저장해줘", "기억해줘", "노트해줘" | frontmatter + Key Facts / Constraints / Decisions / Notes |
| **C — 프로젝트 산출물** | "프로젝트 시작", "코드베이스 분석", "개발하고 싶어" | CLAUDE.md + spec.md + implementation plan |

신호가 모호한 경우 Phase 6이 A/B/C 중 하나를 선택하도록 한 번 묻습니다.

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
#     Version: 3.0.0
#     Scope: user
#     Status: ✔ enabled
```

로컬 디렉토리에서 직접 사용하려면:
```bash
claude --plugin-dir /path/to/context-engineering
```

## 사용법

### 전체 파이프라인

Phase 1부터 Phase 6까지 7개 단계를 순차적으로 실행합니다 (Phase 6 게이트 실패 시 Phase 7 자동 진입).

```
/context-engineering
```

### 서브스킬 — 특정 Phase 직접 진입

특정 Phase를 재작업하거나, 피드백 루프를 통해 돌아오거나, 이전 Phase가 이미 완료된 경우에 사용합니다.

| 커맨드 | Phase | 사용 시점 |
|--------|-------|----------|
| `/context-engineering:gather`  | 1-3 | 새 문제 정의, 또는 범위 재정의 / 소스 재수집 |
| `/context-engineering:build`   | 4-5 | 기존 컨텍스트 재구조화 또는 재압축 |
| `/context-engineering:compose` | 6   | 업데이트된 컨텍스트로 출력 재생성 |
| `/context-engineering:verify`  | 7   | 수동 검증 또는 `--consolidate` KB 유지보수 |

```
/context-engineering:gather
/context-engineering:build
/context-engineering:compose
/context-engineering:verify
/context-engineering:verify --consolidate
```

## 참고 문서

Phase 아티팩트 템플릿은 `skills/context-engineering/references/`에 있습니다:

| 파일 | 용도 |
|------|------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | 프로젝트별 7-Phase 진행 체크리스트 |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | Phase 1 KB 아티팩트 템플릿 |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | Phase 2 CLAUDE.md 템플릿 |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | Phase 3 SPEC 템플릿 (아키텍처 결정 + 패키지 구조) |
| [`architecture-principles.md`](skills/context-engineering/references/architecture-principles.md) | Hexagonal + DDD 원칙 (복잡한 서비스용, 선택 사항) |
| [`entry-template.md`](skills/context-engineering/references/entry-template.md) | KB 엔트리 형식 (frontmatter + 본문) |
| [`index-template.md`](skills/context-engineering/references/index-template.md) | KB 인덱스 형식 |
| [`context-session-template.md`](skills/context-engineering/references/context-session-template.md) | 다단계 태스크 컨텍스트 스크래치 파일 (완료 후 삭제) |

## 라이선스

MIT
