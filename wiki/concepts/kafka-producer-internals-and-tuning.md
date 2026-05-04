# Kafka Producer Internals and Tuning

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Производительность producer-а в Kafka определяется не одной настройкой, а балансом batching, compression, reliability и in-flight concurrency. Неправильные параметры легко дают либо лишнюю latency, либо дубликаты/потери в сбоях.

## Внутренний путь записи

| Компонент | Роль |
|-----------|------|
| RecordAccumulator | буферизация records по partition |
| Sender thread | отправка batches на brokers |
| Partitioner | выбор partition для record |
| Retry logic | повтор отправки при временных ошибках |

## Ключевые настройки

| Параметр | Что дает | Риск |
|----------|----------|------|
| `linger.ms` | больше batching -> лучше throughput | выше latency |
| `batch.size` | крупнее батчи -> меньше overhead | память/задержка fill |
| `compression.type` | меньше network and storage cost | дополнительный CPU |
| `acks` | durability уровень | latency vs reliability |
| `retries` | устойчивость к transient failures | дубликаты без idempotence |
| `max.in.flight.requests.per.connection` | конкарренси в отправке | reordering risk без идемпотентности |

## Idempotence и transactions

- `enable.idempotence=true` нужен почти всегда для production write path.
- Для consume-process-produce workflows с атомарностью нескольких записей применяются transactions (`transactional.id`).

## Типичные вопросы на интервью

**Q: Зачем повышать `linger.ms`, если нужна низкая latency?**  
A: Это trade-off. Малый `linger.ms` минимизирует задержку одиночного сообщения, но режет batching efficiency. Для high-throughput сервисов умеренный `linger.ms` часто снижает общую стоимость и стабилизирует tail latency.

**Q: Почему retries без idempotence опасны?**  
A: При сетевых таймаутах producer может повторно отправить batch, который broker уже записал. Без idempotence это создает duplicates, особенно в event-sourcing и billing-сценариях.

**Q: Как partitioner влияет на систему?**  
A: Он определяет распределение нагрузки и ordering guarantees. Плохой partition key вызывает hot partition, снижает throughput и ухудшает rebalance behavior у consumers.

## Связи

- [Kafka Delivery Semantics and Transactions](kafka-delivery-semantics-and-transactions.md)
- [Kafka Topic Design and Compaction](kafka-topic-design-and-compaction.md)
- [Kafka Consumer Internals and Offsets](kafka-consumer-internals-and-offsets.md)
- [Apache Kafka](../entities/apache-kafka.md)
