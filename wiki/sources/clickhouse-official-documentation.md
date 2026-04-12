# ClickHouse Official Documentation

| Поле | Значение |
|------|----------|
| **Тип** | официальная документация |
| **URL** | https://clickhouse.com/docs |
| **Фокус ingest** | best practices для DWH/OLAP и ingestion |

## Что изучено

- Официальные best practices по проектированию таблиц MergeTree.
- Паттерны ingest (batch, async insert, форматы, мутации).
- Оптимизация запросов (ORDER BY, skipping indices, JOIN-алгоритмы, MV).
- Практика применения для аналитического DWH-контекста.

## Ключевые страницы вики

| Тема | Страница |
|------|----------|
| Schema design | [ClickHouse Schema Design Patterns](../concepts/clickhouse-schema-design-patterns.md) |
| Ingestion и мутации | [ClickHouse Insert and Mutation Strategy](../concepts/clickhouse-insert-and-mutation-strategy.md) |
| Query performance и MV | [ClickHouse Query Optimization Patterns](../concepts/clickhouse-query-optimization-patterns.md) |
| Индексы и фоновые merge | [ClickHouse: индексы и background merges](../concepts/clickhouse-indexes-and-background-merges.md) |
| Как ClickHouse работает внутри | [ClickHouse под капотом](../concepts/clickhouse-under-the-hood.md) |
| Семейство MergeTree движков | [ClickHouse MergeTree Engines](../concepts/clickhouse-merge-tree-engines.md) |
| Kafka ingestion | [ClickHouse + Kafka Ingestion](../concepts/clickhouse-kafka-ingestion.md) |

## Базовые связи

- [Storage and Retrieval](../concepts/storage-and-retrieval.md)
- [Partitioning](../concepts/partitioning.md)
- [ELT vs ETL (на практике)](../comparisons/elt-vs-etl-practice.md)
- [ClickHouse Learning Track (dbt-oriented)](../overviews/clickhouse-learning-track.md)
