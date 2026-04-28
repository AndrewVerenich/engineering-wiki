# Redis Persistence: RDB vs AOF

**Источник:** [Redis in Action](../sources/redis-in-action.md) (гл. 4-5)

## Суть

Redis по умолчанию in-memory, но может писать данные на диск двумя механизмами: **RDB snapshots** и **AOF (Append Only File)**. Это не «какой лучше», а осознанный выбор durability/latency/recovery trade-off. RDB лучше для быстрых backup/restart, AOF — для меньшего окна потери данных.

## RDB и AOF: сравнение

| Критерий | RDB | AOF |
|----------|-----|-----|
| Модель | периодический snapshot всего dataset | лог каждой write-команды |
| Потеря данных при crash | до интервала snapshot | обычно до 1 сек (`everysec`) или меньше |
| Restart speed | быстрее (загрузка одного snapshot) | медленнее (replay log, если большой файл) |
| Размер на диске | обычно компактнее | обычно больше |
| CPU/IO overhead на write path | ниже | выше |
| Human readability | бинарный dump | текстовый command log |

## AOF fsync policies

| Политика | Durability | Latency/IO cost |
|----------|------------|-----------------|
| `always` | максимум (каждая команда fsync) | самый высокий |
| `everysec` | компромисс (до ~1 секунды потерь) | стандарт для production |
| `no` | минимальная (OS decides) | самый низкий overhead |

## Практические конфигурации

1. **Кэш-сценарий, данные легко восстановимы**  
   - Часто RDB редко или вообще без persistence.
2. **Смешанный workload**  
   - RDB + AOF (`everysec`) вместе.
3. **Высокая durability**  
   - AOF (`always`/`everysec`) + репликация + external backups.

## Типичные вопросы на интервью

**Q: Почему часто включают и RDB, и AOF одновременно?**  
A: Они решают разные задачи. AOF уменьшает окно потери данных при сбое, RDB ускоряет recovery и удобен для backup/transfer. Комбинация даёт более сбалансированный профиль: приемлемая durability + быстрый restart.

**Q: Что означает `appendfsync everysec` в терминах риска?**  
A: Redis fsync'ит AOF примерно раз в секунду. При power loss можно потерять до ~1 секунды последних подтверждённых клиенту write'ов. Для большинства cache/session/rate-limit сценариев это приемлемо, для финансовых ledger-подобных задач — обычно нет.

**Q: Почему AOF-файл нужно rewrite'ить?**  
A: Со временем AOF растёт, потому что хранит историю команд (`INCR`, `SET`, `DEL`). `BGREWRITEAOF` компактизирует лог до минимальной последовательности команд для текущего состояния. Без rewrite восстановление замедляется и диск расходуется быстрее.

**Q: Что произойдёт при сбое во время snapshot/rewrite?**  
A: Redis делает snapshot/rewrite через fork + copy-on-write: родитель продолжает обслуживать клиентов, дочерний процесс пишет файл. При сбое обычно сохраняется старый консистентный файл, а временный повреждённый отбрасывается. Цена — всплеск памяти из-за COW при активных writes.

**Q: Можно ли считать Redis durable database как PostgreSQL?**  
A: Не в том же смысле. Redis даёт persistence, но durability profile зависит от конфигурации (RDB/AOF policy), репликации и дисциплины эксплуатации. PostgreSQL ориентирован на durable system-of-record по умолчанию. Redis чаще layer for speed, а не primary source of truth.

## Связи

- [Redis](../entities/redis.md)
- [Redis Replication, Sentinel and Cluster](redis-replication-sentinel-cluster.md)
- [Distributed Systems Pitfalls](distributed-systems-pitfalls.md)
- [Storage and Retrieval](storage-and-retrieval.md)
