# Data Engineering Fundamentals

Краткий **обзор** DE-контента в этой вики. Раздел построен поверх общих [Foundations](#связанные-foundations-общие-с-backend), которые шарятся с Java backend (распределённые системы, транзакции, SQL, моделирование).

## Опорные книги

- [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) — модели данных, хранение, распределение, batch/stream, производные данные.
- [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md) — **dimensional modeling** (Kimball): витрины, SCD, conformed dimensions.
- [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md) — внутреннее устройство и best practices ClickHouse.
- [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) — stateful stream processing, event time, windows, watermarks, fault tolerance.

> Книги по СУБД и SQL ([PostgreSQL 17 изнутри](../sources/postgresql-internals.md), [SQL: The Complete Reference](../sources/sql-complete-reference.md), [Реляционные базы данных в примерах](../sources/relational-databases-in-examples.md)) перенесены в Foundations — см. ниже.

## Темы DE

### DDIA — главы по обработке данных

- [Storage and Retrieval](../concepts/storage-and-retrieval.md)
- [Batch Processing](../concepts/batch-processing.md)
- [Stream Processing](../concepts/stream-processing.md)
- [Derived Data and Systems](../concepts/derived-data-and-systems.md)
- [OLTP vs OLAP](../comparisons/oltp-vs-olap.md)

### Dimensional modeling (Kimball)

- [Dimensional Modeling (Kimball)](../concepts/dimensional-modeling-kimball.md)
- [Staging and Presentation Layers](../concepts/staging-and-presentation-layers.md)
- [Star Schema and Grain](../concepts/star-schema-and-grain.md)
- [Dimension Tables](../concepts/dimension-tables.md)
- [Fact Table Types (Kimball)](../concepts/fact-table-types-kimball.md)
- [Slowly Changing Dimensions](../concepts/slowly-changing-dimensions.md)
- [Conformed Dimensions](../concepts/conformed-dimensions.md)
- [Advanced Dimensional Patterns](../concepts/advanced-dimensional-patterns.md)
- [Kimball vs Inmon](../comparisons/kimball-vs-inmon.md)
- Interview prep: [Dimensional Modeling Interview Prep](dimensional-modeling-interview-prep.md)

### ClickHouse

- [ClickHouse Schema Design Patterns](../concepts/clickhouse-schema-design-patterns.md)
- [ClickHouse Insert and Mutation Strategy](../concepts/clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](../concepts/clickhouse-query-optimization-patterns.md)
- [ClickHouse: индексы и background merges](../concepts/clickhouse-indexes-and-background-merges.md)
- [ClickHouse под капотом](../concepts/clickhouse-under-the-hood.md)
- [ClickHouse MergeTree Engines](../concepts/clickhouse-merge-tree-engines.md)
- [ClickHouse + Kafka Ingestion](../concepts/clickhouse-kafka-ingestion.md)
- [CDC: Debezium → Kafka → ClickHouse](../concepts/cdc-debezium-analytics-pipeline.md)
- Learning track: [ClickHouse Learning Track (dbt-oriented)](clickhouse-learning-track.md)

### dbt

- [dbt: слои проекта](../concepts/dbt-project-layers.md)
- [dbt in Portfolio: Applied Patterns](../concepts/dbt-portfolio-applied-patterns.md)
- [ELT vs ETL (на практике)](../comparisons/elt-vs-etl-practice.md)
- Learning roadmap: [dbt Learning Roadmap](dbt-learning-roadmap.md)

### Spark

- [Spark Batch + Medallion (HDFS)](../concepts/spark-batch-medallion-hdfs.md)

### Stream Processing (Flink)

- [Apache Flink Architecture](../concepts/flink-architecture.md)
- [Flink DataStream API](../concepts/flink-datastream-api.md)
- [Flink Stateless vs Stateful Operators](../concepts/flink-stateless-vs-stateful-operators.md)
- [Flink Time and Watermarks](../concepts/flink-time-and-watermarks.md)
- [Flink Windows](../concepts/flink-windows.md)
- [Flink State Management](../concepts/flink-state-management.md)
- [Flink Checkpoints and Savepoints](../concepts/flink-checkpoints-and-savepoints.md)
- [Flink Exactly-Once Semantics](../concepts/flink-exactly-once-semantics.md)
- [Flink ProcessFunction](../concepts/flink-process-function.md)
- [Flink vs Spark Structured Streaming](../comparisons/flink-vs-spark-structured-streaming.md)

## Связанные foundations (общие с backend)

Эти страницы лежат в зоне Foundations и одинаково важны для backend и DE — они нужны, чтобы понимать DE-контент глубоко.

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

- [PostgreSQL](../entities/postgresql.md), [Apache Kafka](../entities/apache-kafka.md), [Apache Flink](../entities/apache-flink.md), [Redis](../entities/redis.md)

## Связи

- [Java Backend Fundamentals](java-backend-fundamentals.md) — параллельный домен; foundations общие.
- [SCHEMA.md](../../schema/SCHEMA.md) — рубрики и правила вики.
