# Redis in Action

| Поле | Значение |
|------|----------|
| **Автор** | Josiah L. Carlson |
| **Издательство** | Manning Publications |
| **Год/Издание** | 2013 (1st edition) |
| **Тип** | книга |

## О книге

Практическая книга про Redis как in-memory datastore для backend-задач: кэширование, очереди, счётчики, leaderboard'ы, rate limiting, pub/sub и базовые аспекты эксплуатации (persistence, replication, sharding). Главная ценность — не синтаксис команд, а инженерные паттерны: как моделировать данные под структуры Redis и как держать баланс между latency, consistency и durability.

## Структура

1. First steps with Redis — ключи, строковые/хеш/списки/множества/ZSET.
2. Structuring and searching data — моделирование через secondary indexes, sorted sets, ranking.
3. Commands in applications — pipeline, transactions, Lua scripts.
4. Data security and performance — оптимизация, memory constraints.
5. Networking and scaling — replication, sharding, failover.

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| Redis как entity (roles, HA, persistence, use-cases) | [Redis](../entities/redis.md) |
| Структуры данных и data modeling | [Redis Data Structures and Modeling](../concepts/redis-data-structures-and-modeling.md) |
| RDB/AOF persistence trade-offs | [Redis Persistence: RDB vs AOF](../concepts/redis-persistence-rdb-vs-aof.md) |
| Replication, Sentinel, Cluster | [Redis Replication, Sentinel and Cluster](../concepts/redis-replication-sentinel-cluster.md) |
| Caching patterns и консистентность | [Redis Caching Patterns and Consistency](../concepts/redis-caching-patterns-and-consistency.md) |
| Atomic operations: pipelines, MULTI/EXEC, Lua | [Redis Atomicity and Performance Patterns](../concepts/redis-atomicity-and-performance-patterns.md) |

## Связи

- [Java Backend Fundamentals](../overviews/java-backend-fundamentals.md)
- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md)
- [Distributed Systems Pitfalls](../concepts/distributed-systems-pitfalls.md)
