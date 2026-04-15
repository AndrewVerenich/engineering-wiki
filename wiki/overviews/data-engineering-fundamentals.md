# Data Engineering Fundamentals

Краткий **обзор** опорных идей для data engineering в этой вики. Детали — в концептах и источниках.

## Опорные книги

- [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) — модели данных, хранение, распределение, batch/stream, производные данные.
- [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md) — **dimensional modeling** (Kimball), витрины, SCD, conformed dimensions.
- [PostgreSQL 17 изнутри](../sources/postgresql-internals.md) — внутреннее устройство PostgreSQL: MVCC, буферный кеш, блокировки, индексы, планировщик.
- [SQL: The Complete Reference](../sources/sql-complete-reference.md) — SQL-стандарт: транзакции, блокировки, уровни изоляции.
- [Реляционные базы данных в примерах](../sources/relational-databases-in-examples.md) — нормальные формы, индексы, выполнение запросов.

## Ключевые концепты (DDIA)

- [Reliability, Scalability, Maintainability](../concepts/reliability-scalability-maintainability.md)
- [Replication](../concepts/replication.md), [Partitioning](../concepts/partitioning.md)
- [Batch Processing](../concepts/batch-processing.md), [Stream Processing](../concepts/stream-processing.md)
- [OLTP vs OLAP](../comparisons/oltp-vs-olap.md)

## Dimensional modeling (Kimball)

- Обзор: [Dimensional Modeling (Kimball)](../concepts/dimensional-modeling-kimball.md)
- Два слоя (staging ↔ витрина): [Staging and Presentation Layers](../concepts/staging-and-presentation-layers.md)
- Собеседование: [Dimensional Modeling Interview Prep](dimensional-modeling-interview-prep.md)

## ClickHouse (официальная документация + практика)

- Источник: [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)
- Learning track: [ClickHouse Learning Track (dbt-oriented)](clickhouse-learning-track.md)
- Schema: [ClickHouse Schema Design Patterns](../concepts/clickhouse-schema-design-patterns.md)
- Ingestion: [ClickHouse Insert and Mutation Strategy](../concepts/clickhouse-insert-and-mutation-strategy.md)
- Query/MV: [ClickHouse Query Optimization Patterns](../concepts/clickhouse-query-optimization-patterns.md)
- Index/Merge internals: [ClickHouse: индексы и background merges](../concepts/clickhouse-indexes-and-background-merges.md)
- Engine internals: [ClickHouse под капотом](../concepts/clickhouse-under-the-hood.md)
- MergeTree family: [ClickHouse MergeTree Engines](../concepts/clickhouse-merge-tree-engines.md)
- Kafka ingestion: [ClickHouse + Kafka Ingestion](../concepts/clickhouse-kafka-ingestion.md)

## dbt (learning + применение)

- Roadmap: [dbt Learning Roadmap](dbt-learning-roadmap.md)
- Применение: [dbt in Portfolio: Applied Patterns](../concepts/dbt-portfolio-applied-patterns.md)
- База слоев: [dbt: слои проекта](../concepts/dbt-project-layers.md)

## PostgreSQL internals

- Источник: [PostgreSQL 17 изнутри](../sources/postgresql-internals.md)
- MVCC, snapshots, VACUUM: [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md)
- Сравнение с MySQL: [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md)
- Транзакции и блокировки (SQL-стандарт): [SQL Transactions, Locking and Isolation](../concepts/sql-transactions-locking-standard.md)
- Нормализация: [Normalization and Normal Forms](../concepts/normalization-and-normal-forms.md)
- Выполнение запросов и индексы: [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md)

## Сущности

- [PostgreSQL](../entities/postgresql.md), [Apache Kafka](../entities/apache-kafka.md)
