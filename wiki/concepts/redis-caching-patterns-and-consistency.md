# Redis Caching Patterns and Consistency

**Источник:** [Redis in Action](../sources/redis-in-action.md) (гл. 3-5)

## Суть

Redis в backend чаще всего используется как cache между приложением и primary DB. Сложность не в `GET/SET`, а в выборе паттерна синхронизации с источником истины и в контроле cache invalidation. Основной trade-off: latency vs consistency.

## Основные cache-паттерны

| Паттерн | Как работает | Плюсы | Риски |
|---------|--------------|-------|-------|
| **Cache-aside (lazy loading)** | read: cache miss -> DB -> cache; write: DB + invalidate cache | простой, самый популярный | stale read между update и invalidate |
| **Write-through** | write сразу в DB и cache | читаем всегда из cache | больше write latency |
| **Write-behind (write-back)** | write в cache, flush в DB позже | очень быстрые writes | риск потери данных, сложный recovery |
| **Read-through / refresh-ahead** | cache сам подгружает/обновляет hot keys | меньше cold misses | сложнее инфраструктурно |

## TTL и invalidation

| Техника | Когда использовать |
|---------|--------------------|
| **Fixed TTL** | данные относительно статичны и допустима небольшая staleness |
| **Versioned key** | безопасные обновления без гонок (`user:42:v17`) |
| **Explicit invalidation by key** | после транзакции в DB |
| **Tag-based invalidation** | группа ключей, связанных сущностью/категорией |

## Cache stampede / thundering herd

Проблема: hot key истёк -> много клиентов одновременно бьют в DB.

Решения:
- jitter на TTL (распределяем expirations),
- single-flight lock per key,
- stale-while-revalidate,
- prewarming for critical keys.

## Типичные вопросы на интервью

**Q: Почему cache invalidation считается сложной?**  
A: Потому что система распределённая и есть гонки между write в DB, invalidate cache, и параллельными read. Даже при правильном порядке операций возможны короткие stale windows. Нужно проектировать с явным SLA на staleness и idempotent retry-safe логику.

**Q: Когда выбирать cache-aside, а когда write-through?**  
A: Cache-aside — дефолт для большинства систем: проще, дешевле по write path, хорошо для read-heavy нагрузок. Write-through — когда важно минимизировать stale reads после write и приемлемо платить latency на каждом write. Для ultra-write-heavy обычно смотрят write-behind, но он самый рискованный по durability.

**Q: Что такое cache stampede и как его лечить?**  
A: Stampede возникает, когда одновременно истекает hot key и тысячи запросов идут в DB. Лечат: 1) TTL jitter, 2) lock/single-flight на recompute key, 3) stale-while-revalidate, 4) async refresh ahead. Без этого cache может ухудшить, а не улучшить устойчивость системы.

**Q: Нужно ли кэшировать всё подряд?**  
A: Нет. Кэшируют дорогие и часто читаемые данные с высокой reuse. Не кэшируют высокоизменчивые данные с низкой hit rate, данные со строгим consistency SLA без подходящего invalidate-протокола и огромные payload'ы, которые быстро съедят память.

**Q: Как связать Redis cache с транзакцией в PostgreSQL?**  
A: Типичный паттерн: 1) commit в PostgreSQL (source of truth), 2) invalidate/update Redis post-commit. Для надёжности используют outbox/event-driven invalidation и идемпотентные handlers. Никогда не считать cache source of truth для критичных инвариантов.

## Связи

- [Redis](../entities/redis.md)
- [Redis Data Structures and Modeling](redis-data-structures-and-modeling.md)
- [Redis Atomicity and Performance Patterns](redis-atomicity-and-performance-patterns.md)
- [Reliability, Scalability, Maintainability](reliability-scalability-maintainability.md)
- [Transactions and Isolation](transactions-and-isolation.md)
