# ClickHouse: индексы и background merges

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

## 1) Какие индексы есть в MergeTree

### Primary index (sparse)

В ClickHouse primary index не хранит ссылку на каждую строку. Он хранит ключи по **granule** (обычно блок на ~8192 строк), поэтому называется sparse index.

- Читается только нужный диапазон granules.
- Максимальная польза, когда фильтры используют префикс `ORDER BY`.
- Per `schema-pk-filter-on-orderby`: фильтровать нужно по префиксу ключа.

### Partition pruning

Партиционирование дает отдельный уровень отсечения данных: движок может вообще не открывать неподходящие partition'ы.

- Per `schema-partition-lifecycle`: основной смысл partitioning — lifecycle/retention.
- Per `schema-partition-query-tradeoffs`: pruning ускоряет только релевантные запросы.

### Data skipping indices

Дополнительные индексы по блокам (`minmax`, `set(N)`, `bloom_filter`, `ngrambf_v1`, `tokenbf_v1`) помогают для колонок вне `ORDER BY`.

- Per `query-index-skipping-indices`: это второй шаг после хорошего `ORDER BY` и типов.
- Проверка пользы: `EXPLAIN indexes = 1`.

## 2) Как запрос реально читает данные

Упрощенный пайплайн:

1. Анализ фильтров.  
2. Отсечение partition'ов (если применимо).  
3. Применение primary index по granules.  
4. Применение skipping indices.  
5. Чтение только оставшихся granules.

Именно поэтому плохой `ORDER BY` часто нельзя “починить” одним skip-index.

## 3) Что такое parts и почему они важны

Каждый `INSERT` создает **part**. Внутри part данные уже отсортированы по `ORDER BY`.

- Слишком много мелких inserts ⇒ слишком много parts.
- Это увеличивает overhead на чтение и фоновые merge-задачи.
- Per `insert-batch-size`: целиться в батчи 10k-100k строк.

## 4) Background merges: механизм

Фоновые merge-потоки постоянно выбирают parts и сливают их в более крупные.

При merge:

- данные заново записываются в новый part;
- применяются правила конкретного движка (например суммирование/дедуп);
- старые parts удаляются после успешной замены.

Per `insert-optimize-avoid-final`: в норме нужно доверять background merges, а не регулярно делать `OPTIMIZE ... FINAL`.

## 5) Merge и корректность результата

Пока merge не завершился, в таблице могут жить несколько версий данных (для некоторых движков семейства MergeTree это нормальное состояние).

Практика:

- где нужна “финальная” консолидация в запросе, использовать `FINAL` в `SELECT`;
- не строить регулярные cron на `OPTIMIZE FINAL` без крайней необходимости.

## 6) Операционные симптомы и причины

| Симптом | Частая причина | Что проверить |
|--------|----------------|---------------|
| Рост latency | Слишком много parts | `system.parts` (active parts) |
| “Too many parts” | Мелкие insert'ы | размер батча, async insert |
| Плохое pruning | Фильтры мимо `ORDER BY` | `EXPLAIN indexes = 1` |
| Heavy I/O в фоне | Агрессивные мутации/оптимизации | UPDATE/DELETE/OPTIMIZE usage |

## Типичные вопросы на интервью

**Q: Что такое sparse index в ClickHouse и чем он отличается от B-tree?**
A: Sparse index хранит ключи **по granule** (~8192 строк), а не по каждой строке. При чтении отсекаются целые блоки granules. Плюс: компактный индекс, быстрый для аналитики (scan-heavy). Минус: точечный lookup неточный (читает granule, а не строку). B-tree — per-row, точный, но дороже по памяти и поддержке.

**Q: Что такое parts и почему их количество важно?**
A: Каждый INSERT создаёт immutable **part** (набор файлов по колонкам). Много мелких parts → overhead на чтение (открыть/прочитать каждый) и на merge (background I/O). Мониторить: `SELECT count() FROM system.parts WHERE active AND table='...'`. Target: сотни-тысячи active parts, не десятки тысяч.

**Q: Как работает background merge?**
A: Фоновые потоки выбирают parts и сливают в более крупные: данные перезаписываются, правила движка применяются (дедуп в Replacing, сумма в Summing), старые parts удаляются. Это **eventual consolidation** — до завершения merge данные могут быть «не досведены». Для строгого результата — `SELECT ... FINAL`.

**Q: Когда использовать OPTIMIZE FINAL?**
A: Почти **никогда** в production cron. OPTIMIZE FINAL принудительно мержит все parts — тяжёлый I/O, блокирует нормальные merges. Использовать: одноразово перед крупным анализом, или для тестирования корректности ReplacingMergeTree. В runtime — `SELECT ... FINAL` вместо `OPTIMIZE FINAL`.

## Связи

- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse Insert and Mutation Strategy](clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](clickhouse-query-optimization-patterns.md)
- [ClickHouse MergeTree Engines](clickhouse-merge-tree-engines.md)
