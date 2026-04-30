![Version](https://img.shields.io/badge/version-3.1.0-blue)
![License](https://img.shields.io/github/license/SeokRae/context-engineering)
![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)

# context-engineering

> [English](README.md)

AI에게 컨텍스트를 전달하기 전에 체계적으로 준비하는 7-Phase 파이프라인 Claude Code 플러그인 — 어떤 정보를, 어떤 구조로, 어느 수준으로 압축해서 넣을지를 결정한다.

## 언제 / 어떻게 / 얼마나

| 질문 | 답변 |
|------|------|
| **언제** | 2개 이상의 소스에서 정보를 조합해야 하거나, 단순 복붙으로 AI가 맥락을 놓칠 때 |
| **어떻게** | `/context-engineering` 실행 → 3개 질문에 답하면 7-Phase 파이프라인이 자동 진행. 이전 Phase가 완료됐으면 서브스킬로 직접 진입 |
| **얼마나** | 입력 규모에 따라 자동 압축 — 아래 Token Budget 테이블 참조 |

## 의사결정 테이블

### Level 1: 컨텍스트 엔지니어링이 필요한가?

| 상황 | 필요? | 이유 |
|------|-------|------|
| 단일 질문, 추가 맥락 불필요 | No | 프롬프트만으로 충분 |
| 파일 1개 수정, 범위 명확 | No | 직접 지시가 더 빠름 |
| 2개 이상 소스에서 정보 조합 | **Yes** | 없으면 AI가 맥락을 놓침 |
| 이전 결정사항 참조 필요 | **Yes** | KB 검색으로 일관성 유지 |
| 프로젝트 시작 (CLAUDE.md + spec) | **Yes** | Format C로 프로젝트 산출물 생성 |
| 같은 유형의 프롬프트를 반복 작성 | **Yes** | Format A로 재사용 가능한 지시문 생성 |
| 도메인 지식 축적 | **Yes** | Format B로 KB 엔트리 저장 |

### Level 2: 어디서 진입하나?

| 시나리오 | 진입점 | 전제조건 |
|---------|-------|---------|
| 처음부터 시작 | `/context-engineering` | — |
| gather만 다시 실행 | `:gather` | — |
| Phase 1/3 산출물이 이미 존재 | `:build` | gather 산출물 필요 |
| 압축된 컨텍스트 블록이 존재 | `:compose` | build 산출물 필요 |
| 기존 출력물 재검증 | `:verify` | compose 산출물 필요 |
| KB 유지보수 | `:verify --consolidate` | 유지보수 모드 (파이프라인과 독립) |

> 각 서브스킬은 전제조건 산출물을 확인하고, 없으면 안내 메시지와 함께 종료한다.

### Level 3: 어떤 컨텍스트를 우선할까?

진입할 서브스킬이 결정됐으면, 작업 유형에 따라 우선할 컨텍스트 소스를 결정한다:

| 작업 유형 | 우선 컨텍스트 | 후순위 | 출력 형식 |
|----------|-------------|-------|----------|
| Q&A / 사실 확인 | 검색 중심: KB 엔트리, 출처 근거, 공신력 있는 문서 | 대화 이력, 추측성 노트 | — (직접 답변) |
| 프로젝트 셋업 | 아키텍처 원칙, CLAUDE.md 정책, spec 템플릿, 구현 계획 | 무관한 KB 엔트리 | C |
| 지식 베이스 엔트리 | 통합 확인, 중복 감지, 오래된 엔트리 확인 | 원본 텍스트 (이미 정제됨) | B |
| 실행 지시 | Role, Task, Constraints, Output Format | 장황한 배경 정보 (Phase 5에서 압축) | A |

> 이 테이블은 Phase 2 소스 선택과 Phase 5 압축 우선순위를 안내한다. 기반 분류(RAG / Memory / Tool Result / System Prompt)는 [context-source-strategy.md](skills/context-engineering/references/context-source-strategy.md) 참조.

## Token Budget Allocation

Phase 5는 수집된 컨텍스트를 목표 토큰 예산에 맞게 압축한다. 입력 규모에 따라 압축 등급이 결정된다:

| 규모 | 단어 수 기준 | 목표 압축률 | 전략 |
|------|-----------|----------|------|
| 소형 (단일 파일/질문) | < 500 단어 | 압축 불필요 | 그대로 통과 |
| 중형 (모듈/기능 단위) | 500–2,000 단어 | ≤ 50% | Notes 요약 → 중복 제거 |
| 대형 (프로젝트 전체) | 2,000+ 단어 | ≤ 30% | Notes 제거 → Key Facts 축약 |

**압축 순서** (먼저 잘리는 것부터):
1. Notes 섹션 — 요약 또는 제거
2. Key Facts — purpose 허용 범위 내 축약
3. Constraints / Decisions — **압축 금지** (예산 압박과 무관하게 원문 유지)

> 대형 소스(2,000+ 단어)는 Phase 5에서 점진적 로딩 적용: 헤더 먼저 스캔 → Phase 1 purpose와 관련된 섹션만 선택적 읽기.

## 특징

Context Engineering은 **LLM의 컨텍스트 창에 무엇을 넣을지 설계하는 실천법**입니다. 단순히 좋은 프롬프트를 작성하는 것을 넘어, AI에게 어떤 정보를, 언제, 어떤 구조로 줄지를 결정합니다. RAG, 메모리 아키텍처, 토큰 예산 관리, 시스템 프롬프트 설계가 모두 여기에 해당합니다.

- **7-Phase 게이트 파이프라인** — 각 Phase가 기준을 평가하여 자동 통과하거나 확인을 요청
- **4개의 모듈형 서브스킬** — 이전 Phase가 이미 완료된 경우 특정 Phase부터 직접 진입 가능
- **3가지 출력 형식** 자동 결정 — 의도 신호에 따라 (실행 지시 / KB 엔트리 / 프로젝트 산출물)
- **KB 유지보수 모드** — `--consolidate`로 중복·오래된·고아 엔트리 감지
- **순수 마크다운** — 런타임 의존성 없음, 빌드 단계 없음

## 빠른 시작

```bash
# 1. 마켓플레이스에 추가
claude plugin marketplace add https://github.com/SeokRae/context-engineering.git

# 2. 플러그인 설치
claude plugin install context-engineering
```

설치 후 Claude Code를 재시작해야 변경 사항이 적용됩니다.

**설치 확인:**
```bash
claude plugin list
#   ❯ context-engineering@context-engineering
#     Version: 3.1.0
#     Scope: user
#     Status: ✔ enabled
```

**첫 사용:** `/context-engineering`을 입력하고 목적, 제약사항, 성공 기준에 관한 3가지 질문에 답하세요. 파이프라인이 나머지를 처리합니다.

로컬 디렉토리에서 직접 사용하려면:
```bash
claude --plugin-dir /path/to/context-engineering
```

## 파이프라인

```
Phase 1 --[G1]--> Phase 2 --[G2]--> Phase 3 --[G3]-->
Phase 4 --[G4]--> Phase 5 --[G5]--> Phase 6 --[G6]--> [출력]
                                                          |
                                                     (G6 실패)
                                                          v
                                                       Phase 7
```

> **인터랙티브 시각화** → [context-engineering-cycle.ko.html](https://seokrae.github.io/context-engineering/context-engineering-cycle.ko.html)

| 서브스킬 | Phase | 역할 |
|---------|-------|------|
| `/context-engineering:gather` | 1–3 | 문제 정의 (3가지 질문) → 후보 수집 → 선택 (Keep/Skip/Merge) |
| `/context-engineering:build` | 4–5 | Key Facts/Constraints/Decisions/Notes로 구조화 → 토큰 예산 기반 압축 |
| `/context-engineering:compose` | 6 | 형식(A/B/C) 자동 결정 후 출력 생성 |
| `/context-engineering:verify` | 7 | 4가지 검증 (누락/충돌/추측/일관성) + 신뢰도 평가 |

각 게이트는 기준이 명확히 충족되면 자동 통과합니다. 모호한 경우 파이프라인이 멈추고 사용자 확인을 요청합니다.

## 출력 형식

Phase 6은 Phase 1 success criteria 신호에 따라 출력 형식을 자동 결정합니다:

| 형식 | 트리거 신호 | 출력 |
|------|-----------|------|
| **A — 실행 지시** (System Prompt) | "프롬프트 만들어줘", "어떻게 써야 해", "지시해줘" | Role + Task + Constraints + Output Format — 대상 AI에 주입되는 시스템 프롬프트로 조립 |
| **B — KB 엔트리** (Memory) | "저장해줘", "기억해줘", "노트해줘" | frontmatter + Key Facts / Constraints / Decisions / Notes |
| **C — 프로젝트 산출물** (Persistent System Prompt) | "프로젝트 시작", "코드베이스 분석", "개발하고 싶어" | CLAUDE.md (영구 시스템 프롬프트) + spec.md + implementation plan |

신호가 모호한 경우 Phase 6이 A/B/C 중 하나를 선택하도록 한 번 묻습니다.

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
| `/context-engineering:gather` | 1–3 | 새 문제 정의, 또는 범위 재정의 / 소스 재수집 |
| `/context-engineering:build` | 4–5 | 기존 컨텍스트 재구조화 또는 재압축 |
| `/context-engineering:compose` | 6 | 업데이트된 컨텍스트로 출력 재생성 |
| `/context-engineering:verify` | 7 | 수동 검증 또는 `--consolidate` KB 유지보수 |

```
/context-engineering:gather
/context-engineering:build
/context-engineering:compose
/context-engineering:verify
/context-engineering:verify --consolidate
```

## 참고 템플릿

Phase 아티팩트 템플릿은 `skills/context-engineering/references/`에 있습니다:

| 파일 | Phase | 용도 |
|------|-------|------|
| [`phase-checklist.md`](skills/context-engineering/references/phase-checklist.md) | 전체 | 7-Phase 진행 체크리스트 |
| [`knowledge-base-template.md`](skills/context-engineering/references/knowledge-base-template.md) | 6-C | KB 아티팩트 템플릿 (Format C 산출물) |
| [`claude-md-policy-template.md`](skills/context-engineering/references/claude-md-policy-template.md) | 6-C | CLAUDE.md 템플릿 (Format C 산출물) |
| [`spec-template.md`](skills/context-engineering/references/spec-template.md) | 6-C | 아키텍처 결정 + 패키지 구조 (Format C 산출물) |
| [`entry-template.md`](skills/context-engineering/references/entry-template.md) | 6-B | KB 엔트리 형식 (frontmatter + 본문) |
| [`index-template.md`](skills/context-engineering/references/index-template.md) | 6-B | KB 인덱스 형식 |
| [`context-session-template.md`](skills/context-engineering/references/context-session-template.md) | 6-A | 다단계 태스크 스크래치 파일 (완료 후 삭제) |
| [`context-source-strategy.md`](skills/context-engineering/references/context-source-strategy.md) | 2 | RAG / Memory / Tool Result / System Prompt 전략 분류 |
| [`failure-cases.md`](skills/context-engineering/references/failure-cases.md) | 전체 | 파이프라인 실패 시나리오 + 복구 프로토콜 |

## 프로젝트 구조

```
context-engineering/
├── .claude-plugin/
│   ├── plugin.json          # 플러그인 메타데이터 (v3.1.0)
│   └── marketplace.json     # 마켓플레이스 등록
├── skills/
│   ├── context-engineering/
│   │   ├── SKILL.md         # 오케스트레이터 — 전체 7-Phase 파이프라인
│   │   └── references/      # 10개 아티팩트 템플릿 및 참조 문서
│   ├── gather/SKILL.md      # Phase 1–3
│   ├── build/SKILL.md       # Phase 4–5
│   ├── compose/SKILL.md     # Phase 6
│   └── verify/SKILL.md      # Phase 7
├── README.md
├── README.ko.md
└── LICENSE
```

## 기여

1. 저장소를 Fork하고 Clone하세요
2. 관련 `SKILL.md` 파일을 수정하세요
3. 실제 Claude Code 세션에서 `/context-engineering`을 실행하여 변경 사항을 테스트하세요
4. Phase 1 질문이 정상적으로 나타나고 게이트가 올바르게 동작하는지 확인하세요
5. Pull Request를 제출하세요

스킬 수정 시 [`CLAUDE.md`](CLAUDE.md)의 가이드라인을 따르세요:
- Phase 일관성 유지 (gather → build → compose → verify)
- `references/` 템플릿과 SKILL.md 인라인 구조를 항상 동기화
- 각 SKILL.md의 핵심 행동 계약을 첫 5,000 토큰 내에 배치 (압축 안전성)

## 라이선스

[MIT](LICENSE)
