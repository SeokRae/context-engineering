# 아키텍처 원칙

Phase 3 패키지/모듈 구조 설계 시 참조한다.

## 왜 레이어를 나누는가 (SoC + Cohesion/Coupling)

- **High Cohesion**: 같은 이유로 변경되는 코드는 같은 모듈에
- **Low Coupling**: 다른 이유로 변경되는 코드는 다른 모듈로 분리
- **SoC**: 하나의 모듈은 하나의 관심사(도메인 규칙 / 기술 세부사항 / 입출력 변환)만

> SOLID는 이 원칙의 OOP 구체화. DDD는 도메인 관심사를 비즈니스 언어로 명시.

## 레이어 설계 (Ports & Adapters + DDD)

| 레이어 | Ports & Adapters | 책임 (SRP) | 의존 규칙 (DIP) | DDD 개념 |
|--------|-----------------|-----------|----------------|---------|
| 도메인 | — (순수 도메인) | 비즈니스 규칙·불변식 | 외부 의존 없음 (순수) | Entity, Value Object, Aggregate, Domain Service, Repository 포트(인터페이스) |
| 애플리케이션 | — (Use Case) | 유스케이스 조율 (CQS 적용) | 도메인만 | Application Service, Command Handler, Query Handler |
| 인프라 | Driven Adapter (Secondary) | 외부 시스템 구현체 | 도메인 포트 역전(DIP) | Repository 구현, External API Adapter |
| 진입점 | Driving Adapter (Primary) | 입력 수신·변환 | 애플리케이션만 | HTTP Controller, Message Consumer |
| 설정 | — | 의존성 조립 | 전 레이어 | DI 설정, 환경 변수 바인딩 |

의존 방향: `진입점(Primary) → 애플리케이션 → 도메인 ← 인프라(Secondary)` (역방향 금지)

## 책임 할당 — GRASP Information Expert

책임은 그 정보를 가진 객체에 둔다:
- 주문 총액 계산 → `Order` (항목 정보를 가진 쪽이 책임 — OrderService 아님)
- 할인 정책 적용 → `DiscountPolicy` (정책 규칙을 아는 쪽이 책임 — Order 아님)
- 재고 확인 → `Inventory` (재고 수량을 가진 쪽이 책임)

## CQS — Application Service 설계 기준

Application Service의 메서드는 둘 중 하나만:
- **Command**: 상태 변경 + 반환값 없음 — `createOrder()`, `cancelPayment()`
- **Query**: 상태 변경 없음 + 값 반환 — `getOrderById()`, `listPendingOrders()`

## OCP 적용 기준

- 비즈니스 규칙 변경 → 도메인 레이어만 수정
- 외부 API 교체 → 인프라(Secondary Adapter)만 수정 (도메인·애플리케이션 무변경)
- 새 유스케이스 추가 → 애플리케이션 레이어 확장 (기존 코드 닫힘)

## 언어별 폴더명 대응

| 레이어 | Java | Node.js | Python | Go |
|--------|------|---------|--------|----|
| 도메인 | `domain/` | `domain/` | `domain/` | `internal/domain/` |
| 애플리케이션 | `application/` | `services/` | `services/` | `internal/service/` |
| 인프라 | `infrastructure/` | `repositories/` | `infra/` | `internal/repository/` |
| 진입점 | `adapter/in/` | `routes/` | `api/` | `internal/handler/` |
| 설정 | `config/` | `config/` | `core/` | `internal/config/` |
