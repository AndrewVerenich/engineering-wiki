# Partitioning

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 6)

## Суть

**Партиционирование (шардирование)** распределяет данные по узлам по ключу. Важны: выбор ключа, **rebalancing** при добавлении узлов, запросы, затрагивающие несколько партиций (**scatter/gather**), и вторичные индексы (локальные vs глобальные).

## Связи

- [Replication](replication.md) — каждая партиция обычно реплицируется.
- [Transactions and Isolation](transactions-and-isolation.md) — транзакции между партициями дороже.
- [Apache Kafka](../entities/apache-kafka.md) — партиции топиков.
