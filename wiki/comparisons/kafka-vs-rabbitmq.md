# Kafka vs RabbitMQ

**Контекст:** обе системы передают сообщения между сервисами, но их базовая модель разная: Kafka — distributed log platform, RabbitMQ — queue-oriented broker.

## Главная разница

| Критерий | Kafka | RabbitMQ |
|----------|-------|----------|
| Базовая модель | append-only log | очереди и маршрутизация сообщений |
| Повторное чтение | естественный replay по offsets | обычно не replay-centric модель |
| Порядок | гарантирован внутри partition | гарантии зависят от queue/consumer setup |
| Масштабирование | через partitions и consumer groups | через queues/exchanges + clustering patterns |
| Лучший fit | event streaming, analytics, durable event history | task queues, routing-heavy messaging |

## Trade-offs по эксплуатации

| Тема | Kafka | RabbitMQ |
|------|-------|----------|
| Throughput-heavy workloads | сильная сторона | зависит от сценария, часто ниже |
| Сложность topic design | выше (keys, partitions, retention) | ниже для простых queue-сценариев |
| Routing flexibility | базовая | сильная сторона (exchanges/bindings) |
| Историчность данных | built-in через retention | обычно ограниченная, не core use-case |

## Типичные вопросы на интервью

**Q: Можно ли сказать, что Kafka «лучше» RabbitMQ?**  
A: Нет, это разные инструменты под разные workload. Kafka лучше как event log backbone и replay-платформа. RabbitMQ лучше для routing-heavy task messaging, где не требуется долгий retention событий.

**Q: Когда RabbitMQ может быть проще для backend?**  
A: Когда нужен классический queue semantics с TTL/DLQ/routing и не нужна долгоживущая история событий. Для таких кейсов Kafka может дать лишнюю операционную сложность.

**Q: Почему Kafka обычно выбирают в DE и event-driven аналитике?**  
A: Из-за partitioned log модели, replay-окна и хорошей интеграции со stream/data-инструментами. Это делает Kafka удобной шиной между ingestion, processing и serving слоями.

## Связи

- [Apache Kafka](../entities/apache-kafka.md)
- [Kafka Topic Design and Compaction](../concepts/kafka-topic-design-and-compaction.md)
- [Kafka In DE Pipelines](../concepts/kafka-in-de-pipelines.md)
- [Kafka Backend Patterns: Outbox, DLQ, Retry](../concepts/kafka-backend-patterns-outbox-dlq-retry.md)
