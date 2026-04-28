# Redis

**Тип:** in-memory key-value datastore / cache / lightweight message and data-structure server

## В контексте вики

В [Redis in Action](../sources/redis-in-action.md) Redis используется как низколатентный слой между приложением и долговременным хранилищем: кэширование, счётчики, rate limiting, очереди задач и leaderboards. В доменной структуре вики Redis пересекает **Java Backend** (application cache/session/rate limiting) и **Foundations** (replication, failover, consistency trade-offs).

## Ключевые характеристики

| Характеристика | Суть |
|----------------|------|
| **In-memory first** | Данные хранятся в RAM -> latency обычно 0.1-1 ms. Trade-off: память дороже диска, объём ограничен. |
| **Богатые структуры** | Не только `GET/SET`: Hash, List, Set, Sorted Set, Bitmap, HyperLogLog, Streams. Это позволяет моделировать бизнес-операции без внешних join'ов. |
| **Single-threaded command execution** | Команды выполняются последовательно в event loop (с I/O multiplexing). Простая модель atomicity на уровне одной команды. |
| **Persistence options** | RDB snapshots и/или AOF log. Выбор = trade-off между durability, latency и скоростью recovery. |
| **Replication & HA** | Master-replica replication, Redis Sentinel для failover, Redis Cluster для sharding + HA. |
| **TTL и eviction** | Native expiration per key + политики eviction (`allkeys-lru`, `volatile-ttl` и др.) — база для cache workloads. |
| **Atomic operations** | MULTI/EXEC, Lua scripts, optimistic locking (`WATCH`) для multi-step операций. |

## Типичные вопросы на интервью

**Q: Redis — это database или cache?**  
A: И то, и другое, но по инженерной роли чаще cache/fast state store. Redis даёт persistence (RDB/AOF), replication и failover, поэтому может хранить важные данные, но при строгих durability/consistency требованиях обычно используется вместе с primary DB (PostgreSQL/MySQL), а не вместо неё.

**Q: Почему Redis быстрый?**  
A: 1) In-memory storage, 2) простой wire protocol, 3) single-threaded execution без lock contention на уровне команд, 4) компактные структуры данных. Основное ограничение — RAM и network bandwidth, а не CPU на типичных workload'ах.

**Q: Чем Sentinel отличается от Cluster?**  
A: Sentinel решает **HA без шардирования**: мониторинг, election, failover master'а. Cluster решает и **шардирование, и HA**: ключи распределены по hash slots (16384), каждый slot имеет master/replica. Если нужен один dataset > RAM одного узла, нужен Cluster.

**Q: Когда Redis нельзя использовать как единственный storage?**  
A: Когда критичны строгие ACID-транзакции, сложные ad-hoc запросы/joins, долгий retention большого объёма данных на диске или жёсткая согласованность между многими сущностями. Redis лучше как быстрый state/cache-слой, а system-of-record — в реляционной/документной БД.

**Q: Как выбрать между RDB и AOF?**  
A: RDB — компактные snapshots, быстрый restart, но возможна потеря данных между snapshot'ами. AOF — лучший durability profile (особенно `appendfsync everysec/always`), но больше write amplification и размер файла. На практике часто включают оба: RDB для быстрого восстановления, AOF для меньшего data loss window.

## Связи

- [Redis Data Structures and Modeling](../concepts/redis-data-structures-and-modeling.md)
- [Redis Persistence: RDB vs AOF](../concepts/redis-persistence-rdb-vs-aof.md)
- [Redis Replication, Sentinel and Cluster](../concepts/redis-replication-sentinel-cluster.md)
- [Redis Caching Patterns and Consistency](../concepts/redis-caching-patterns-and-consistency.md)
- [Distributed Systems Pitfalls](../concepts/distributed-systems-pitfalls.md)
