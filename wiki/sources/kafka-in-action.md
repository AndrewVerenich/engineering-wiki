# Kafka in Action

| Поле | Значение |
|------|----------|
| **Авторы** | Dylan Scott, Viktor Gamov, Dave Klein |
| **Издательство** | Manning Publications |
| **Год/Издание** | 2022 (1st edition) |
| **Тип** | книга |

## О книге

Практическая книга про Apache Kafka: от устройства кластера и storage internals до producer/consumer API, topic design, observability и интеграций в data pipelines. Основная ценность для вики — связка **как Kafka работает под капотом** и **как её правильно применять** в Java backend и data engineering.

## Структура

1. Introduction to Kafka — базовая модель event log и экосистема.
2. Getting to know Kafka — брокеры, топики, партиции, клиентская модель.
3. Designing a Kafka project — ключи, партиционирование, репликация, SLA.
4. Producers — batching, reliability, delivery guarantees.
5. Consumers — consumer groups, offsets, rebalance.
6. Brokers — replication, ISR, controller mechanics.
7. Topics and partitions — retention, compaction, scaling.
8. Storage internals — log segments, index files, disk/page cache.
9. Management and tools — operations, мониторинг, диагностика.
10. Protecting Kafka — security overview.
11. Schema Registry and evolution.
12. Stream processing ecosystem (Kafka Streams/ksqlDB).
13. Kafka in broader data ecosystem.

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| Storage internals | [Kafka Storage and Log Internals](../concepts/kafka-storage-and-log-internals.md) |
| Replication and ISR | [Kafka Replication and ISR](../concepts/kafka-replication-and-isr.md) |
| Controller and KRaft | [Kafka Controller and KRaft](../concepts/kafka-controller-and-kraft.md) |
| Delivery semantics and transactions | [Kafka Delivery Semantics and Transactions](../concepts/kafka-delivery-semantics-and-transactions.md) |
| Topic design and compaction | [Kafka Topic Design and Compaction](../concepts/kafka-topic-design-and-compaction.md) |
| Schema Registry | [Kafka Schema Registry and Evolution](../concepts/kafka-schema-registry-and-evolution.md) |
| Producer internals | [Kafka Producer Internals and Tuning](../concepts/kafka-producer-internals-and-tuning.md) |
| Consumer internals | [Kafka Consumer Internals and Offsets](../concepts/kafka-consumer-internals-and-offsets.md) |
| Backend delivery patterns | [Kafka Backend Patterns: Outbox, DLQ, Retry](../concepts/kafka-backend-patterns-outbox-dlq-retry.md) |
| Kafka observability | [Kafka Observability and Production Gotchas](../concepts/kafka-observability-and-production-gotchas.md) |
| Kafka in DE pipelines | [Kafka in DE Pipelines](../concepts/kafka-in-de-pipelines.md) |
| Kafka Connect | [Kafka Connect Fundamentals](../concepts/kafka-connect-fundamentals.md) |
| Kafka vs RabbitMQ | [Kafka vs RabbitMQ](../comparisons/kafka-vs-rabbitmq.md) |

## Связи

- [Apache Kafka](../entities/apache-kafka.md)
- [Java Backend Fundamentals](../overviews/java-backend-fundamentals.md)
- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md)
- [Stream Processing](../concepts/stream-processing.md)
