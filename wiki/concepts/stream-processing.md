# Stream Processing

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 11)

## Суть

**Потоковая обработка** — непрерывная обработка событий с низкой задержкой. Темы: **event streams**, окна по времени, **join** потоков, семантика **exactly-once** (через идемпотентность, дедупликацию, транзакционные sink’и), сравнение с batch как обработка бесконечного набора vs ограниченного.

## Связи

- [Encoding and Schema Evolution](encoding-and-schema-evolution.md) — форматы в событиях.
- [Apache Kafka](../entities/apache-kafka.md) — лог как транспорт потоков.
- [Derived Data and Systems](derived-data-and-systems.md) — unifying batch и stream.
