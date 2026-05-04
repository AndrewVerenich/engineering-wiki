# Redis Internals: Event Loop and Encodings

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (internals, memory optimization, command processing)

## Суть

Redis быстрый не только из-за RAM, но и из-за архитектуры выполнения: **команды исполняются в одном main thread**, что убирает lock contention на структурах данных. Параллелизм достигается в I/O и в горизонтальном масштабировании (replicas/cluster), а не через многопоточную обработку одной keyspace.

Начиная с Redis 6, добавлены **I/O threads** для чтения/записи сокетов, но логика команд и изменение keyspace всё равно выполняется последовательно в main thread. Это сохраняет простую модель атомарности, но делает long-running операции опасными для latency.

## Архитектура обработки команд

| Этап | Что делает | Где bottleneck |
|------|------------|----------------|
| Socket read / parse | Читает RESP, парсит аргументы | network, размер payload |
| Command dispatch | Находит обработчик команды | обычно дешёво |
| Data structure ops | Изменяет dict/expire/index структуры | CPU при сложных операциях |
| Response write | Формирует и отправляет ответ | network egress, slow clients |

## I/O multiplexing и threads

| Механизм | Назначение | Важно помнить |
|----------|------------|---------------|
| `epoll`/`kqueue`/`select` | отслеживание готовности сокетов | минимизирует блокировки на I/O |
| Main event loop | последовательное исполнение команд | предсказуемая atomicity |
| I/O threads (6+) | параллельный read/write sockets | **не** исполняют команды |

## Object encodings (почему это важно)

Redis использует несколько internal encodings для экономии памяти:

| Тип данных | Возможные encodings | Что даёт |
|------------|----------------------|----------|
| String | `int`, `embstr`, `raw` | меньше аллокаций и копирований |
| List | `quicklist` | баланс random access и компактности |
| Hash / ZSet | `listpack` / `hashtable` / `skiplist+hashtable` | компромисс memory vs speed |
| Set | `intset` / `hashtable` | компактность для small-integer set |

Практика: small контейнеры хранятся в компактном виде, при росте cardinality/размера автоматически конвертируются в более быстрые, но более «тяжёлые» encodings.

## Когда single-thread становится проблемой

- Большие `O(N)` команды на hot path (`KEYS`, большие `LRANGE`, massive `SMEMBERS`).
- Длинные Lua/Functions, которые блокируют event loop.
- Большие payload'ы и медленные клиенты (output buffer pressure).
- Небезопасные массовые операции без batching (`DEL` огромных ключей вместо асинхронных стратегий).

## Типичные вопросы на интервью

**Q: Если Redis single-threaded, почему он быстрее многих multi-thread решений?**  
A: Из-за упрощения критического пути: нет lock contention при изменении данных, меньше context switching, компактные структуры и in-memory доступ. На типичных cache workload bottleneck чаще network RTT/throughput, а не CPU. Multi-threaded I/O в Redis 6+ снимает часть сетевого overhead, сохраняя детерминированное исполнение команд.

**Q: Что именно делают I/O threads в Redis 6+?**  
A: Они распараллеливают чтение/запись клиентских сокетов и часть parsing/serialization, но не бизнес-логику команд. Все модификации keyspace идут через main thread, поэтому модель atomicity на уровне команды сохраняется.

**Q: Почему encoding важен для production-тюнинга?**  
A: Encoding напрямую влияет на memory footprint и latency. Например, переход small container -> hashtable/skiplist повышает потребление памяти и может изменить latency profile. Поэтому на senior-уровне мониторят `MEMORY USAGE`, `OBJECT ENCODING`, распределение размерностей ключей и контролируют cardinality.

**Q: Какие команды особенно опасны для latency SLA?**  
A: Команды с большой линейной сложностью по числу элементов (`KEYS`, большой `HGETALL`, `SMEMBERS`, range на огромных коллекциях), длинные Lua-скрипты и массовые удаления без стратегии. Они блокируют main loop и увеличивают tail latency всего инстанса.

## Связи

- [Redis](../entities/redis.md)
- [Redis Memory Management and Eviction](redis-memory-management-and-eviction.md)
- [Redis Atomicity and Performance Patterns](redis-atomicity-and-performance-patterns.md)
- [Redis Advanced Data Structures](redis-advanced-data-structures.md)
