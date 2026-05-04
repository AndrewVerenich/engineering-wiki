# Redis Rate Limiting Patterns

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (patterns, scripting, cluster constraints)

## Суть

Rate limiting в Redis — классическая senior-тема, потому что здесь важно не только «ограничить запросы», но и выбрать корректный алгоритм под SLA: точность, memory cost, burst tolerance, latency и cluster-совместимость.

Практически любой production limiter должен быть атомарным (Lua/Functions), observability-ready и устойчивым к retries/clock skew.

## Паттерны ограничения

| Паттерн | Идея | Плюсы | Минусы |
|---------|------|-------|--------|
| Fixed window | `INCR` + `EXPIRE` на окно | простой и быстрый | burst на границе окон |
| Sliding window log | ZSet с timestamp per request | точный лимит | дорогой по памяти/операциям |
| Sliding window counter | агрегированные под-окна | компромисс точности и памяти | сложнее логика |
| Token bucket | токены пополняются со временем | хорошо контролирует burst | нужна аккуратная математика времени |
| Leaky bucket | очередь/дренаж фиксированным rate | стабилизирует output rate | возможны задержки/дропы |

## Почему Lua/Functions почти обязательны

Операции limiter обычно multi-step: read current state -> вычислить allowance -> обновить state -> вернуть решение. Если делать это несколькими round-trip без server-side атомарности, легко получить race conditions.

## Cluster-особенности

| Проблема | Решение |
|----------|---------|
| Multi-key операция на разных слотах | hash tags: `rate:{user123}:sec`, `rate:{user123}:min` |
| Cross-slot Lua | не работает без co-location ключей |
| Hot partition на популярных ключах | шардирование ключа/доп. dimension (api-key, route) |

## Быстрый выбор алгоритма

| Требование | Рекомендация |
|------------|---------------|
| Минимальная сложность | fixed window + TTL jitter |
| Высокая точность fairness | sliding window log |
| Баланс cost/accuracy | sliding window counter |
| Контроль burst + постоянный rate | token bucket |

## Типичные вопросы на интервью

**Q: Почему fixed window может пропускать двойной burst?**  
A: Клиент может отправить N запросов в конце окна и ещё N сразу в начале следующего, получив почти 2N за короткий интервал. Это нормально для дешёвого limiter, но не подходит для строгих anti-abuse сценариев.

**Q: Когда sliding window log слишком дорогой?**  
A: При очень высоком QPS на ключ: нужно хранить timestamp каждого запроса и регулярно чистить старые записи, что увеличивает memory и CPU (ZSet operations O(log N)). Тогда лучше counter/bucket подход.

**Q: Зачем hash tags в Redis Cluster для limiter?**  
A: Чтобы все ключи одного лимитера попали в один hash slot и Lua-операция оставалась атомарной. Без этого получишь cross-slot errors и не сможешь выполнить multi-key скрипт.

**Q: Что чаще ломают в rate limiting реализации?**  
A: 1) Неатомарные read-modify-write операции. 2) Отсутствие метрик (allow/deny, latency, script errors). 3) Игнорирование clock/source-of-time. 4) Единый hot key для всех клиентов.

## Связи

- [Redis Atomicity and Performance Patterns](redis-atomicity-and-performance-patterns.md)
- [Redis Distributed Locks](redis-distributed-locks.md)
- [Redis Memory Management and Eviction](redis-memory-management-and-eviction.md)
- [Redis](../entities/redis.md)
