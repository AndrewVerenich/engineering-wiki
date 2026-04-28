# Java Backend Fundamentals

Краткий **обзор** опорных идей для Java backend в этой вики. Раздел в стадии наполнения — состав тем зафиксирован, концепты добавляются по мере чтения источников.

## Опорные книги

- [Redis in Action](../sources/redis-in-action.md) — caching patterns, atomic operations, HA (replication/sentinel/cluster), data modeling под in-memory store.

Источники по JVM/Spring/Kotlin будут добавляться в `wiki/sources/` по мере прочтения. Ожидаемые направления:

- JVM internals (Java Performance, Java Concurrency in Practice или их аналоги)
- Spring (Spring in Action / документация Spring Framework и Spring Boot)
- Kotlin (Kotlin in Action, Kotlin Coroutines: Deep Dive или аналог)
- API design (REST API design rulebook, gRPC docs, system-design книги)
- Observability (OpenTelemetry docs, Distributed Tracing in Practice)

## Темы

### JVM internals (TBD)

Память (heap, metaspace, off-heap), GC (G1/ZGC/Shenandoah/Generational ZGC), JIT (C1/C2, профилирование, escape analysis), class loading, native memory tracking.

### Concurrency & Memory Model (TBD)

Java Memory Model, happens-before, `volatile`, `synchronized`, `java.util.concurrent` (executors, locks, atomics, queues), `CompletableFuture`, virtual threads (Loom), structured concurrency, reactive (Reactor/RxJava — где уместно).

### Spring (TBD)

IoC/DI, lifecycle бинов, AOP, Spring Boot (auto-configuration), Spring MVC vs WebFlux, Spring Data (JPA/JDBC/R2DBC/Redis), Spring Security (filters, OAuth2/JWT), Spring Cloud (config, gateway, circuit breakers).

### Caching & Redis

- [Redis](../entities/redis.md) — роль Redis в backend-архитектуре.
- [Redis Caching Patterns and Consistency](../concepts/redis-caching-patterns-and-consistency.md)
- [Redis Atomicity and Performance Patterns](../concepts/redis-atomicity-and-performance-patterns.md)
- [Redis Replication, Sentinel and Cluster](../concepts/redis-replication-sentinel-cluster.md)

### Kotlin (TBD)

Null-safety, data classes, sealed hierarchies, extension functions, **coroutines** (suspend, `Flow`, structured concurrency), interop с Java, идиомы для backend (DTO, value classes, DSL).

### Web & APIs (TBD)

HTTP semantics (методы, статусы, кэширование), REST design (idempotency, pagination, versioning), gRPC (protobuf, streaming), OpenAPI/Swagger, API contracts, retries и idempotency keys.

### Observability (TBD)

Метрики (Micrometer, Prometheus), tracing (OpenTelemetry, W3C Trace Context), structured logging, sampling, SLO/SLI/SLA, alerting.

## Связанные foundations (общие с DE)

Эти страницы лежат в зоне Foundations и одинаково важны для backend и DE:

### Distributed Systems & System Design

- [Reliability, Scalability, Maintainability](../concepts/reliability-scalability-maintainability.md)
- [Replication](../concepts/replication.md)
- [Partitioning](../concepts/partitioning.md)
- [Distributed Systems Pitfalls](../concepts/distributed-systems-pitfalls.md)
- [Consistency and Consensus](../concepts/consistency-and-consensus.md)
- [Encoding and Schema Evolution](../concepts/encoding-and-schema-evolution.md)

### Транзакции и СУБД

- [Transactions and Isolation](../concepts/transactions-and-isolation.md)
- [SQL Transactions, Locking and Isolation](../concepts/sql-transactions-locking-standard.md)
- [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md)
- [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md)

### SQL и моделирование

- [Data Models and Query Languages](../concepts/data-models-and-query-languages.md)
- [Normalization and Normal Forms](../concepts/normalization-and-normal-forms.md)
- [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md)

## Сущности

- [PostgreSQL](../entities/postgresql.md) — основная OLTP СУБД для backend.
- [Apache Kafka](../entities/apache-kafka.md) — event log / message bus, типичная часть backend-стека.
- [Redis](../entities/redis.md) — low-latency cache/state layer для backend.

## Связи

- [Data Engineering Fundamentals](data-engineering-fundamentals.md) — параллельный домен; foundations общие.
- [SCHEMA.md](../../schema/SCHEMA.md) — рубрики и правила вики.
