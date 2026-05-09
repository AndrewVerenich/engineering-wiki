# Kafka Streams Exactly-Once

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Kafka Streams поддерживает `processing.guarantee=exactly_once_v2`, используя транзакции Kafka под капотом. Цель — чтобы запись входа повлияла на state и output-topic **ровно один раз** в наблюдаемом результате, даже при ретраях и сбоях.

## Как это работает

| Компонент | Роль |
|-----------|------|
| **Transactional producer** | Пишет output и offsets атомарно в одной транзакции. |
| **Read-process-write цикл** | Обработка batch входа, обновление state, запись результата. |
| **Commit transaction** | Публикует output и committed offsets как единое действие. |
| **Abort/retry** | При ошибке транзакция откатывается, batch обрабатывается повторно без дублей в committed view. |

## Ограничения

- EOS Kafka Streams в первую очередь про Kafka->Kafka pipeline.
- Внешние side effects (REST, БД без idempotency) не становятся автоматически exactly-once.
- Требует корректной конфигурации broker/client и дисциплины по timeout/retries.

## v2 vs v1 кратко

- `exactly_once_v2` проще операционно и эффективнее по producer lifecycle.
- Лучше масштабируется при большом количестве tasks.
- Рекомендуемый режим для современных версий Kafka.

## Типичные вопросы на интервью

**Q: Дает ли exactly_once_v2 абсолютный exactly-once для всей системы?**  
A: Нет, только для границ, где есть транзакционно согласованные Kafka read/write. Внешние системы требуют идемпотентности или outbox-паттерна.

**Q: Почему offsets важны в той же транзакции?**  
A: Чтобы исключить рассинхрон «результат записан, а offset нет» или наоборот. Это и обеспечивает отсутствие дублей/потерь в committed представлении.

**Q: Почему EOS дороже, чем at-least-once?**  
A: Транзакции добавляют coordination overhead, увеличивают latency и требования к стабильности кластера.

**Q: Когда можно выбрать at-least-once?**  
A: Когда downstream идемпотентен и приоритет — throughput/простота, а не строгая семантика единичной обработки.

## Связи

- [Kafka Delivery Semantics and Transactions](kafka-delivery-semantics-and-transactions.md) — базовая теория delivery guarantees.
- [Kafka Streams Architecture](kafka-streams-architecture.md) — где EOS встроен в runtime.
- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md) — согласованность state.
- [Kafka Backend Patterns: Outbox, DLQ, Retry](kafka-backend-patterns-outbox-dlq-retry.md) — границы EOS во внешних системах.
