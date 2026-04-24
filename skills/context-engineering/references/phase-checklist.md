# Context Engineering 4단계 체크리스트

새 프로젝트 시작 시 이 파일을 복사해서 사용한다.

---

## 프로젝트 정보

- **프로젝트**: 
- **코드 경로**: 
- **출력 경로**: 
- **기술 스택**: 
- **시작일**: 

---

## Step 0: Context Gathering

- [ ] 요구사항 탐색 — 무엇을/왜/알려진 제약 확인
- [ ] 요구사항 요약 사용자 확인 (HARD-GATE 통과)
- [ ] 기술 파라미터 수집 (PROJECT_NAME, CODE_PATH, TECH_STACK 등)
- [ ] 파라미터 최종 요약 사용자 확인

## Phase 1: Knowledge Base

- [ ] Context Assessment — 시나리오 판별 (A/B/C/D) + 사용자 확인
- [ ] Source Scan 실행 (시나리오에 따라 조건부)
  - [ ] A: Step 0 대화에서 도메인 개념 추출
  - [ ] B: 스펙 문서 Explore 에이전트 병렬 탐색
  - [ ] C: 코드베이스 스캔 (패키지 구조, 패턴, 테스트 전략)
  - [ ] D: B + C 병렬 실행
- [ ] 도메인 용어 Glossary 작성 (최소 10개)
- [ ] 제약조건 레지스트리 작성 (TC/BL/OC 분류)
- [ ] 참조 문서 매니페스트 작성 (시나리오 B/C/D)
- [ ] `knowledge-base.md` 저장 (시나리오 A는 "초안" 표시)

## Phase 2: Policy (CLAUDE.md)

- [ ] CLAUDE.md 레벨 결정 (프로젝트 / 모듈)
- [ ] 아키텍처 규칙 정의 (레이어 분리, 패턴 제약)
- [ ] 코딩 컨벤션 문서화 (네이밍, 테스트 패턴)
- [ ] 빌드/테스트 명령어 정리
- [ ] Non-obvious gotchas 기록
- [ ] 150줄 초과 항목은 별도 문서 분리 + 링크
- [ ] `{CODE_PATH}/CLAUDE.md` 저장

## Phase 3: Spec

- [ ] 설계 결정 (D-*) — 대안/선택/근거 포함
- [ ] 패키지/모듈 구조 확정
- [ ] `spec.md` 저장

## Phase 4: Implementation

- [ ] 모듈별 스켈레톤 생성
- [ ] 핵심 도메인 객체 구현
- [ ] 어댑터 구현
- [ ] 테스트 작성 + 통과
- [ ] 피드백 루프 반영 (발견 내용 → 해당 단계 갱신)
  - [ ] 도메인/용어 오류 → Phase 1 `knowledge-base.md` 갱신
  - [ ] AI 행동 규칙 변경 → Phase 2 `CLAUDE.md` 갱신
  - [ ] 아키텍처·설계 변경 → Phase 3 `spec.md` 갱신
- [ ] 아키텍처 문서 작성
- [ ] 온보딩 가이드 작성
