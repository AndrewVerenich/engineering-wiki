# Redis Observability and Production Gotchas

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (observability, latency troubleshooting, cluster ops)

## Суть

Большинство Redis-инцидентов в production вызваны не «падением Redis», а деградацией: рост tail latency, hot keys, replication lag, неожиданный eviction, full resync или неправильные команды в runtime. Senior-level эксплуатация Redis — это раннее обнаружение этих паттернов и предсказуемый rollback path.

## Базовый observability-набор

| Сигнал | Инструмент | Что ловит |
|--------|------------|-----------|
| Slow commands | `SLOWLOG GET` | команды, блокирующие event loop |
| Latency spikes | `LATENCY DOCTOR`, latency monitor | кратковременные и периодические пики |
| Throughput/errors | `INFO stats`, app metrics | saturating traffic, command errors |
| Memory pressure | `INFO memory` | рост fragmentation, nearing maxmemory |
| Replication health | `INFO replication` | lag, backlog pressure, resync risk |
| Key skew | `--hotkeys`, app cardinality metrics | hot partitions/hot keys |

## Частые production gotchas

| Проблема | Симптом | Что делать |
|----------|---------|------------|
| `KEYS *`/большие `SMEMBERS` в runtime | скачок p99 latency | использовать `SCAN`, pagination, bounded reads |
| Full resync replicas | network spike + disk IO + lag | увеличить `repl-backlog-size`, стабилизировать сеть |
| Hot key | один shard/instance перегружен | key sharding, local cache, request coalescing |
| Memory fragmentation | RSS сильно выше used memory | active defrag, controlled restart windows |
| Cluster redirects storm (`MOVED`/`ASK`) | рост latency и retriable errors | корректная клиентская topology refresh политика |

## Dangerous commands и режимы

- `FLUSHALL`, `FLUSHDB` без multi-layer guardrails.
- `MONITOR` в high-load production (очень тяжёлый).
- Debug-команды в runtime.
- Длинные Lua-скрипты без timeout design.

## Что обязательно мониторить как SLO guardrail

- p95/p99 Redis command latency по типам команд.
- Eviction rate, hit/miss ratio и rejected writes.
- Replication offset lag и частота partial/full resync.
- Cluster slot migration время и доля redirect responses.
- Script failures/timeouts.

## Типичные вопросы на интервью

**Q: Почему `SLOWLOG` может быть полезнее общего CPU графика?**  
A: CPU не покажет, какие именно команды создают tail latency. `SLOWLOG` даёт command-level видимость и помогает быстро найти проблемные паттерны (`HGETALL` на huge hash, mass range queries, Lua scripts).

**Q: Чем опасен hot key, если средняя нагрузка нормальная?**  
A: Redis распределяет нагрузку по ключам, а hot key концентрирует её в одном месте: один core/shard становится bottleneck, tail latency растет, retries усиливают шторм. Средние метрики могут «скрывать» проблему до момента деградации.

**Q: Что означает рост `MOVED`/`ASK` в Redis Cluster?**  
A: Обычно это признак resharding/migration или устаревшей topology у клиента. Небольшой рост нормален при перестройке, устойчиво высокий — сигнал, что клиент неэффективно обновляет карту слотов.

**Q: Почему full resync реплики считается дорогой операцией?**  
A: Потому что требует пересылки большого объёма данных и дополнительных IO/CPU ресурсов на master и replica. Частые full resync в пике могут усугубить инцидент вместо восстановления.

## Связи

- [Redis Memory Management and Eviction](redis-memory-management-and-eviction.md)
- [Redis Replication, Sentinel and Cluster](redis-replication-sentinel-cluster.md)
- [Redis Pub/Sub and Streams](redis-pubsub-and-streams.md)
- [Redis](../entities/redis.md)
