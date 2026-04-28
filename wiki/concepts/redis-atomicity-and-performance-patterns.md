# Redis Atomicity and Performance Patterns

**Источник:** [Redis in Action](../sources/redis-in-action.md) (гл. 3-4)

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

## Типичные вопросы на интервью

**Q: Чем Redis MULTI/EXEC отличается от SQL-транзакции?**  
A: В Redis MULTI/EXEC гарантирует последовательное выполнение набора команд без интерливинга, но это не полный SQL ACID: нет rollback журнала в стиле реляционных БД и нет сложной изоляции уровней. Для многих cache/state use-cases этого достаточно, но для строгих бизнес-инвариантов часто нужна system-of-record DB.

**Q: Зачем нужен WATCH?**  
A: `WATCH key` реализует optimistic locking: если watched key изменился до `EXEC`, транзакция отменяется (EXEC вернёт null/empty), клиент делает retry. Это аналог compare-and-swap и хороший инструмент для конкурентных обновлений без глобальных lock'ов.

**Q: Pipeline и транзакция — это одно и то же?**  
A: Нет. Pipeline уменьшает сетевые round-trips и повышает throughput, но команды всё ещё могут интерливиться с командами других клиентов. Транзакционная атомарность даётся MULTI/EXEC или Lua.

**Q: Когда выбирать Lua scripts?**  
A: Когда нужна атомарная read-modify-write логика сложнее одной команды и хочется избежать нескольких сетевых hop'ов. Например: rate limit с несколькими ключами, условное обновление score+TTL, idempotency check-and-set. Следить, чтобы скрипт был коротким и не блокировал event loop слишком долго.

**Q: Где главные bottlenecks Redis в production?**  
A: Часто не CPU, а network RTT и memory. Поэтому сначала: pipelining/batching, правильный key design, ограничение cardinality структур, eviction policy, мониторинг hot keys. Только потом — low-level тюнинг.

## Связи

- [Redis](../entities/redis.md)
- [Redis Caching Patterns and Consistency](redis-caching-patterns-and-consistency.md)
- [SQL Transactions, Locking and Isolation](sql-transactions-locking-standard.md)
- [Distributed Systems Pitfalls](distributed-systems-pitfalls.md)
