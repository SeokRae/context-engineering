# Architecture Principles

**Optionally** referenced during Phase 3 package/module structure design.

> **Applies to**: Services requiring external API integration, complex business rules, or a domain model.
> Simple CLI scripts, data pipelines, and analysis tools should define their own structure directly instead of following these principles.

## When to use these principles

| Project type | Recommended | Alternative |
|-------------|:-----------:|-------------|
| REST API (many external dependencies) | ✅ | — |
| Service with complex domain logic | ✅ | — |
| CLI scripts, one-off tools | ❌ | Simple module separation |
| ML pipelines, data analysis | ❌ | Stage-based structure |
| Library / SDK | ⚠️ | Public API-centric structure |

---

## Why separate layers (SoC + Cohesion/Coupling)

- **High Cohesion**: Code that changes for the same reason belongs in the same module
- **Low Coupling**: Code that changes for different reasons should be separated into different modules
- **SoC**: Each module handles a single concern (domain rules / technical details / input-output transformation)

> SOLID is the OOP concretization of these principles. DDD expresses domain concerns in business language.

## Layer design (Ports & Adapters + DDD)

| Layer | Ports & Adapters | Responsibility (SRP) | Dependency rule (DIP) | DDD concepts |
|-------|-----------------|---------------------|----------------------|-------------|
| domain | — (pure domain) | Business rules and invariants | No external dependencies (pure) | Entity, Value Object, Aggregate, Domain Service, Repository port (interface) |
| application | — (Use Case) | Use case orchestration (CQS applied) | domain only | Application Service, Command Handler, Query Handler |
| infrastructure | Driven Adapter (Secondary) | External system implementations | Inverted via domain port (DIP) | Repository implementation, External API Adapter |
| entry | Driving Adapter (Primary) | Input reception and transformation | application only | HTTP Controller, Message Consumer |
| config | — | Dependency assembly | All layers | DI configuration, environment variable binding |

Dependency direction: `entry (Primary) -> application -> domain <- infrastructure (Secondary)` (reverse direction forbidden)

## Responsibility assignment — GRASP Information Expert

Assign responsibility to the object that holds the relevant information:
- Order total calculation → `Order` (the side that holds item data is responsible — not OrderService)
- Discount policy application → `DiscountPolicy` (the side that knows the policy rules is responsible — not Order)
- Inventory check → `Inventory` (the side that holds stock quantity is responsible)

## CQS — Application Service design criteria

Each Application Service method does exactly one of the following:
- **Command**: state change + no return value — `createOrder()`, `cancelPayment()`
- **Query**: no state change + returns a value — `getOrderById()`, `listPendingOrders()`

## OCP application criteria

- Business rule change → modify domain layer only
- External API replacement → modify infrastructure (Secondary Adapter) only (domain and application unchanged)
- New use case addition → extend application layer (existing code remains closed)

## Language-specific folder names

| Layer | Java | Node.js | Python | Go |
|-------|------|---------|--------|----|
| domain | `domain/` | `domain/` | `domain/` | `internal/domain/` |
| application | `application/` | `services/` | `services/` | `internal/service/` |
| infrastructure | `infrastructure/` | `repositories/` | `infra/` | `internal/repository/` |
| entry | `adapter/in/` | `routes/` | `api/` | `internal/handler/` |
| config | `config/` | `config/` | `core/` | `internal/config/` |
