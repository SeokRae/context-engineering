# CLAUDE.md 정책 템플릿

> Phase 2 산출물. 이 템플릿을 기반으로 `{CODE_PATH}/CLAUDE.md`를 작성한다.
> 목표 줄 수: 100~150줄. 초과 항목은 별도 문서로 분리.

---

```markdown
# {PROJECT_NAME}

This file provides guidance to Claude Code when working with code in this repository.

> {프로젝트 한 줄 설명 — 무엇을 하는 서비스인가}

## 아키텍처

{패턴 이름}: {왜 이 패턴을 선택했는가}

**레이어 구조**

| 레이어 | 책임 | 외부 의존 허용 |
|--------|------|--------------|
| `config` | Bean 조립, 설정 바인딩 | Spring, 외부 라이브러리 |
| `adapter` | 외부 경계 변환 | `application` |
| `application` | 비즈니스 로직 | 없음 |
| `infrastructure` | 부가 기능 | 외부 시스템 |

의존 방향: adapter → application (역방향 금지)

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
