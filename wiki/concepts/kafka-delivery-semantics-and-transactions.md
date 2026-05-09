# Kafka Delivery Semantics and Transactions

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Kafka может работать в разных delivery semantics: at-most-once, at-least-once и effectively/exactly-once в ограниченных сценариях. Реальные гарантии зависят от комбинации producer/consumer настроек и границ системы: внутри Kafka гарантии сильнее, чем на границе с внешними DB/API sinks.

## Семантики доставки

| Семантика | Как достигается | Риск |
|-----------|------------------|------|
| At-most-once | commit offset до обработки / no retry | потери сообщений |
| At-least-once | process -> commit, retries включены | дубликаты |
| Exactly-once (Kafka scope) | idempotent + transactional producer + `read_committed` | сложнее конфиг, внешние sink все еще риск |

## Idempotent producer

- `enable.idempotence=true` включает PID + sequence numbers.
- Брокер отбрасывает повторы от одного producer session при retry.
- Снижает duplicates при временных сетевых сбоях.

## Transactions

| Элемент | Роль |
|---------|------|
| `transactional.id` | стабильный identity транзакционного producer |
| Begin/commit/abort | атомарные границы batch записи |
| `read_committed` | consumer видит только committed transactional records |

EOS-паттерн внутри Kafka: consume -> process -> produce в рамках транзакции.

## Границы exactly-once

Exactly-once в Kafka не означает автоматически exactly-once во внешний мир (например, SQL sink или REST side effect). Там обычно нужен idempotent consumer/upsert/dedup и явный протокол повторов.

## Типичные вопросы на интервью

**Q: Чем idempotent producer отличается от transactional producer?**  
A: Idempotence защищает от duplicate writes при retries для одного producer session. Transactions добавляют атомарность набора записей (в т.ч. между несколькими partitions/topics) и согласование с consumer read_committed flow.

**Q: Почему at-least-once чаще практический default?**  
A: Потому что проще эксплуатационно и устойчиво к сбоям, а дубликаты обычно дешевле обработать через idempotency, чем поддерживать полный transactional pipeline end-to-end.

**Q: Когда exactly-once «ломается» в реальной системе?**  
A: На границе с внешними sink/side effects: Kafka-транзакция не откатывает запись в внешнюю DB/API. Поэтому без idempotent upsert/outbox/inbox паттернов остаются дубликаты и race conditions.

## Связи

- [Transactions and Isolation](transactions-and-isolation.md)
- [Kafka Producer Internals and Tuning](kafka-producer-internals-and-tuning.md)
- [Kafka Consumer Internals and Offsets](kafka-consumer-internals-and-offsets.md)
- [Kafka Streams Exactly-Once](kafka-streams-exactly-once.md)
- [Kafka Backend Patterns: Outbox, DLQ, Retry](kafka-backend-patterns-outbox-dlq-retry.md)
