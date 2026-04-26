# Context Source Strategy

> 파이프라인이 사용하는 4가지 컨텍스트 소싱 전략의 분류.

## 전략 분류

| 전략 | 정의 | 파이프라인 매핑 | 예시 |
|------|------|---------------|------|
| **RAG** (Retrieval-Augmented Generation) | 저장된 지식에서 질의 기반으로 관련 정보를 검색하여 주입 | Phase 2 KB 검색 (키워드 매칭 → 태그 확장 → 관련 엔트리 추적) | KB index.md에서 "transaction isolation" 검색 → 관련 엔트리 3개 로드 |
| **Memory** (Persistent Knowledge) | 세션 간 지속되는 구조화된 지식 저장소 | KB 시스템 전체 (Format B 저장 + `--consolidate` 유지보수) | `domain/transaction-isolation.md`에 fact 저장 → 다음 세션에서 검색 |
| **Tool Result** (Dynamic Injection) | 실행 시점에 도구를 호출하여 최신 상태를 동적으로 수집 | Phase 2 동적 컨텍스트 주입 (`git log`, `find`, `head`) | `git log --oneline -5`로 최근 변경사항 수집 → Keep/Skip 판단 근거 |
| **System Prompt** (Instruction Placement) | 조립된 컨텍스트를 모델이 권위 있는 지시로 읽는 구조화된 컨테이너에 배치; 기존 시스템 프롬프트(CLAUDE.md, rules/)는 Phase 2에서 소스로도 읽힘 | Phase 6 출력 (Format A 실행 지시문, Format C CLAUDE.md) + Phase 2 기존 프롬프트 읽기 | Format A: Role+Task+Constraints+Output Format 구조로 조립 → 모델에 주입; Format C: CLAUDE.md 생성 → 이후 세션마다 자동 주입 |

## Phase 2 소스 결정과의 매핑

Phase 2 "소스 자동 결정" 테이블의 각 행이 위 전략 중 하나에 매핑된다:

| Phase 1 신호 | 수집 소스 | 전략 |
|-------------|---------|------|
| 질문 / 태스크 요청 | KB `index.md` 스캔 → 매칭 엔트리 읽기 | **RAG** |
| 프로젝트 / 코드베이스 언급 | `git log`, `find`, 빌드 파일 | **Tool Result** |
| 문서 / 파일 / URL 제공 | 파일 전문 읽기 / URL 페치 | **RAG** (일회성) |
| "저장해줘" / 데이터 입력 | 입력 텍스트 전문 | **Memory** (쓰기) |
| 기존 CLAUDE.md / rules/ 존재 | 기존 시스템 프롬프트 파일 읽기 | **System Prompt** (읽기) |

> **권위 있는 소스 결정 로직은 `gather/SKILL.md` Phase 2를 참조한다.** 이 파일은 분류 체계만 정의한다.

> **System Prompt의 이중 역할**: 다른 3가지 전략은 컨텍스트의 *입력*만 다루지만, System Prompt는 입력(기존 CLAUDE.md/rules/ 읽기)과 출력(Phase 6에서 새 시스템 프롬프트 생성) 양쪽에 걸친다. Phase 2에서는 소스로, Phase 6에서는 최종 전달 형식으로 기능한다.
