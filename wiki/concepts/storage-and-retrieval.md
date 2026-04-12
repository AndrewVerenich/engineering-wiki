# Storage and Retrieval

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 3)

## Суть

Данные на диске организуются через **структуры хранения** и **индексы**. Типичный контраст: **LSM-tree** (порционные слияния, хорош для write-heavy) vs **B-tree** (часто в БД, сбалансированное чтение/запись). Индексы ускоряют чтение ценой места и стоимости записи. Понимание движка помогает объяснять latency и план запросов.

## Связи

- [Partitioning](partitioning.md) — индексы и партиции вместе задают производительность.
- [Encoding and Schema Evolution](encoding-and-schema-evolution.md) — сериализация значений.
- [PostgreSQL](../entities/postgresql.md) — B-tree и другие индексы в практике.
