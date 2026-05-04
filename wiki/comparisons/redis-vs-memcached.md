# Redis vs Memcached

**Контекст:** обе технологии часто используются как in-memory cache, но имеют разные архитектурные trade-offs и operational profile.

## Главная разница

| Критерий | Redis | Memcached |
|----------|-------|-----------|
| Модель данных | key-value + rich data structures (Hash, ZSet, Streams, etc.) | key-value строки/байты |
| Persistence | RDB/AOF (опционально) | нет встроенной durability |
| Replication/HA | replication, Sentinel, Cluster | обычно client-side sharding, HA на уровне приложения |
| Threading | single-thread command execution + I/O threads | multi-threaded processing |
| Операционный профиль | больше функций, сложнее эксплуатация | проще и легче по operational surface |
| Лучший fit | cache + state/patterns (rate limit, queue, lock) | ultra-simple distributed cache |

## Когда Redis лучше

- Нужны структуры данных и серверные операции (ZSet ranking, Lua, Streams).
- Нужна встроенная репликация/cluster/failover.
- Нужно совмещать cache и lightweight state coordination.

## Когда Memcached лучше

- Чистый cache-слой без требований к persistence.
- Максимально простая эксплуатация и предсказуемый runtime.
- Нагрузки, где выгодна простая multi-thread архитектура и нет потребности в Redis-функциях.

## Риски и анти-паттерны выбора

| Ошибка | Последствие |
|--------|-------------|
| Выбрать Redis «на всякий случай», но использовать только GET/SET | лишняя операционная сложность |
| Выбрать Memcached при необходимости надежной очереди/lock | рост custom logic и риск ошибок |
| Считать Redis replacement for primary DB | несоответствие durability/consistency ожиданий |

## Типичные вопросы на интервью

**Q: Если нужен только кэш с TTL, зачем Redis, если есть Memcached?**  
A: Если действительно нужен только простой cache, Memcached может быть рациональнее по простоте и эксплуатационной стоимости. Redis оправдан, когда нужны data structures, scripting, HA-фичи или дополнительные backend-паттерны помимо plain cache.

**Q: Почему Redis часто выбирают в backend даже для кэша?**  
A: Из-за универсальности: на одной платформе закрываются cache, counters, rate limits, distributed locks, lightweight queues/streams. Это уменьшает количество компонентов, но требует более зрелой эксплуатации.

**Q: Можно ли считать Memcached «быстрее Redis» всегда?**  
A: Нет. Производительность зависит от workload (payload, network, hit ratio, pipeline, command mix). На простых GET/SET сценариях Memcached может быть очень эффективен, но при сложных операциях Redis выигрывает функционально и часто архитектурно.

## Связи

- [Redis](../entities/redis.md)
- [Redis Caching Patterns and Consistency](../concepts/redis-caching-patterns-and-consistency.md)
- [Redis Memory Management and Eviction](../concepts/redis-memory-management-and-eviction.md)
- [Redis Observability and Production Gotchas](../concepts/redis-observability-and-production-gotchas.md)
