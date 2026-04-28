# LLM Wiki Schema

Эта вики — персональная база знаний инженера, ориентированная на два рабочих домена: **Java Backend** и **Data Engineering**. Часть материала (распределённые системы, транзакции, SQL, моделирование данных) одинаково нужна обоим доменам — он живёт в общей зоне **Foundations**.

LLM полностью отвечает за создание, обновление и поддержание всех страниц.
Задача пользователя — добавлять источники (книги, статьи, курсы) и задавать вопросы.
Задача LLM — поддерживать структуру, связи, целостность и **зональное размещение** материала.

---

# 1. Wiki Zones (главное)

Wiki разбита на **три параллельные зоны**. Каждая страница принадлежит **ровно одной зоне** — дублирование ссылок в `index.md` не делается. Пересечения между доменами выражаются через секцию `## Связи` внутри страниц.

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

## 1.4 Зональный маршрутизатор (как принимать решение)

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
| Apache Kafka (entity), сравнения брокеров | Foundations + DE (entity, не concept — лежит в `entities/`, отображается в обоих доменах) |

**Если страница попадает в две зоны** — выбирается та, где она наиболее «корневая» (где её используют для понимания базы), а в другой зоне ставится cross-link через `## Связи`. Пример: `encoding-and-schema-evolution.md` — в Foundations (нужно и для distributed systems, и для serialization в backend, и для DE), а Avro/Protobuf-специфика для DE упоминается через ссылки.

## 1.5 Multi-zone ingest

**Одна книга может породить страницы в разных зонах.** Это ожидаемо. Пример:

- *Designing Data-Intensive Applications* → Foundations (replication, partitioning, transactions, consistency, encoding) + Data Engineering (storage and retrieval, batch, stream, derived data).
- *PostgreSQL 17 изнутри* → Foundations (MVCC, locking, query execution).
- *Stream Processing with Apache Flink* → Data Engineering целиком.
- (будущее) *Java Concurrency in Practice* → Java Backend / Concurrency, частично Foundations (memory model — пограничная).

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
**Зона:** ровно одна зона. Решение — через зональный маршрутизатор (1.4).

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

---

# 3. Rules for ingest

При добавлении нового источника LLM должен:

1. Создать страницу источника в `wiki/sources/`.
2. Выделить ключевые концепты и для каждого:
   - Определить **зону** через зональный маршрутизатор (1.4).
   - Создать или обновить concept/entity страницу в `wiki/concepts/` или `wiki/entities/`.
3. Если источник многозонный (см. 1.5) — концепты разносятся по соответствующим зонам, не складываются в одну.
4. Добавить cross-links между страницами через `## Связи`.
5. Обновить `wiki/index.md`, добавив новые страницы в **правильную зону и подсекцию**.
6. Обновить нужный domain-overview (`java-backend-fundamentals.md` или `data-engineering-fundamentals.md`), если у domain'а появились новые темы.
7. Добавить запись в `wiki/log.md` с указанием затронутых зон.
8. Строго следовать структуре SCHEMA.md.

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
- Проверять, что domain-overviews актуальны.
- Улучшать cross-links — особенно между Foundations и обоими доменами.
- Обновлять `index.md`.
