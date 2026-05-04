# Redis Memory Management and Eviction

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (memory optimization, eviction, expiration, ops)

## Суть

В production Redis почти всегда ограничен не CPU, а **памятью**. Поэтому senior-подход к Redis начинается с явного memory budget: сколько RAM доступно под данные, какой fragmentation допустим, какая eviction policy соответствует бизнес-SLA и как быстро система деградирует при memory pressure.

Ключевая мысль: `maxmemory` + eviction policy — это не «техническая деталь», а часть контрактов приложения о потере кэша, stale windows и нагрузке на primary DB.

## Из чего состоит memory profile

| Метрика | Что показывает | Практический смысл |
|--------|-----------------|--------------------|
| `used_memory` | память под данные/структуры Redis | базовый размер dataset |
| `used_memory_rss` | реальный RSS процесса | важно для OOM и cgroup лимитов |
| Fragmentation ratio | `rss / used_memory` | >1 значит overhead/фрагментация |
| `mem_not_counted_for_evict` | память, не входящая в eviction-учёт | может вызвать surprise OOM при плохом запасе |

## `maxmemory` и политики eviction

| Политика | Как вытесняет | Когда использовать |
|----------|---------------|-------------------|
| `noeviction` | не вытесняет, write получает error | когда loss данных недопустим |
| `allkeys-lru` | LRU среди всех ключей | универсальный cache default |
| `allkeys-lfu` | LFU среди всех ключей | когда есть устойчивые hot keys |
| `allkeys-random` | случайный ключ | редко, только для грубых сценариев |
| `volatile-lru` | LRU только среди TTL ключей | mixed workload с persistent + cache |
| `volatile-lfu` | LFU только среди TTL ключей | аналогично, но с frequency bias |
| `volatile-random` | random среди TTL ключей | узкоспец. |
| `volatile-ttl` | keys с ближайшим истечением | когда TTL отражает приоритет |

## Expiration: lazy vs active

| Механизм | Что делает | Trade-off |
|----------|------------|-----------|
| Lazy expiration | удаляет expired key при доступе | дешево, но «мертвые» ключи могут висеть |
| Active expiration | фоновый sampling истекающих ключей | стабильнее по памяти, но потребляет CPU |

На практике нужен баланс: без активной очистки memory usage может отставать от логического TTL-профиля.

## Approximated LRU/LFU и sampling

Redis применяет приближённые версии LRU/LFU через sampling (`maxmemory-samples`), чтобы не хранить дорогой full-ordering для каждого ключа. Большее значение sample повышает качество выбора кандидатов на eviction, но добавляет CPU overhead.

## Типичные operational паттерны

- Перед вводом Redis в критичный контур зафиксировать memory budget и allowed data loss profile.
- Делать отдельные инстансы под persistent state и cache, если политики eviction конфликтуют.
- Регулярно проверять big keys/hot keys (`redis-cli --bigkeys`, `--hotkeys`, `MEMORY USAGE`).
- Использовать TTL jitter для избежания одновременного истечения огромного количества ключей.

## Типичные вопросы на интервью

**Q: Почему Redis может OOM, даже если выставлен `maxmemory`?**  
A: Потому что часть памяти не участвует в eviction-логике (репликационные буферы, внутренние структуры, fragmentation overhead). Если не оставлен запас по RSS, процесс может уткнуться в лимит контейнера/OS до «идеального» срабатывания eviction.

**Q: Чем `allkeys-lru` отличается от `volatile-lru` на практике?**  
A: `allkeys-*` работает по всему keyspace и подходит для чистого cache-инстанса. `volatile-*` затрагивает только TTL keys: если ttl-ключей мало, Redis может не найти кандидатов и начать возвращать ошибки записи. Для mixed workload это осознанный выбор, но требует контроля доли TTL ключей.

**Q: Как выбирать между LRU и LFU?**  
A: LRU лучше для workload, где «недавно использованное» с высокой вероятностью будет снова прочитано. LFU лучше при долгоживущих hot keys: он устойчивее к временным всплескам, потому что учитывает частоту, а не только последнюю активность.

**Q: Что хуже: aggressive eviction или noeviction?**  
A: Зависит от бизнес-семантики. Aggressive eviction увеличивает misses и давление на DB, но сервис продолжает отвечать. `noeviction` сохраняет данные, но может ломать write-path. Senior-решение основано на SLA: что болезненнее — рост latency/DB load или write errors.

## Связи

- [Redis](../entities/redis.md)
- [Redis Internals: Event Loop and Encodings](redis-internals-event-loop-and-encodings.md)
- [Redis Caching Patterns and Consistency](redis-caching-patterns-and-consistency.md)
- [Redis Persistence: RDB vs AOF](redis-persistence-rdb-vs-aof.md)
