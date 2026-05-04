# Redis Atomicity and Performance Patterns

**Источник:** [Redis in Action](../sources/redis-in-action.md) (гл. 3-4), [Redis Official Documentation](../sources/redis-official-documentation.md) (Lua scripting, programmability)

## Суть

Redis команды атомарны по одной штуке, но реальные бизнес-операции часто требуют несколько шагов. Для этого используются `MULTI/EXEC`, `WATCH` и Lua scripts. Параллельно производительность сильно зависит от сетевых round-trips — `pipelining` часто даёт больший выигрыш, чем микрооптимизация отдельных команд.

## Atomicity инструменты

| Инструмент | Для чего | Ограничения |
|------------|----------|-------------|
| **Single command atomicity** | одна команда (`INCR`, `HSET`, `ZADD`) выполняется целиком | только в рамках одной команды |
| **MULTI/EXEC** | пакет команд как транзакция (без интерливинга других клиентов) | нет rollback как в SQL, ошибки команд обрабатываются отдельно |
| **WATCH + MULTI/EXEC** | optimistic concurrency control (CAS-style) | при конфликте нужна retry loop |
| **Lua scripts (`EVAL`)** | сложная атомарная логика на сервере в одном шаге | скрипт должен быть детерминирован и быстрый |

## Pipelining

| Что делает | Эффект |
|------------|--------|
| Отправляет много команд без ожидания ответа на каждую | сокращает network RTT overhead, повышает throughput |

Важно: pipeline не равен транзакции. Он оптимизирует транспорт, но не добавляет atomicity между командами.

## Когда Lua лучше MULTI/EXEC

Lua предпочтителен, когда:
- есть read-modify-write с ветвлением,
- нужно минимизировать round-trips,
- нужна строгая атомарность сложной операции на одном ключе/наборе ключей.

## Lua scripting: зачем и как

### Зачем

| Проблема без Lua | Что даёт Lua |
|------------------|--------------|
| много сетевых hop'ов при multi-step логике | один `EVAL`/`EVALSHA` round-trip |
| race condition между командами клиента | атомарное исполнение на сервере |
| сложная retry-логика в приложении | детерминированный check-and-set в одном блоке |

Типичные кейсы: rate limiting, distributed lock release (compare-and-delete), idempotency guard, условные обновления c TTL.

### Как

1. Передать ключи через `KEYS[]`, аргументы через `ARGV[]`.
2. Внутри скрипта использовать `redis.call(...)` для строгого поведения (или `redis.pcall(...)` если нужна обработка ошибок).
3. Держать скрипт коротким и детерминированным.
4. В бою вызывать через `EVALSHA`, чтобы не пересылать тело скрипта каждый раз.

### Минимальный пример (lock release)

```lua
if redis.call("GET", KEYS[1]) == ARGV[1] then
  return redis.call("DEL", KEYS[1])
else
  return 0
end
```

Смысл: удаляем lock только если token совпадает с владельцем, иначе не трогаем ключ.

### Ограничения и anti-patterns

| Ограничение | Почему важно |
|-------------|--------------|
| Скрипт блокирует main event loop на время выполнения | длинные скрипты увеличивают p99 для всех клиентов |
| В Cluster все ключи скрипта должны быть в одном slot | иначе получишь cross-slot error |
| Нельзя полагаться на недетерминированные побочные эффекты | репликация и воспроизводимость операций усложняются |
| Lua не заменяет system-of-record транзакции | инварианты между Redis и внешней БД всё равно требуют протокол |

## Типичные вопросы на интервью

**Q: Чем Redis MULTI/EXEC отличается от SQL-транзакции?**  
A: В Redis MULTI/EXEC гарантирует последовательное выполнение набора команд без интерливинга, но это не полный SQL ACID: нет rollback журнала в стиле реляционных БД и нет сложной изоляции уровней. Для многих cache/state use-cases этого достаточно, но для строгих бизнес-инвариантов часто нужна system-of-record DB.

**Q: Зачем нужен WATCH?**  
A: `WATCH key` реализует optimistic locking: если watched key изменился до `EXEC`, транзакция отменяется (EXEC вернёт null/empty), клиент делает retry. Это аналог compare-and-swap и хороший инструмент для конкурентных обновлений без глобальных lock'ов.

**Q: Pipeline и транзакция — это одно и то же?**  
A: Нет. Pipeline уменьшает сетевые round-trips и повышает throughput, но команды всё ещё могут интерливиться с командами других клиентов. Транзакционная атомарность даётся MULTI/EXEC или Lua.

**Q: Когда выбирать Lua scripts?**  
A: Когда нужна атомарная read-modify-write логика сложнее одной команды и хочется избежать нескольких сетевых hop'ов. Например: rate limit с несколькими ключами, условное обновление score+TTL, idempotency check-and-set. Следить, чтобы скрипт был коротким и не блокировал event loop слишком долго.

**Q: Почему `EVALSHA` предпочтительнее `EVAL` в горячем пути?**  
A: Потому что `EVALSHA` вызывает уже загруженный скрипт по SHA и уменьшает сетевой overhead. Это особенно заметно при высоком QPS, когда пересылка тела скрипта на каждый запрос становится лишней нагрузкой.

**Q: Как понять, что Lua-скрипт вреден для production latency?**  
A: Симптомы: рост p99 команд и записи в `SLOWLOG` именно на `EVAL`/`EVALSHA`. Причины обычно в длинных циклах по большим структурам, heavy branching и отсутствии лимитов по размеру обрабатываемых данных в скрипте.

**Q: Где главные bottlenecks Redis в production?**  
A: Часто не CPU, а network RTT и memory. Поэтому сначала: pipelining/batching, правильный key design, ограничение cardinality структур, eviction policy, мониторинг hot keys. Только потом — low-level тюнинг.

## Связи

- [Redis](../entities/redis.md)
- [Redis Internals: Event Loop and Encodings](redis-internals-event-loop-and-encodings.md)
- [Redis Caching Patterns and Consistency](redis-caching-patterns-and-consistency.md)
- [Redis Distributed Locks](redis-distributed-locks.md)
- [Redis Rate Limiting Patterns](redis-rate-limiting-patterns.md)
- [Redis Observability and Production Gotchas](redis-observability-and-production-gotchas.md)
- [SQL Transactions, Locking and Isolation](sql-transactions-locking-standard.md)
- [Distributed Systems Pitfalls](distributed-systems-pitfalls.md)
