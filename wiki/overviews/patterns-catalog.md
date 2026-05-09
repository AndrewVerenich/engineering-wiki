# Patterns Catalog Overview

**Каркас каталога** [Patterns](../patterns/README.md). Раздел в стадии наполнения — конкретные паттерны добавляются по мере необходимости (см. [backlog](../backlog.md) → Patterns).

## Зачем эта страница существует

[`patterns/README.md`](../patterns/README.md) — это **landing-страница каталога**: что такое pattern, чем отличается от concept, шаблон страницы, категории.

Эта overview-страница — **roadmap по обучению**: в каком порядке изучать паттерны, какие cross-links между ними, какие прежде всего нужны для собеседований senior-уровня.

## Опорные источники

См. [backlog](../backlog.md) → раздел Patterns. Главные:

- *Enterprise Integration Patterns* (Hohpe, Woolf) — messaging.
- *Release It!* (Nygard) — stability patterns.
- *microservices.io* (Chris Richardson) — saga, outbox, CQRS.
- *Patterns of Enterprise Application Architecture* (Fowler).

## Порядок изучения (рекомендация)

### 1. Reliability & messaging (must-know на интервью)

Сначала пишутся как наиболее частые в реальных вопросах backend-собеседования.

- (TBD) Outbox / Transactional messaging
- (TBD) Idempotency keys
- (TBD) Retry with exponential backoff + jitter
- (TBD) Dead-letter queue strategies

### 2. Distributed transactions

- (TBD) Saga (orchestration vs choreography)
- (TBD) Compensating actions
- (TBD) 2PC vs TCC (обзор)

### 3. Resilience

- (TBD) Circuit breaker
- (TBD) Bulkhead
- (TBD) Timeout & deadline propagation

### 4. Consistency & state

- (TBD) Optimistic locking
- (TBD) Pessimistic locking
- (TBD) CQRS
- (TBD) Event sourcing

### 5. Scalability

- (TBD) Distributed ID generation (Snowflake, UUIDv7)
- (TBD) Sharding strategies
- (TBD) Leader election
- (TBD) Rate limiting algorithms

### 6. Architecture / migration

- (TBD) Strangler fig
- (TBD) Anti-corruption layer
- (TBD) BFF (backend-for-frontend)

## Кросс-связи

Паттерны редко существуют поодиночке. Типичные комбинации:

- **Reliable messaging:** Outbox + Idempotency + Retry + DLQ.
- **Distributed transaction:** Saga + Idempotency + Compensating actions.
- **Resilient client:** Circuit breaker + Bulkhead + Timeout + Retry with backoff.
- **High-throughput write:** Sharding + Distributed ID + Outbox.

(Эти комбинации станут отдельными секциями по мере наполнения каталога.)

## Связи

- [Patterns (каталог)](../patterns/README.md) — landing + шаблон.
- [System Design](../system-design/README.md) — кейсы используют паттерны.
- [Backlog](../backlog.md) — план добавления.
