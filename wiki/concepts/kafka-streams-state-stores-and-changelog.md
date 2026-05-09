# Kafka Streams State Stores and Changelog

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Stateful-обработка в Kafka Streams строится на локальном state store (часто RocksDB) и журнале изменений в Kafka (**changelog topic**). Такой дизайн даёт низкую latency локальных lookup и при этом позволяет восстановить состояние после падения инстанса.

## Где хранится state

| Вариант | Плюсы | Минусы |
|---------|-------|--------|
| **RocksDB store** | Большой объём state, устойчивость к GC, зрелый production-path. | Диск/IO overhead, tuning сложнее. |
| **In-memory store** | Минимальная latency, простота. | Ограничен RAM, риск долгого восстановления. |

## Changelog и восстановление

- Для state store создаётся internal changelog topic.
- Любое изменение state записывается в changelog.
- При rebalance/failover новый владелец task восстанавливает store чтением changelog.
- Для compacted changelog важно корректно управлять ключами и tombstone.

## Standby replicas

- `num.standby.replicas` поддерживает копии state на других инстансах.
- Standby обновляются из changelog и не исполняют активную обработку.
- При падении активного task failover быстрее, потому что большая часть state уже локально доступна.

## Типичные вопросы на интервью

**Q: Почему Kafka Streams хранит state локально, а не только в Kafka?**  
A: Локальный store нужен для быстрых чтений в процессе обработки. Kafka выступает как durable changelog для восстановления и репликации состояния.

**Q: Что произойдёт при падении инстанса со stateful task?**  
A: Task переедет на другой инстанс, который восстановит state из changelog (или быстро переключится на standby-копию, если она есть).

**Q: Когда in-memory store оправдан?**  
A: Когда state небольшой, критична latency и допустим более жёсткий контроль над объёмом данных.

**Q: Какие operational-риски у RocksDB stores?**  
A: Диск может стать bottleneck, нужны мониторинг restore-lag, размер local state и корректные retention/compaction настройки changelog.

## Связи

- [Kafka Streams Architecture](kafka-streams-architecture.md) — tasks/threads и размещение state.
- [Kafka Streams Exactly-Once](kafka-streams-exactly-once.md) — согласованность state и output.
- [Kafka Topic Design and Compaction](kafka-topic-design-and-compaction.md) — compaction для changelog topics.
- [Kafka Streams Interactive Queries and Deployment](kafka-streams-interactive-queries-and-deployment.md) — как state используется в runtime-сервисе.
