# Reading & Ingest Backlog

Список **источников** (книги, документация, статьи, курсы) и **тем** для будущего наполнения вики. Используется как roadmap: что прочитать дальше, что добавить в зоны и каталоги.

Формат:

- `[ ]` — не начато
- `[~]` — в процессе чтения / частичный ingest
- `[x]` — прочитано и заингежено в вики (со ссылкой на `log.md` запись)

---

## Java Backend (приоритет: высокий)

Зона `Java Backend` сейчас почти пустая — это главный gap под цель «senior Java engineer».

### JVM internals

- [ ] **Java Performance: The Definitive Guide** — Scott Oaks (или *Optimizing Java* — Ben Evans)  
  _Закроет:_ GC (G1/ZGC/Shenandoah), JIT (C1/C2, escape analysis), JMH, memory layout, native memory tracking.
- [ ] **The Garbage Collection Handbook** — Jones, Hosking, Moss _(deep dive, опционально)_

### Concurrency & memory model

- [ ] **Java Concurrency in Practice** — Brian Goetz  
  _Закроет:_ JMM, happens-before, `volatile`, `synchronized`, `j.u.c` (executors, locks, atomics, queues), безопасная публикация.
- [ ] **Modern Java in Action** — Urma, Fusco, Mycroft — section про CompletableFuture / Reactor / Loom.
- [ ] Документация по virtual threads (Project Loom) и structured concurrency (JEP 453, JEP 480).

### Effective Java / идиомы

- [ ] **Effective Java** — Joshua Bloch (3rd edition)  
  _Закроет:_ equals/hashCode, immutability, builders, generics, lambda/Stream идиомы — частые мелкие интервью-вопросы.

### Spring

- [ ] **Spring in Action** — Craig Walls (последнее издание)  
  _Закроет:_ IoC/DI, lifecycle бинов, AOP, Boot, MVC vs WebFlux, Spring Data, Security.
- [ ] **Spring Microservices in Action** — John Carnell _(если будет потребность по микросервисам)_
- [ ] Spring Framework Reference Documentation — `docs.spring.io` (раздел Transactions: propagation, rollback rules — обязательно).
- [ ] Spring Boot Reference (auto-configuration, externalized config, actuator).

### Kotlin

- [ ] **Kotlin Coroutines: Deep Dive** — Marcin Moskała  
  _Закроет:_ suspend, structured concurrency, `Flow`, dispatchers, exception handling.
- [ ] **Kotlin in Action** (2nd edition) — Jemerov, Isakova — основа языка.

### Web & APIs

- [ ] **Microsoft REST API Guidelines** (`github.com/microsoft/api-guidelines`) — REST design.
- [ ] **gRPC Documentation** — protobuf, streaming, deadlines.
- [ ] **OpenAPI Specification** — контракты.
- [ ] Статья «Idempotency, Stripe API» — pattern из реального production.

### Observability

- [ ] **OpenTelemetry Documentation**.
- [ ] **Distributed Tracing in Practice** — Parker, Spoonhower, Sigelman.
- [ ] **Practical Monitoring** — Mike Julian (или *Observability Engineering* — Majors, Fong-Jones, Miranda).

---

## Data Engineering (приоритет: средний — зона уже плотная)

### Уже заингежено
- [x] Designing Data-Intensive Applications — Kleppmann (см. `log.md`).
- [x] The Data Warehouse Toolkit — Kimball & Ross.
- [x] ClickHouse Official Documentation.
- [x] Stream Processing with Apache Flink — Hueske, Kalavri.
- [x] Kafka in Action.
- [x] Kafka Official Documentation.
- [x] Kafka Streams in Action, 2nd ed.

### Дальше

- [ ] **Fundamentals of Data Engineering** — Joe Reis & Matt Housley  
  _Закроет:_ data engineering lifecycle, source systems, undercurrents (security, observability, governance).
- [ ] **Streaming Systems** — Tyler Akidau, Slava Chernyak, Reuven Lax  
  _Закроет:_ event time vs processing time (более глубоко чем Hueske), watermarks, accumulation modes.
- [ ] **The Kimball Group Reader** — Kimball — расширенные кейсы (если будет глубокая тема).
- [ ] **Spark: The Definitive Guide** — Chambers, Zaharia _(если Spark станет приоритетом)_.
- [ ] **Trino: The Definitive Guide** — Fuller, Moser, Traverso _(если будет federated query на стеке)_.

---

## Foundations (приоритет: средний)

### Уже заингежено

- [x] PostgreSQL 17 изнутри — Егор Рогов.
- [x] SQL: The Complete Reference — Groff, Weinberg, Oppel.
- [x] Реляционные базы данных в примерах — Куликов.
- [x] Redis in Action — Carlson.
- [x] Redis Official Documentation.

### Дальше

- [ ] **Database Internals** — Alex Petrov  
  _Закроет:_ B-tree vs LSM глубже, storage engines, distributed transactions, consensus implementations.
- [ ] **Designing Distributed Systems** — Brendan Burns _(краткие паттерны Kubernetes-style)_.
- [ ] **Foundations of Scalable Systems** — Ian Gorton.
- [ ] Папка-сборник статей: Jepsen reports — `jepsen.io` (consistency violations).

---

## System Design (приоритет: высокий)

Раздел только что создан — нужны и **методология**, и **источники**.

- [ ] **System Design Interview, vol. 1** — Alex Xu  
  _Закроет:_ framework + 16 канонических кейсов (URL shortener, rate limiter, news feed, chat, search, notifications).
- [ ] **System Design Interview, vol. 2** — Alex Xu  
  _Закроет:_ payment system, S3, hotel reservation, real-time gaming leaderboard, distributed counter.
- [ ] **Building Microservices** (2nd ed.) — Sam Newman.
- [ ] **The Software Architect Elevator** — Gregor Hohpe.
- [ ] Сайт `bytebytego.com` — визуальные разборы (опционально, как дополнение).
- [ ] Блог Martin Fowler (`martinfowler.com`) — отдельные статьи по паттернам.

---

## Patterns (приоритет: высокий — каталог пустой)

Каталог `wiki/patterns/` — наполняется по мере нужды и из тех же источников.

### Источники паттернов

- [ ] **Enterprise Integration Patterns** — Hohpe, Woolf  
  _Закроет:_ messaging patterns (EIP — outbox, inbox, claim check, content-based router).
- [ ] **Release It!** (2nd ed.) — Michael Nygard  
  _Закроет:_ stability patterns (circuit breaker, bulkhead, timeout, fail-fast, steady state).
- [ ] **Patterns of Enterprise Application Architecture** — Martin Fowler  
  _Закроет:_ unit of work, identity map, repository, optimistic/pessimistic offline lock.
- [ ] **microservices.io** (Chris Richardson) — saga, outbox, transactional inbox, CQRS.

### Топ-патернов для интервью (вне зависимости от источника)

- [ ] Outbox / Transactional messaging
- [ ] Saga (orchestration vs choreography)
- [ ] Idempotency keys
- [ ] Circuit breaker
- [ ] Retry with exponential backoff + jitter
- [ ] Bulkhead
- [ ] Optimistic vs pessimistic locking
- [ ] CQRS / Event sourcing
- [ ] Distributed ID generation (Snowflake, UUIDv7)
- [ ] Strangler fig
- [ ] Rate limiting algorithms (token bucket, leaky bucket, sliding window log/counter)
- [ ] Leader election (lease-based, etcd/ZooKeeper)
- [ ] Anti-corruption layer
- [ ] BFF (Backend-for-Frontend)

---

## SQL Practice (приоритет: средний-высокий)

Раздел `wiki/sql-practice/` — упражнения для интервью.

### Источники

- [ ] **Practical SQL** — Anthony DeBarros _(если нужно базу)_.
- [ ] **SQL Antipatterns** — Bill Karwin.
- [ ] LeetCode Database section — топ-50 задач.
- [ ] StrataScratch / DataLemur — задачи DE-интервью реальных компаний.
- [ ] PostgreSQL Documentation — Window Functions, Recursive CTE главы.

### Топ-задач для тренировки

- [ ] Top-N per group (window vs LATERAL vs DISTINCT ON)
- [ ] Sessionization (gaps & islands)
- [ ] Retention cohort matrix
- [ ] Funnel из event-stream
- [ ] Recursive org chart / category tree traversal
- [ ] Pivot months → columns без расширений
- [ ] Median / percentiles в чистом SQL
- [ ] Running totals + window frames с разными PARTITION
- [ ] Detect duplicate / overlapping intervals
- [ ] SCD Type 2: SQL-реализация upsert (MERGE / DML с версионированием)
- [ ] Анализ медленного запроса по `EXPLAIN ANALYZE` → переписать
- [ ] DDL-задача: спроектировать схему под use case (e-commerce / messaging)

---

## Дополнительно (Foundations / cross-cutting)

- [ ] Algorithms & Data Structures — *Algorithms* (Sedgewick) или *Cracking the Coding Interview* (Gayle Laakmann McDowell) — для live-coding round'ов.
- [ ] **Designing Event-Driven Systems** — Ben Stopford _(Kafka-centric event-driven архитектура)_.
- [ ] **The Tao of Microservices** — Richard Rodger _(опционально)_.

---

## Принципы приоритезации

1. **Java Backend > всё остальное**, пока зона пустая (главный gap под цель).
2. **System Design + Patterns** идут параллельно: один кейс часто демонстрирует 3-5 паттернов сразу.
3. **SQL Practice** — добавлять по 1-2 задаче в неделю как warm-up, не блок-чтение.
4. **Foundations** — догнать Database Internals когда будет окно (нужно для углубления Postgres + DE).
5. **Не начинать новый источник, пока предыдущий не дошёл до состояния «концепты записаны, Q&A есть»**.

---

## Связи

- [Index](index.md)
- [System Design](system-design/README.md)
- [Patterns](patterns/README.md)
- [SQL Practice](sql-practice/README.md)
- [Log](log.md) — фактическая история ingest'ов.
