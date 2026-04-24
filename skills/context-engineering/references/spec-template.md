# 구현 Spec — {PROJECT_NAME}

> Phase 3 산출물. 무엇을 어떤 순서로 만들지 정의한다.

---

## 기능 요구사항 (FR-*)

| ID | 요구사항 | 수용 기준 | 우선순위 |
|----|---------|---------|---------|
| FR-1 | {기능명}: {한 줄 설명} | {검증 가능한 조건} | High |
| FR-2 | | | Medium |
| FR-3 | | | Low |

---

## 설계 결정 (D-*)

| ID | 결정 사항 | 고려한 대안 | 선택 | 근거 |
|----|---------|-----------|------|------|
| D-1 | {무엇을 결정했나} | {대안1}, {대안2} | {선택한 것} | {이유 — 제약 조건 참조} |
| D-2 | | | | |

---

## 패키지/모듈 구조

### 왜 레이어를 나누는가 (SoC + Cohesion/Coupling)

- **High Cohesion**: 같은 이유로 변경되는 코드는 같은 모듈에
- **Low Coupling**: 다른 이유로 변경되는 코드는 다른 모듈로 분리
- **SoC**: 도메인 규칙 / 기술 세부사항 / 입출력 변환은 각각 독립된 관심사

### 레이어 설계 (Ports & Adapters + DDD)

| 레이어 | Ports & Adapters | 책임 (SRP) | 의존 규칙 (DIP) | DDD 개념 |
|--------|-----------------|-----------|----------------|---------|
| 도메인 | — (순수 도메인) | 비즈니스 규칙·불변식 | 외부 의존 없음 (순수) | Entity, Value Object, Aggregate, Domain Service, Repository 포트(인터페이스) |
| 애플리케이션 | — (Use Case) | 유스케이스 조율 (CQS 적용) | 도메인만 | Application Service, Command Handler, Query Handler |
| 인프라 | Driven Adapter (Secondary) | 외부 시스템 구현체 | 도메인 포트 역전(DIP) | Repository 구현, External API Adapter |
| 진입점 | Driving Adapter (Primary) | 입력 수신·변환 | 애플리케이션만 | HTTP Controller, Message Consumer |
| 설정 | — | 의존성 조립 | 전 레이어 | DI 설정, 환경 변수 바인딩 |

의존 방향: `진입점(Primary) → 애플리케이션 → 도메인 ← 인프라(Secondary)` (역방향 금지)

**책임 할당 — GRASP Information Expert**: 책임은 그 정보를 가진 객체에 둔다.
- 주문 총액 계산 → `Order` (항목 정보를 가진 쪽)
- 할인 정책 적용 → `DiscountPolicy` (정책 규칙을 아는 쪽)

**CQS**: Application Service 메서드는 Command(상태 변경, void) 또는 Query(값 반환, 무변경) 둘 중 하나만.

**OCP**: 외부 API 교체 → Secondary Adapter만 수정. 새 유스케이스 → 애플리케이션 확장(기존 닫힘).

### 이 프로젝트의 구조

```
{레이어} [{Ports & Adapters 역할}]: {폴더 경로}  ← {담당 DDD 개념}
```

> 언어별 폴더명 참고 — Java: `domain/`, `application/`, `infrastructure/`, `adapter/in/`, `config/`
> / Node.js: `domain/`, `services/`, `repositories/`, `routes/`, `config/`
> / Python: `domain/`, `services/`, `infra/`, `api/`, `core/`
> / Go: `internal/domain/`, `internal/service/`, `internal/repository/`, `internal/handler/`, `internal/config/`

---

## 데이터 흐름

```
{입력} → {컴포넌트A} → {컴포넌트B} → {출력}
```

주요 흐름:
1. {흐름 1}: {설명}
2. {흐름 2}: {설명}

---

## WBS

| Phase | 모듈/작업 | 의존성 | 예상 산출물 |
|-------|---------|--------|-----------|
| P1 | {공통 모듈} | — | {jar/패키지} |
| P2 | {핵심 서버} | P1 | {실행 가능한 서비스} |
| P3 | {클라이언트/SDK} | P1, P2 | {라이브러리} |
| P4 | {통합 테스트} | P1~P3 | {테스트 결과} |

---

## 구현 순서

```
{모듈A} (공유 기반)
  └── {모듈B} (핵심 서버)
        ├── {모듈C} (클라이언트)
        └── {모듈D} (통합 테스트)
```

**이유**: {왜 이 순서인가 — 의존성 방향 기준}
