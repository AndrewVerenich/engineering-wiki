# Redis Replication, Sentinel and Cluster

**Источник:** [Redis in Action](../sources/redis-in-action.md) (гл. 5)

## Суть

Redis масштабируется и повышает доступность через три уровня:

1. **Replication** — master-replica копирование данных.
2. **Sentinel** — мониторинг и автоматический failover master'а.
3. **Cluster** — sharding + HA через hash slots.

Главный trade-off: чем выше availability и scale, тем сложнее операционная модель и больше вероятность временной несогласованности при failover.

## Replication

| Аспект | Суть |
|--------|------|
| Модель | асинхронная master -> replicas |
| Использование | read scaling, backup replica, DR |
| Риск | при падении master возможна потеря последних write'ов, не успевших реплицироваться |
| Команда контроля | `WAIT numreplicas timeout` может уменьшить риск loss, но не даёт full consensus semantics |

## Sentinel

| Что делает | Как |
|------------|-----|
| Мониторит master/replica | ping, health checks |
| Выбирает новый master | quorum + election между sentinel-узлами |
| Переключает replicas | reconfigure topology на новый master |
| Уведомляет клиентов | через Sentinel API / service discovery |

Sentinel не даёт sharding — только HA для single master setup.

## Redis Cluster

| Аспект | Суть |
|--------|------|
| Шардирование | 16384 hash slots, каждый key -> slot -> master node |
| HA | master + replicas per shard |
| Масштаб | горизонтальный рост dataset/throughput |
| Ограничения | multi-key операции требуют ключи в одном slot (hash tags) |

## Типичные вопросы на интервью

**Q: Чем Sentinel отличается от Cluster?**  
A: Sentinel = HA orchestration для одной master-replica topology без шардирования. Cluster = и шардирование, и HA: данные распределяются по hash slots между несколькими master'ами с replica для каждого. Если данные не помещаются на один узел, Sentinel не поможет — нужен Cluster.

**Q: Почему Redis replication асинхронная и к чему это ведёт?**  
A: Асинхронность снижает write latency на master, но создаёт окно, когда write подтверждён клиенту, но ещё не на replica. При failover в этот момент эти write'ы могут потеряться. Это стандартный availability vs durability trade-off.

**Q: Как уменьшить риск потерь при failover?**  
A: 1) использовать `WAIT` для подтверждения репликации на N replica, 2) настроить `min-replicas-to-write` и `min-replicas-max-lag`, 3) выбрать подходящую AOF policy, 4) использовать idempotent writes и retry-safe протокол в приложении.

**Q: Что такое hash tags в Redis Cluster?**  
A: Это часть ключа в `{}` (например, `cart:{user42}:items` и `cart:{user42}:meta`) — Cluster хеширует только содержимое `{}`. Так можно гарантировать co-location ключей в одном slot и делать multi-key операции атомарно в пределах этого slot.

**Q: Может ли Redis Cluster дать строгую консистентность?**  
A: Нет в ACID/linearizable смысле для всего кластера. Есть eventual consistency между master/replicas и ограничения на cross-slot операции. Для строгих инвариантов обычно используют внешнюю system-of-record БД и компенсирующую логику.

## Связи

- [Redis](../entities/redis.md)
- [Redis Persistence: RDB vs AOF](redis-persistence-rdb-vs-aof.md)
- [Redis Pub/Sub and Streams](redis-pubsub-and-streams.md)
- [Redis Observability and Production Gotchas](redis-observability-and-production-gotchas.md)
- [Replication](replication.md)
- [Partitioning](partitioning.md)
- [Consistency and Consensus](consistency-and-consensus.md)
