# Kafka Backend Patterns: Outbox, DLQ, Retry

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

На backend-интервью Kafka почти всегда обсуждают через паттерны доставки, а не через API. Главные темы: как публиковать события из транзакции БД без потерь, как обрабатывать retries без бесконечных циклов и как строить idempotent consumers.

## Outbox pattern

| Шаг | Идея |
|-----|------|
| 1 | бизнес-изменение и запись в outbox в одной DB транзакции |
| 2 | relay-процесс публикует outbox в Kafka |
| 3 | после успешной публикации outbox mark as sent |

Это решает «dual write problem» (DB и Kafka write неатомарны сами по себе).

## Retry и DLQ

| Паттерн | Когда применять |
|---------|-----------------|
| Retry topic chain | transient ошибки downstream |
| Exponential backoff topics | контролируемая повторная доставка |
| DLQ | неисправимые/ядовитые сообщения |

## Idempotent consumer

- dedup key (event id/business id);
- storage of processed ids;
- upsert semantics в target system;
- safe reprocessing при restarts/retries.

## Ordering vs concurrency

Key-affinity (один ключ -> одна partition) сохраняет порядок по сущности, но ограничивает параллелизм. Если порядок не критичен, можно ослабить ключевую стратегию ради throughput.

## Типичные вопросы на интервью

**Q: Почему outbox считается safer, чем «сначала DB, потом Kafka»?**  
A: Потому что «сначала DB, потом Kafka» оставляет окно, где DB commit уже произошел, а publish в Kafka провалился. Outbox переносит publish в отдельный надежный процесс и делает recovery повторяемым.

**Q: Как избежать бесконечного retry loop?**  
A: Ограничивать число попыток, использовать backoff topics и после порога отправлять в DLQ с полной диагностикой. Повторять «навсегда» без контроля — прямой путь к инцидентам и очередям-зомби.

**Q: Почему at-least-once почти всегда требует idempotent consumer?**  
A: Потому что retries/rebalances неизбежно дают дубликаты. Без idempotency дубликаты становятся двойными списаниями, повторными email/side effects и несогласованным состоянием.

## Связи

- [Kafka Delivery Semantics and Transactions](kafka-delivery-semantics-and-transactions.md)
- [Kafka Consumer Internals and Offsets](kafka-consumer-internals-and-offsets.md)
- [Transactions and Isolation](transactions-and-isolation.md)
- [Apache Kafka](../entities/apache-kafka.md)
