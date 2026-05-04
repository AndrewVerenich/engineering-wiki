# Redis Pub/Sub and Streams

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (pub/sub, streams, cluster behavior)

## Суть

В Redis есть два разных messaging-инструмента: **Pub/Sub** и **Streams**. На интервью важно объяснить, что они решают разные задачи, а не являются взаимозаменяемыми API.

- Pub/Sub — push-рассылка «здесь и сейчас», без истории и ack.
- Streams — append-only log с persisted сообщениями, consumer groups и retry-механикой.

## Pub/Sub

| Характеристика | Что это значит |
|----------------|----------------|
| Fire-and-forget | offline consumer пропускает сообщения |
| Нет offsets/history | нельзя надежно replay старых событий |
| Низкая задержка | хорошо для live notifications |
| Cluster support | sharded pub/sub (Redis 7+) снижает межузловой broadcast overhead |

## Streams

| Компонент | Назначение |
|-----------|------------|
| `XADD` | append message в stream |
| Consumer Group | распределение чтения между потребителями |
| PEL | учет доставленных, но не подтвержденных сообщений |
| `XACK` | подтверждение обработки |
| `XAUTOCLAIM` | забрать «зависшие» сообщения у умершего consumer |

## Pub/Sub vs Streams vs Kafka (кратко)

| Критерий | Redis Pub/Sub | Redis Streams | Kafka |
|----------|---------------|---------------|-------|
| История сообщений | нет | есть (в пределах retention) | есть, основной сценарий |
| Ack/retry | нет | да | да |
| Replay | нет | ограниченно | сильная модель replay |
| Ops complexity | низкая | средняя | выше |
| Масштаб event log | ограничен Redis memory/архитектурой | средний | высокий |

## Ограничения Streams, о которых часто забывают

- Stream key живет в одном shard slot (нет внутреннего partitioning как в Kafka topic).
- Нужна дисциплина trimming/retention (`XTRIM`), иначе memory growth.
- Consumer group offset и PEL требуют мониторинга «зависших» сообщений.

## Когда что выбирать

| Сценарий | Инструмент |
|----------|------------|
| ephemeral realtime notifications | Pub/Sub |
| background processing с retry/ack | Streams |
| долгий retention и масштабный event backbone | Kafka (или аналог) |

## Типичные вопросы на интервью

**Q: Можно ли на Redis Pub/Sub построить надежную очередь задач?**  
A: Обычно нет, потому что Pub/Sub не хранит историю и не имеет ack/retry semantics. Если consumer был offline, сообщение потеряно. Для reliable processing в Redis нужен Streams с consumer groups.

**Q: Чем Streams принципиально отличаются от Kafka?**  
A: Streams дают persisted log и consumer groups, но у них менее мощная модель масштабирования и экосистема, чем у Kafka. Kafka лучше как центральный event backbone с долгим retention/replay, Redis Streams — как встроенный lightweight broker для сервисного контура.

**Q: Что делает `XAUTOCLAIM` и зачем он нужен?**  
A: Он позволяет новому/здоровому consumer забрать сообщения, зависшие в PEL у умершего или зависшего consumer, после `min-idle-time`. Это ключ к self-healing processing без ручного вмешательства.

**Q: Какой частый anti-pattern с Streams?**  
A: Писать в stream без стратегии retention/trim и без мониторинга PEL. В итоге растет память, а «зависшие» сообщения не перерабатываются, хотя система формально «работает».

## Связи

- [Redis Advanced Data Structures](redis-advanced-data-structures.md)
- [Redis Replication, Sentinel and Cluster](redis-replication-sentinel-cluster.md)
- [Redis Observability and Production Gotchas](redis-observability-and-production-gotchas.md)
- [Redis](../entities/redis.md)
