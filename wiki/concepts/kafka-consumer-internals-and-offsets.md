# Kafka Consumer Internals and Offsets

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Consumer correctness строится вокруг offset lifecycle: когда читать, когда коммитить, как переживать rebalance и что делать при lag. Ошибки здесь чаще всего приводят либо к потерям (commit слишком рано), либо к дубликатам (commit слишком поздно без idempotency).

## Внутренняя модель consumer

| Элемент | Роль |
|---------|------|
| `poll()` loop | fetch + heartbeat + delivery records приложению |
| Consumer group coordinator | назначение partition и rebalance |
| Offset commit | фиксация прогресса чтения |
| Rebalance protocol | eager/cooperative перераспределение partition |

## Ключевые параметры

| Параметр | Что влияет | Типичный риск |
|----------|------------|---------------|
| `max.poll.interval.ms` | max время обработки между poll | consumer kicked out из group |
| `session.timeout.ms` | детект падения consumer | слишком мало -> фальшивые rebalance |
| `max.poll.records` | размер пачки на обработку | слишком много -> long processing pauses |
| `enable.auto.commit` | автокоммит offsets | неконтролируемые потери при slow processing |

## Commit стратегии

| Стратегия | Плюсы | Минусы |
|-----------|-------|--------|
| Auto commit | просто стартовать | риск потерь/дубликатов в edge cases |
| Manual sync commit | предсказуемость | latency и блокировки на commit path |
| Manual async commit | throughput | сложнее error handling |

## Offset reset

- `earliest` — читать с начала доступного retention.
- `latest` — только новые сообщения.
- `none` — fail fast, если offset отсутствует.

## Типичные вопросы на интервью

**Q: Почему auto commit часто не подходит для критичных pipeline?**  
A: Потому что commit может произойти до успешной бизнес-обработки. При сбое между commit и фактической обработкой потеряешь сообщения без шанса replay.

**Q: Что такое consumer lag и почему это не только «медленный consumer»?**  
A: Lag — разница между latest offset и committed/processed offset. Причины: slow processing, hot partition, rebalance storms, проблемы брокера, downstream bottleneck. Лечится не только масштабированием consumers.

**Q: Зачем cooperative rebalance?**  
A: Он уменьшает stop-the-world эффект при перераспределении partition: consumers не обязаны одновременно отпускать все assignment, поэтому меньше downtime и меньше duplicate processing окон.

## Связи

- [Kafka Producer Internals and Tuning](kafka-producer-internals-and-tuning.md)
- [Kafka Delivery Semantics and Transactions](kafka-delivery-semantics-and-transactions.md)
- [Kafka Observability and Production Gotchas](kafka-observability-and-production-gotchas.md)
- [Kafka Streams vs Kafka Consumer](../comparisons/kafka-streams-vs-kafka-consumer.md)
- [Apache Kafka](../entities/apache-kafka.md)
