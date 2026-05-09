# LLM Wiki Schema

Эта вики — персональная база знаний инженера, ориентированная на два рабочих домена: **Java Backend** и **Data Engineering**. Часть материала (распределённые системы, транзакции, SQL, моделирование данных) одинаково нужна обоим доменам — он живёт в общей зоне **Foundations**. Дополнительно есть **System Design** — отдельная зона для архитектурных кейсов и методологии собеседований.

Кроме зон, в вики есть **горизонтальные каталоги** — `Patterns` (cross-zone каталог паттернов) и `SQL Practice` (раздел упражнений). Они не привязаны к одной зоне; конкретная зона указывается на самой странице (для паттерна) или вытекает из категории задачи (для SQL practice).

LLM полностью отвечает за создание, обновление и поддержание всех страниц.
Задача пользователя — добавлять источники (книги, статьи, курсы) и задавать вопросы.
Задача LLM — поддерживать структуру, связи, целостность и **зональное размещение** материала.

---

# 1. Wiki Zones (главное)

Wiki разбита на **четыре параллельные зоны**. Каждая страница принадлежит **ровно одной зоне** — дублирование ссылок в `index.md` не делается. Пересечения между доменами выражаются через секцию `## Связи` внутри страниц.

## 1.1 Foundations (общее)

Материал, нужный одинаково и Java backend, и data engineer.

Подсекции:

- **Distributed Systems & System Design** — DDIA-distributed (replication, partitioning, consistency, distributed pitfalls), encoding/schema evolution.
- **Transactions, Isolation & Storage Internals** — теория транзакций, SQL standard locking, MVCC, PG/MySQL internals.
- **SQL & Relational Modeling** — модели данных, нормальные формы, выполнение запросов и индексы.

## 1.2 Data Engineering

DWH, batch/stream processing, моделирование витрин, инструменты данных.

Подсекции:

- **DDIA — главы по обработке данных** (storage and retrieval, batch, stream, derived data, OLTP vs OLAP).
- **Dimensional modeling (Kimball)** — звёздные/snowflake схемы, dimension tables, SCD, conformed dimensions.
- **ClickHouse** — engines, schema, ingestion, optimization.
- **dbt** — слои проекта, applied patterns, ELT vs ETL.
- **Spark** — batch + medallion, structured streaming.
- **Stream Processing (Flink)** — архитектура, DataStream API, time/watermarks, windows, state, checkpointing, exactly-once.

## 1.3 Java Backend

JVM, Spring, Kotlin, concurrency, web/API (раздел в стадии наполнения).

Подсекции (планируемые):

- **JVM internals** — GC, JIT, class loading, memory model.
- **Concurrency & Memory Model** — JMM, `java.util.concurrent`, virtual threads, structured concurrency.
- **Spring** — IoC, AOP, Boot, MVC/WebFlux, Data, Security, Cloud.
- **Kotlin** — null-safety, coroutines, идиомы.
- **Web & APIs** — REST/HTTP semantics, gRPC, OpenAPI, idempotency.
- **Observability** — metrics (Micrometer), tracing (OpenTelemetry), logging.

## 1.4 System Design

Cross-cutting зона, ориентированная на интервью senior-уровня и архитектурные решения. Здесь живёт **методология** проектирования и **разборы кейсов** (X — спроектировать).

Подсекции:

- **Framework & methodology** — алгоритм собеседования, capacity estimation, чеклисты non-functional requirements, карта типичных subsystems.
- **Cases** — конкретные системы (URL shortener, rate limiter, distributed cache, real-time analytics, payment system, CDC pipeline и т. д.).

System Design — отдельная зона, потому что:

- Кейс не является «концептом» — это **применение многих концептов сразу** (replication + partitioning + caching + queueing + transactions + observability в одной задаче).
- На senior-собеседовании System Design — **отдельный round**, отличный от доменных вопросов.
- Кейсы пересекают и Java Backend, и Data Engineering, и Foundations.

Контент live'ит в `wiki/system-design/` (cases) и `wiki/overviews/system-design-fundamentals.md` (framework).

## 1.5 Зональный маршрутизатор (как принимать решение)

Когда возникает новая concept-страница, LLM решает зону по этой матрице:

| Тема страницы | Зона |
|---------------|------|
| Replication, partitioning, consensus, CAP, distributed time, fault models | Foundations / Distributed Systems |
| Schema evolution, кодирование (Avro/Protobuf/JSON) | Foundations / Distributed Systems |
| Transactions, isolation levels, locking, deadlocks, MVCC | Foundations / Transactions |
| PostgreSQL/MySQL/SQL Server internals, сравнения СУБД | Foundations / Transactions |
| SQL execution, indexes, normalization, data models | Foundations / SQL |
| Storage engines (LSM, B-tree, columnar) — теория | Foundations / Distributed Systems |
| OLTP/OLAP, DWH, dimensional modeling, SCD | Data Engineering |
| ClickHouse, Spark, dbt, Airflow, Flink, Kafka Streams | Data Engineering |
| Batch/stream processing, derived data, CDC, lakehouse | Data Engineering |
| JVM, GC, JIT, classloaders | Java Backend / JVM |
| Java Memory Model, concurrency primitives, virtual threads | Java Backend / Concurrency |
| Spring (любой модуль) | Java Backend / Spring |
| Kotlin coroutines, idioms, DSL | Java Backend / Kotlin |
| REST/gRPC/OpenAPI design, HTTP semantics, idempotency | Java Backend / Web & APIs |
| Micrometer, OpenTelemetry, logging patterns | Java Backend / Observability |
| **Проектирование end-to-end системы, capacity estimation, architectural case** | **System Design / Cases** |
| **System design framework, методология собеседования, чеклисты** | **System Design / Framework** |
| Apache Kafka (entity), сравнения брокеров | Foundations + DE (entity, не concept — лежит в `entities/`, отображается в обоих доменах) |

**Если страница попадает в две зоны** — выбирается та, где она наиболее «корневая» (где её используют для понимания базы), а в другой зоне ставится cross-link через `## Связи`. Пример: `encoding-and-schema-evolution.md` — в Foundations (нужно и для distributed systems, и для serialization в backend, и для DE), а Avro/Protobuf-специфика для DE упоминается через ссылки.

**System Design vs concept:** если страница описывает **как работает** механизм (например, «consistent hashing») — это **concept** в Foundations. Если страница **проектирует систему** (например, «distributed cache» — задача с requirements, capacity estimation, trade-offs) — это **system design case**, даже если внутри обсуждается тот же consistent hashing.

## 1.6 Multi-zone ingest

**Одна книга может породить страницы в разных зонах.** Это ожидаемо. Пример:

- *Designing Data-Intensive Applications* → Foundations (replication, partitioning, transactions, consistency, encoding) + Data Engineering (storage and retrieval, batch, stream, derived data).
- *PostgreSQL 17 изнутри* → Foundations (MVCC, locking, query execution).
- *Stream Processing with Apache Flink* → Data Engineering целиком.
- (будущее) *Java Concurrency in Practice* → Java Backend / Concurrency, частично Foundations (memory model — пограничная).
- (будущее) *System Design Interview* (Alex Xu) → System Design (cases) + часть глав даст паттерны в каталог.
- (будущее) *Release It!* (Nygard) → Patterns (catalog: circuit breaker, bulkhead, timeout) + cross-links в Java Backend.

При ingest LLM сам определяет, в какую зону отнести каждый порождённый концепт.

---

# 2. Types of pages

## 2.1 Source pages
Страницы отдельных источников (книги, статьи, курсы, видео).
Хранятся в `wiki/sources/`.
**Зона:** источник может относиться к одной или нескольким зонам — это указывается в `wiki/index.md` рядом со ссылкой (см. таблицу источников в index).

## 2.2 Concept pages
Страницы концептов, технологий, паттернов.
Хранятся в `wiki/concepts/`.
**Зона:** ровно одна зона. Решение — через зональный маршрутизатор (1.5).

**Concept vs Pattern:** концепт описывает **как устроен механизм** (MVCC, watermarks, ISR). Паттерн описывает **решение типовой проблемы** (outbox, saga, idempotency). Если страница ближе ко «второму» — она идёт в `wiki/patterns/` (см. 2.6), а не в `concepts/`.

Примеры:
- JVM Memory Model → Java Backend
- Spring IoC → Java Backend
- Kotlin Coroutines → Java Backend
- CAP Theorem → Foundations / Distributed Systems
- Event Sourcing → Foundations / Distributed Systems (теория) или Data Engineering (применение в pipeline) — выбирается по контексту источника
- OLTP vs OLAP → Data Engineering
- Kafka Partitions → Foundations / Distributed Systems

## 2.3 Entity pages
Конкретные сущности:
- фреймворки (Spring Boot, Micronaut)
- инструменты (Kafka, Spark, Flink, Airflow)
- базы данных (Postgres, ClickHouse)

Хранятся в `wiki/entities/`.
**Зона:** entity-страница появляется в `index.md` в секции `## Entities` с пометкой зоны (Foundations / DE / backend / mixed). Она может ссылаться из обеих доменных секций через cross-link.

## 2.4 Overview pages
Обзоры тем и learning tracks.

Хранятся в `wiki/overviews/`.

Главные overview — по одному на домен:
- `wiki/overviews/java-backend-fundamentals.md` — каркас Java Backend.
- `wiki/overviews/data-engineering-fundamentals.md` — каркас Data Engineering, включает ссылку на Foundations.

Дополнительные overview — interview prep, learning roadmaps по подтемам (`dimensional-modeling-interview-prep`, `dbt-learning-roadmap`, `clickhouse-learning-track`).

## 2.5 Comparison pages
Сравнения X vs Y.
Хранятся в `wiki/comparisons/`.
**Зона:** та же, что у сравниваемых концептов; если объекты в разных зонах — comparison идёт в зону «более узкоспециализированного» из них (например, Flink vs Spark Streaming → Data Engineering / Stream).

## 2.6 Pattern pages
Страницы архитектурных и интеграционных паттернов: outbox, saga, circuit breaker, idempotency keys и т. п.
Хранятся в `wiki/patterns/`.

**Зона:** паттерн **cross-zone** — один и тот же outbox используется и в Java Backend, и в Data Engineering. Зона указывается на самой странице через поле `**Зона:**` в шапке. В `wiki/index.md` паттерн появляется в секции `## Patterns` (а не в зональной секции) — это горизонтальный каталог.

**Pattern vs Concept:** концепт = «как устроен механизм X», паттерн = «как решается проблема Y». Когда сомнение — паттерн пишется как `## Проблема → ## Решение → ## Реализация → ## Trade-offs → ## Когда не применять`. Концепт — `## Суть → ## Внутренности → ## Trade-offs → ## Q&A`.

Шаблон pattern page — см. [wiki/patterns/README.md](../wiki/patterns/README.md).

## 2.7 System Design pages
Страницы кейсов system design: проектирование конкретной системы под собеседование senior-уровня.
Хранятся в `wiki/system-design/`.

**Зона:** все страницы принадлежат зоне System Design (1.4). В `wiki/index.md` они живут в секции `## System Design`.

**System Design case vs concept:** если страница описывает **как работает** механизм — это concept. Если страница **проектирует систему** (functional + non-functional requirements + capacity + API + data model + high-level design + trade-offs) — это system design case.

Кейс **использует** концепты и паттерны через секцию `## Связи`, но не пересказывает их. Если в процессе разбора кейса возникает новый общеприменимый концепт — он выделяется в отдельную страницу `wiki/concepts/` (или `wiki/patterns/`).

Шаблон system design case — см. [wiki/system-design/README.md](../wiki/system-design/README.md).

## 2.8 SQL Practice pages
Страницы практических SQL-задач: формулировка → решение → объяснение → план выполнения → альтернативы.
Хранятся в `wiki/sql-practice/`.

**Зона:** не относятся к зонам через зональный маршрутизатор. Это горизонтальный раздел упражнений; в `wiki/index.md` живут в секции `## SQL Practice`.

**SQL Practice vs Concept:** концепт-страницы (`sql-query-execution-and-indexes`, `sql-transactions-locking-standard`) объясняют **теорию**. SQL Practice — это **прикладные задачи**, формат интервью на DE/senior-backend позиции. Каждая задача ссылается на релевантный концепт через `## Связи`.

Шаблон SQL practice page — см. [wiki/sql-practice/README.md](../wiki/sql-practice/README.md).

## 2.9 Backlog (особый файл)
Файл `wiki/backlog.md` — единственный своего рода. Это **roadmap источников и тем** для будущего наполнения вики, по зонам и каталогам.

Не является source/concept/pattern/case. Поддерживается параллельно с `log.md`:
- `log.md` — что **уже сделано** (фактическая история ingest'ов).
- `backlog.md` — что **планируется сделать** (с приоритетами и обоснованием).

При закрытии книги/темы — пункт переезжает из `backlog.md` в `log.md` со ссылкой на запись.

---

# 3. Rules for ingest

При добавлении нового источника LLM должен:

1. Создать страницу источника в `wiki/sources/`.
2. Выделить ключевые элементы и для каждого выбрать **тип страницы**:
   - Концепт (как устроено) → `wiki/concepts/`.
   - Паттерн (как решается проблема) → `wiki/patterns/`.
   - Кейс system design (проектирование системы) → `wiki/system-design/`.
   - SQL-задача (упражнение) → `wiki/sql-practice/`.
   - Сравнение → `wiki/comparisons/`.
   - Сущность (инструмент / СУБД) → `wiki/entities/`.
3. Для concept-страницы определить **зону** через зональный маршрутизатор (1.5). Pattern и SQL Practice — горизонтальные каталоги (зона указывается в шапке pattern-страницы; SQL Practice не имеет зоны). System Design всегда зона 1.4.
4. Если источник многозонный (см. 1.6) — материал разносится по соответствующим зонам/каталогам, не складывается в одну.
5. Добавить cross-links между страницами через `## Связи`.
6. Обновить `wiki/index.md`, добавив новые страницы в **правильную секцию**:
   - Concept/Comparison → зональная секция Foundations / DE / Java Backend.
   - Pattern → секция `## Patterns`.
   - System Design case → секция `## System Design`.
   - SQL Practice → секция `## SQL Practice`.
   - Entity → секция `## Entities` с пометкой зоны.
7. Обновить нужный domain-overview (`java-backend-fundamentals.md`, `data-engineering-fundamentals.md`, `system-design-fundamentals.md`, `patterns-catalog.md`, `sql-practice-roadmap.md`), если у каталога появились новые темы.
8. Добавить запись в `wiki/log.md` с указанием затронутых зон/каталогов.
9. Если источник упоминался в `wiki/backlog.md` — пометить его как `[x]` со ссылкой на запись в `log.md`.
10. Строго следовать структуре SCHEMA.md.

---

# 4. Rules for query

При ответе на вопросы:

1. Читать `wiki/index.md`, ориентируясь по зонам.
2. Находить релевантные страницы в нужной зоне; смотреть cross-links для пересечений.
3. Давать структурированный ответ с цитатами и ссылками на конкретные страницы.
4. Предлагать сохранить ответ как новую страницу, если он полезен — с указанием предполагаемой зоны.

---

# 5. Rules for lint

Периодически:

- Искать противоречия между страницами (особенно на стыке зон).
- Находить устаревшие страницы.
- Предлагать новые концепты, в т.ч. недостающие в Java Backend домене.
- Проверять, что concept-страница лежит в **правильной зоне** в `index.md`.
- Проверять, что pattern-страница имеет поле `**Зона:**` в шапке и появляется в секции `## Patterns` индекса (а не в зональной секции).
- Проверять, что system design case лежит в `wiki/system-design/` и в секции `## System Design` индекса; имеет functional/non-functional/capacity/API/data-model/high-level/trade-offs.
- Проверять, что SQL practice задача лежит в `wiki/sql-practice/`, в секции `## SQL Practice` индекса; имеет DDL входных данных, ожидаемый результат, объяснение и план выполнения.
- Проверять, что domain-overviews актуальны: `java-backend-fundamentals`, `data-engineering-fundamentals`, `system-design-fundamentals`, `patterns-catalog`, `sql-practice-roadmap`.
- Улучшать cross-links — особенно между Foundations и обоими доменами; и между patterns ↔ system design cases (кейсы должны ссылаться на использованные паттерны).
- Проверять синхронизацию `wiki/backlog.md` ↔ `wiki/log.md`: каждый закрытый источник в backlog должен иметь запись в log; новые ingest'ы не должны игнорировать backlog (если книга была в плане, статус обновляется).
- Обновлять `index.md`.
