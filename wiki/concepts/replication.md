# Replication

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 5)

## Суть

**Репликация** — копии данных на нескольких узлах: для отказоустойчивости и масштабирования чтения. Модели обновления: **single-leader**, **multi-leader**, **leaderless**. Ключевые темы: синхронная vs асинхронная репликация, **replication lag**, чтение собственных записей, монотонное чтение, consistent prefix. Конфликты при multi-leader требуют явной стратегии.

## Связи

- [Partitioning](partitioning.md) — репликация и партиции часто комбинируются.
- [Consistency and Consensus](consistency-and-consensus.md) — согласованность реплик.
- [Apache Kafka](../entities/apache-kafka.md) — репликация логов партиций.
