# Index

Wiki разбита на **три зоны**:

- **Foundations** — нужно и Java backend-инженеру, и data engineer'у: распределённые системы, транзакции, SQL, реляционное моделирование.
- **Data Engineering** — DWH, batch/stream, моделирование витрин, инструменты данных.
- **Java Backend** — JVM, Spring, Kotlin, concurrency, web/API (раздел в стадии наполнения).

Дублирование ссылок не делаем: каждая страница лежит в одной зоне; cross-links живут внутри страниц в секции «Связи».

## Domains (overviews)

- [Java Backend Fundamentals](overviews/java-backend-fundamentals.md)
- [Data Engineering Fundamentals](overviews/data-engineering-fundamentals.md)

## Sources

| Источник | Зона |
|----------|------|
| [Designing Data-Intensive Applications](sources/designing-data-intensive-applications.md) | Foundations + DE |
| [PostgreSQL 17 изнутри](sources/postgresql-internals.md) | Foundations |
| [SQL: The Complete Reference](sources/sql-complete-reference.md) | Foundations |
| [Реляционные базы данных в примерах](sources/relational-databases-in-examples.md) | Foundations |
| [The Data Warehouse Toolkit](sources/the-data-warehouse-toolkit.md) | DE |
| [ClickHouse Official Documentation](sources/clickhouse-official-documentation.md) | DE |
| [Stream Processing with Apache Flink](sources/stream-processing-with-apache-flink.md) | DE |

## Foundations (shared)

### Distributed Systems & System Design

- [Reliability, Scalability, Maintainability](concepts/reliability-scalability-maintainability.md)
- [Replication](concepts/replication.md)
- [Partitioning](concepts/partitioning.md)
- [Distributed Systems Pitfalls](concepts/distributed-systems-pitfalls.md)
- [Consistency and Consensus](concepts/consistency-and-consensus.md)
- [Encoding and Schema Evolution](concepts/encoding-and-schema-evolution.md)

### Transactions, Isolation & Storage Internals

- [Transactions and Isolation](concepts/transactions-and-isolation.md)
- [SQL Transactions, Locking and Isolation](concepts/sql-transactions-locking-standard.md)
- [PostgreSQL MVCC Internals](concepts/postgresql-mvcc-internals.md)
- [PostgreSQL vs MySQL: MVCC](comparisons/postgresql-vs-mysql-mvcc.md)

### SQL & Relational Modeling

- [Data Models and Query Languages](concepts/data-models-and-query-languages.md)
- [Normalization and Normal Forms](concepts/normalization-and-normal-forms.md)
- [SQL Query Execution and Indexes](concepts/sql-query-execution-and-indexes.md)

## Data Engineering

### DDIA — главы по обработке данных

- [Storage and Retrieval](concepts/storage-and-retrieval.md)
- [Batch Processing](concepts/batch-processing.md)
- [Stream Processing](concepts/stream-processing.md)
- [Derived Data and Systems](concepts/derived-data-and-systems.md)
- [OLTP vs OLAP](comparisons/oltp-vs-olap.md)

### Dimensional modeling (Kimball)

- [Dimensional Modeling (Kimball)](concepts/dimensional-modeling-kimball.md)
- [Staging and Presentation Layers](concepts/staging-and-presentation-layers.md)
- [Star Schema and Grain](concepts/star-schema-and-grain.md)
- [Dimension Tables](concepts/dimension-tables.md)
- [Fact Table Types (Kimball)](concepts/fact-table-types-kimball.md)
- [Slowly Changing Dimensions](concepts/slowly-changing-dimensions.md)
- [Conformed Dimensions](concepts/conformed-dimensions.md)
- [Advanced Dimensional Patterns](concepts/advanced-dimensional-patterns.md)
- [Kimball vs Inmon](comparisons/kimball-vs-inmon.md)

### ClickHouse

- [ClickHouse Schema Design Patterns](concepts/clickhouse-schema-design-patterns.md)
- [ClickHouse Insert and Mutation Strategy](concepts/clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](concepts/clickhouse-query-optimization-patterns.md)
- [ClickHouse: индексы и background merges](concepts/clickhouse-indexes-and-background-merges.md)
- [ClickHouse под капотом](concepts/clickhouse-under-the-hood.md)
- [ClickHouse MergeTree Engines](concepts/clickhouse-merge-tree-engines.md)
- [ClickHouse + Kafka Ingestion](concepts/clickhouse-kafka-ingestion.md)
- [CDC: Debezium → Kafka → ClickHouse](concepts/cdc-debezium-analytics-pipeline.md)

### dbt

- [dbt: слои проекта](concepts/dbt-project-layers.md)
- [dbt in Portfolio: Applied Patterns](concepts/dbt-portfolio-applied-patterns.md)
- [ELT vs ETL (на практике)](comparisons/elt-vs-etl-practice.md)

### Spark

- [Spark Batch + Medallion (HDFS)](concepts/spark-batch-medallion-hdfs.md)

### Stream Processing (Flink)

- [Apache Flink Architecture](concepts/flink-architecture.md)
- [Flink DataStream API](concepts/flink-datastream-api.md)
- [Flink Stateless vs Stateful Operators](concepts/flink-stateless-vs-stateful-operators.md)
- [Flink Time and Watermarks](concepts/flink-time-and-watermarks.md)
- [Flink Windows](concepts/flink-windows.md)
- [Flink State Management](concepts/flink-state-management.md)
- [Flink Checkpoints and Savepoints](concepts/flink-checkpoints-and-savepoints.md)
- [Flink Exactly-Once Semantics](concepts/flink-exactly-once-semantics.md)
- [Flink ProcessFunction](concepts/flink-process-function.md)
- [Flink vs Spark Structured Streaming](comparisons/flink-vs-spark-structured-streaming.md)

## Java Backend

> Раздел в стадии наполнения. Структура задана в [SCHEMA.md](../schema/SCHEMA.md); состав тем — в [Java Backend Fundamentals](overviews/java-backend-fundamentals.md). Концепты добавляются по мере чтения источников.

Темы (планируемые):

- **JVM internals** — память (heap/metaspace), GC, JIT, class loading.
- **Concurrency & memory model** — JMM, `java.util.concurrent`, virtual threads, structured concurrency.
- **Spring** — IoC/DI, AOP, Boot, MVC/WebFlux, Data, Security, Cloud.
- **Kotlin** — null-safety, coroutines, идиомы для backend.
- **Web & APIs** — REST/HTTP semantics, gRPC, OpenAPI, идемпотентность, retries.
- **Observability** — metrics, tracing, logging (Micrometer, OpenTelemetry).

## Entities

- [PostgreSQL](entities/postgresql.md) — Foundations / DE
- [Apache Kafka](entities/apache-kafka.md) — Foundations / DE
- [Apache Flink](entities/apache-flink.md) — DE

## Overviews

- [Java Backend Fundamentals](overviews/java-backend-fundamentals.md)
- [Data Engineering Fundamentals](overviews/data-engineering-fundamentals.md)
- [Dimensional Modeling Interview Prep](overviews/dimensional-modeling-interview-prep.md)
- [dbt Learning Roadmap](overviews/dbt-learning-roadmap.md)
- [ClickHouse Learning Track (dbt-oriented)](overviews/clickhouse-learning-track.md)
