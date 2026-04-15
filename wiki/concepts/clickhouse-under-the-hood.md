# ClickHouse под капотом (как работает движок)

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

## 1) Базовая модель исполнения

ClickHouse — columnar engine:

- данные хранятся поколоночно;
- читаются только нужные колонки;
- агрегации и фильтры векторизованы по блокам.

Это дает сильный выигрыш для OLAP: сканы, группировки, time-series, витрины.

## 2) Физическая организация: partition → part → granule

Иерархия хранения:

1. **Partition** — логический диапазон (часто по времени), нужен для lifecycle.  
2. **Part** — физический immutable-файл(набор), создается на каждый insert.  
3. **Granule** — минимальная единица чтения внутри part.

Primary index хранит “маяки” по granules, а не по строкам — отсюда sparse-логика.

## 3) Что происходит на INSERT

Упрощенно:

1. Принимается батч (лучше 10k-100k строк, per `insert-batch-size`).  
2. Данные сортируются по `ORDER BY` для нового part.  
3. Колонки пишутся на диск в сжатом виде.  
4. Part становится видимым для чтения.

Параллельно background merge-потоки начинают консолидировать parts.

## 4) Что происходит на SELECT

Упрощенно:

1. Планировщик анализирует фильтры.  
2. Отсекаются partition'ы и granules по индексам.  
3. Читаются только нужные колонки в оставшихся granules.  
4. Выполняются операции (filter/join/aggregate/sort).  
5. Результат стримится клиенту.

Ключевой момент производительности: сколько данных удалось **не читать**.

## 5) Merge, мутации и eventual consolidation

В MergeTree многие операции “досводятся” в фоне:

- merges укрупняют parts;
- некоторые движки применяют дедуп/суммирование именно на merge;
- DELETE/UPDATE мутации тяжелые, т.к. переписывают parts.

Per `insert-mutation-avoid-update` и `insert-mutation-avoid-delete`: частые изменения лучше моделировать вставками новых версий и lifecycle-операциями (например `DROP PARTITION`).

## 6) Materialized Views в runtime

MV в ClickHouse — это insert-trigger:

- новые строки в source-таблице;
- MV сразу вычисляет projection/агрегацию;
- пишет в target-таблицу.

Per `query-mv-incremental`: для real-time агрегатов использовать incremental MV.  
Per `query-mv-refreshable`: для тяжелых prejoin/кэша — refreshable MV по расписанию.

## 7) Почему ORDER BY решает почти все

Per `schema-pk-plan-before-creation`: ключ нужно проектировать заранее — менять его потом дорого.  
Per `schema-pk-prioritize-filters` и `schema-pk-cardinality-order`: если ключ не совпадает с фильтрами и кардинальностью, движок читает слишком много данных.

## Типичные вопросы на интервью

**Q: Почему ClickHouse быстрый для аналитики?**
A: 1) **Columnar storage** — читаются только нужные колонки. 2) **Compression** — однородные данные в колонке сжимаются лучше (LZ4, ZSTD). 3) **Vectorized execution** — обработка блоками по ~8192 строк, SIMD-инструкции. 4) **Sparse index** — отсечение granules без per-row index overhead. 5) **Append-only model** — writes не конкурируют с reads.

**Q: Объясни иерархию хранения: partition → part → granule.**
A: **Partition** — логическая группа для lifecycle (TTL, DROP). **Part** — физический immutable файл(набор), создаётся на INSERT, содержит все колонки. **Granule** — минимальная единица чтения внутри part (~8192 строк). Primary index ссылается на granules, не на строки.

**Q: Что происходит, когда ClickHouse выполняет SELECT?**
A: 1) Анализ фильтров. 2) Partition pruning. 3) Primary index → отсечение granules. 4) Skipping indices (если есть). 5) Чтение только нужных колонок в оставшихся granules. 6) Filter/join/aggregate/sort. 7) Streaming результата клиенту. Ключ к производительности: сколько данных удалось **не читать**.

**Q: Чем MV в ClickHouse отличается от MV в PostgreSQL?**
A: В ClickHouse MV — это **insert-trigger**: срабатывает на новые строки, пишет результат в target-таблицу. Инкрементальный, стоимость O(batch). В PostgreSQL MV — **snapshot**: `REFRESH MATERIALIZED VIEW` пересчитывает весь запрос. ClickHouse MV дешевле для streaming, но не может пересмотреть историю без refreshable MV.

## Связи

- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse: индексы и background merges](clickhouse-indexes-and-background-merges.md)
- [ClickHouse MergeTree Engines](clickhouse-merge-tree-engines.md)
