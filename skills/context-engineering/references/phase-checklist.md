# Context Engineering 7-Phase 체크리스트

프로젝트 시작 시 복사하여 사용한다.

---

## 프로젝트 정보

- **프로젝트**: 
- **실행 날짜**: 
- **목적**: 

---

## gather (Phase 1-3)

### Phase 1. 문제 정의

- [ ] 질문 1 완료: purpose (해결하려는 문제)
- [ ] 질문 2 완료: constraints (필요한 배경 정보)
- [ ] 질문 3 완료: success criteria (결과물 형태)
- [ ] Scope Reduction 완료: 세션 범위 선언 블록 생성 (최소 문제 / Must-have / Nice-to-have / 제외)
- [ ] Role 자동 추론 완료
- [ ] G1 통과: purpose / constraints / success criteria / scope 모두 명확

### Phase 2. 컨텍스트 후보 수집

- [ ] Phase 1 신호 기반 소스 결정
- [ ] 결정된 소스에서 후보 수집 완료
- [ ] G2 통과: 필요한 소스 모두 수집, 명백한 누락 없음

### Phase 3. 컨텍스트 선택

- [ ] 관련성 / 최신성 / 신뢰성 기준 적용
- [ ] Keep / Skip / Merge 분류 완료
- [ ] G3 통과: 기준 일관 적용, Skip 이유 명확

---

## build (Phase 4-5)

### Phase 4. 컨텍스트 구조화

- [ ] Key Facts 섹션 작성
- [ ] Constraints 섹션 작성
- [ ] Decisions 섹션 작성
- [ ] Notes 섹션 작성 (해당 시)
- [ ] 빈 섹션 제거
- [ ] G4 통과: 모호한 항목 없음, 섹션 간 중복 없음

### Phase 5. 컨텍스트 압축

- [ ] 중복 항목 제거
- [ ] 접선 정보 한 줄 요약으로 대체
- [ ] Constraints / Decisions 원문 유지 확인
- [ ] G5 통과: Constraints / Decisions 보존 + 예산 준수 + 과압축 방지

---

## compose (Phase 6)

### Phase 6. 실행 지시 생성

- [ ] Phase 1 success criteria 신호 재확인
- [ ] 출력 형식 결정 (지시문 / KB 엔트리 / 프로젝트 산출물)
- [ ] 출력물 생성 완료
- [ ] G6 통과: success criteria 일치, 누락·충돌·추측 없음

**출력 형식**: ☐ 지시문  ☐ KB 엔트리  ☐ 프로젝트 산출물

---

## verify (Phase 7) — G6 실패 시 또는 수동 실행

### Phase 7. 최종 검증

- [ ] 누락 점검: Phase 1 purpose/constraints 반영 여부
- [ ] 충돌 점검: 출력물 내 모순 없음
- [ ] 추측 점검: 근거 없는 주장 없음
- [ ] 일관성 점검: 출력 형식과 success criteria 일치
- [ ] 신뢰도 표시: H / M / L
- [ ] 피드백 루프 필요 시 해당 Phase 복귀

---

## 자가 점검 기준

| Phase | 핵심 기준 |
|-------|---------|
| G1 | purpose / constraints / success criteria 3개 모두 명확 + 세션 범위 선언 블록 생성 |
| G2 | 필요한 소스 모두 수집, 명백한 누락 없음 |
| G3 | 관련성·최신성·신뢰성 일관 적용, Skip 이유 명확 |
| G4 | 모호한 항목 없음, 섹션 간 중복 없음 |
| G5 | Constraints / Decisions 보존, 예산 준수, 과압축 방지 |
| G6 | success criteria 일치, 누락·충돌·추측 없음 |
| G7 | 신뢰도 H/M/L 표시, 피드백 루프 완료 |
