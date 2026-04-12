# Designing Data-Intensive Applications

| Поле | Значение |
|------|----------|
| **Автор** | Martin Kleppmann |
| **Издательство** | O'Reilly Media |
| **Первое издание** | 2017 |
| **Тип** | книга |

## О книге

Практическое руководство по проектированию приложений, для которых **данные — центральная проблема**: надёжность, масштабирование, сопровождение, выбор моделей данных, распределённые системы, потоковая и пакетная обработка. Книга связывает теорию распределённых систем с инженерными компромиссами и реальными системами (БД, очереди, batch/stream-стеки).

## Структура (части)

### Part I. Foundations of Data Systems

1. Надёжные, масштабируемые и сопровождаемые приложения — см. [Reliability, Scalability, Maintainability](../concepts/reliability-scalability-maintainability.md).
2. Модели данных и языки запросов — см. [Data Models and Query Languages](../concepts/data-models-and-query-languages.md).
3. Хранение и извлечение данных — см. [Storage and Retrieval](../concepts/storage-and-retrieval.md).
4. Кодирование и эволюция — см. [Encoding and Schema Evolution](../concepts/encoding-and-schema-evolution.md).

### Part II. Distributed Data

5. Репликация — см. [Replication](../concepts/replication.md).
6. Партиционирование — см. [Partitioning](../concepts/partitioning.md).
7. Транзакции — см. [Transactions and Isolation](../concepts/transactions-and-isolation.md).
8. Проблемы распределённых систем — см. [Distributed Systems Pitfalls](../concepts/distributed-systems-pitfalls.md).
9. Согласованность и консенсус — см. [Consistency and Consensus](../concepts/consistency-and-consensus.md).

### Part III. Derived Data

10. Пакетная обработка — см. [Batch Processing](../concepts/batch-processing.md).
11. Потоковая обработка — см. [Stream Processing](../concepts/stream-processing.md).
12. Будущее систем данных, производные данные — см. [Derived Data and Systems](../concepts/derived-data-and-systems.md).

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| Три столпа надёжных систем | [Reliability, Scalability, Maintainability](../concepts/reliability-scalability-maintainability.md) |
| Модели данных | [Data Models and Query Languages](../concepts/data-models-and-query-languages.md) |
| Индексы, LSM, B-trees | [Storage and Retrieval](../concepts/storage-and-retrieval.md) |
| Форматы, схемы, совместимость | [Encoding and Schema Evolution](../concepts/encoding-and-schema-evolution.md) |
| Репликация | [Replication](../concepts/replication.md) |
| Шардирование / партиции | [Partitioning](../concepts/partitioning.md) |
| Транзакции, изоляция | [Transactions and Isolation](../concepts/transactions-and-isolation.md) |
| Сети, время, сбои | [Distributed Systems Pitfalls](../concepts/distributed-systems-pitfalls.md) |
| Линеаризуемость, консенсус | [Consistency and Consensus](../concepts/consistency-and-consensus.md) |
| MapReduce, batch | [Batch Processing](../concepts/batch-processing.md) |
| Event logs, stream | [Stream Processing](../concepts/stream-processing.md) |
| Производные представления | [Derived Data and Systems](../concepts/derived-data-and-systems.md) |
| OLTP vs OLAP | [OLTP vs OLAP](../comparisons/oltp-vs-olap.md) |

## Сущности (примеры из книги)

- [PostgreSQL](../entities/postgresql.md)
- [Apache Kafka](../entities/apache-kafka.md)

## Связанные обзоры

- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md)
