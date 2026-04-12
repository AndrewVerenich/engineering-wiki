# Data Engineering Fundamentals

Краткий **обзор** опорных идей для data engineering в этой вики. Детали — в концептах и источниках.

## Опорные книги

- [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) — модели данных, хранение, распределение, batch/stream, производные данные.
- [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md) — **dimensional modeling** (Kimball), витрины, SCD, conformed dimensions.

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
- План обучения: [Личный план изучения ClickHouse (через dbt)](../sources/clickhouse-learning-plan.md)
- Learning track: [ClickHouse Learning Track (dbt-oriented)](clickhouse-learning-track.md)
- Schema: [ClickHouse Schema Design Patterns](../concepts/clickhouse-schema-design-patterns.md)
- Ingestion: [ClickHouse Insert and Mutation Strategy](../concepts/clickhouse-insert-and-mutation-strategy.md)
- Query/MV: [ClickHouse Query Optimization Patterns](../concepts/clickhouse-query-optimization-patterns.md)
- Index/Merge internals: [ClickHouse: индексы и background merges](../concepts/clickhouse-indexes-and-background-merges.md)
- Engine internals: [ClickHouse под капотом](../concepts/clickhouse-under-the-hood.md)
- MergeTree family: [ClickHouse MergeTree Engines](../concepts/clickhouse-merge-tree-engines.md)
- Kafka ingestion: [ClickHouse + Kafka Ingestion](../concepts/clickhouse-kafka-ingestion.md)

## dbt (learning + применение)

- План: [Личный план изучения dbt](../sources/dbt-learning-plan.md)
- Roadmap: [dbt Learning Roadmap](dbt-learning-roadmap.md)
- Применение: [dbt in Portfolio: Applied Patterns](../concepts/dbt-portfolio-applied-patterns.md)
- База слоев: [dbt: слои проекта](../concepts/dbt-project-layers.md)

## Сущности

- [PostgreSQL](../entities/postgresql.md), [Apache Kafka](../entities/apache-kafka.md)
