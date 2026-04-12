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

## Связи

- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse: индексы и background merges](clickhouse-indexes-and-background-merges.md)
- [ClickHouse MergeTree Engines](clickhouse-merge-tree-engines.md)
