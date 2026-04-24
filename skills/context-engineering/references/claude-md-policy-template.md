# CLAUDE.md 정책 템플릿

> Phase 2 산출물. 이 템플릿을 기반으로 `{CODE_PATH}/CLAUDE.md`를 작성한다.
> 목표 줄 수: 100~150줄. 초과 항목은 별도 문서로 분리.

---

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {프로젝트 한 줄 설명 — 무엇을 하는 서비스인가}

## 아키텍처

**설계 근거 (SoC + Cohesion/Coupling)**: 같은 이유로 변경되는 코드는 같이, 다른 이유로 변경되는 코드는 분리. 하나의 모듈은 하나의 관심사만.

**레이어 구조 (Ports & Adapters + DDD)**

| 레이어 | 폴더 | Ports & Adapters | 책임 (SRP) | 의존 규칙 (DIP) | DDD 개념 |
|--------|------|-----------------|-----------|----------------|---------|
| 도메인 | `{경로}` | — (순수 도메인) | 비즈니스 규칙·불변식 | 외부 의존 없음 | Entity, Value Object, Aggregate, Repository 포트 |
| 애플리케이션 | `{경로}` | — (Use Case) | 유스케이스 조율 (CQS 적용) | 도메인만 | Command Handler, Query Handler |
| 인프라 | `{경로}` | Driven Adapter (Secondary) | 외부 시스템 구현체 | 도메인 포트 역전(DIP) | Repository 구현, API Adapter |
| 진입점 | `{경로}` | Driving Adapter (Primary) | 입력 수신·변환 | 애플리케이션만 | Controller, Consumer |
| 설정 | `{경로}` | — | 의존성 조립 | 전 레이어 | DI 설정 |

의존 방향: `진입점(Primary) → 애플리케이션 → 도메인 ← 인프라(Secondary)` (역방향 금지)

**책임 할당 (GRASP Information Expert)**: 책임은 그 정보를 가진 객체에. 계산 로직은 데이터를 가진 도메인 객체에 두고 Service로 위임하지 않는다.

**레이어 경계 규칙 (OCP)**: 외부 API 교체 → Secondary Adapter만 수정. 새 유스케이스 → 애플리케이션 확장, 기존 닫힘.

## 빌드 & 테스트

\`\`\`bash
# 전체 빌드
{BUILD_CMD}

# 단위 테스트
{TEST_CMD}

# 특정 모듈 테스트
{MODULE_TEST_CMD}

# 서버 실행
{RUN_CMD}
\`\`\`

## 포트 / 엔드포인트

| 포트 | 프로토콜 | 용도 |
|------|---------|------|
| {포트} | {TCP/HTTP} | {용도} |

## 핵심 제약

{Phase 1 TC/BL 중 개발에 직접 영향을 주는 항목만 — 최대 5개}

- TC-1: {제약}
- BL-1: {규칙}

## Non-obvious Gotchas

> 코드만 봐서는 알 수 없는 함정. 새 팀원이 반드시 알아야 할 것만.

- {함정 1}: {설명}
- {함정 2}: {설명}

## 참고 문서

- [{문서명}]({경로}): {한 줄 설명}
```

---

## 작성 기준

1. **WHAT/WHY/HOW**: 무엇을(아키텍처), 왜(설계 의도), 어떻게(명령어)
2. **줄 수 제한**: 프로젝트 레벨 100~150줄, 모듈 레벨 60~100줄
3. **초과 시**: 별도 `docs/WORKFLOW_*.md`로 분리하고 링크만 유지
4. **gotchas 우선**: 코드에서 읽히는 내용은 쓰지 않는다. 함정만.
5. **명령어 검증**: 실제 실행해서 동작 확인 후 기록
