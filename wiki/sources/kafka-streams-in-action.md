# Kafka Streams in Action, Second Edition

| Поле | Значение |
|------|----------|
| **Автор** | Bill Bejeck |
| **Издательство** | Manning Publications |
| **Год/Издание** | 2024, Second Edition |
| **Тип** | книга |

## О книге

Практическое инженерное руководство по Kafka Streams уровня production: как строить topology, работать с state stores, join/window/time semantics, обеспечивать exactly-once и наблюдаемость в реальных JVM-сервисах. Книга полезна тем, что объясняет не только API DSL, но и поведение под нагрузкой и при отказах.

## Структура

1. Introduction to Kafka Streams и программная модель.
2. Topology, tasks, threads, deployment model. См. [Kafka Streams Architecture](../concepts/kafka-streams-architecture.md).
3. DSL: `KStream`, `KTable`, `GlobalKTable`. См. [Kafka Streams: KStream, KTable, GlobalKTable](../concepts/kafka-streams-kstream-and-ktable.md).
4. Stateful processing и local stores. См. [Kafka Streams State Stores and Changelog](../concepts/kafka-streams-state-stores-and-changelog.md).
5. Joins (stream-stream, stream-table, table-table). См. [Kafka Streams Joins](../concepts/kafka-streams-joins.md).
6. Windowing и late events. См. [Kafka Streams Windowing](../concepts/kafka-streams-windowing.md).
7. Time semantics и stream time. См. [Kafka Streams Time and Stream Time](../concepts/kafka-streams-time-and-streamtime.md).
8. Exactly-once в Kafka Streams. См. [Kafka Streams Exactly-Once](../concepts/kafka-streams-exactly-once.md).
9. Processor API и punctuations. См. [Kafka Streams Processor API](../concepts/kafka-streams-processor-api.md).
10. Testing topology и production operations. См. [Kafka Streams Testing with TopologyTestDriver](../concepts/kafka-streams-testing-topology-test-driver.md), [Kafka Streams Interactive Queries and Deployment](../concepts/kafka-streams-interactive-queries-and-deployment.md).

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| Архитектура topology/tasks/threads | [Kafka Streams Architecture](../concepts/kafka-streams-architecture.md) |
| DSL: KStream/KTable/GlobalKTable | [Kafka Streams: KStream, KTable, GlobalKTable](../concepts/kafka-streams-kstream-and-ktable.md) |
| Stateful processing и changelog | [Kafka Streams State Stores and Changelog](../concepts/kafka-streams-state-stores-and-changelog.md) |
| Join-паттерны и co-partitioning | [Kafka Streams Joins](../concepts/kafka-streams-joins.md) |
| Окна, grace и suppress | [Kafka Streams Windowing](../concepts/kafka-streams-windowing.md) |
| Time semantics и stream time | [Kafka Streams Time and Stream Time](../concepts/kafka-streams-time-and-streamtime.md) |
| Exactly-once v2 | [Kafka Streams Exactly-Once](../concepts/kafka-streams-exactly-once.md) |
| Processor API | [Kafka Streams Processor API](../concepts/kafka-streams-processor-api.md) |
| Тестирование topology | [Kafka Streams Testing with TopologyTestDriver](../concepts/kafka-streams-testing-topology-test-driver.md) |
| Interactive Queries и operations | [Kafka Streams Interactive Queries and Deployment](../concepts/kafka-streams-interactive-queries-and-deployment.md) |
| Kafka Streams vs Flink | [Kafka Streams vs Flink](../comparisons/kafka-streams-vs-flink.md) |
| Kafka Streams vs Kafka Consumer API | [Kafka Streams vs Kafka Consumer](../comparisons/kafka-streams-vs-kafka-consumer.md) |

## Связи

- [Apache Kafka](../entities/apache-kafka.md) — Kafka Streams работает поверх core-механик Kafka.
- [Kafka Streams](../entities/kafka-streams.md) — сущность технологии в контексте вики.
- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md) — место Kafka Streams в DE-треке.
