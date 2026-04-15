# ClickHouse Query Optimization Patterns

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

## 1) Фильтрация и ORDER BY

Per `schema-pk-filter-on-orderby`: фильтры должны использовать префикс `ORDER BY`, чтобы первичный sparse index реально отрабатывал.

Практическое правило:

- если фильтр не попадает в префикс `ORDER BY`, ждать full scan-подобное поведение;
- смотреть `EXPLAIN indexes = 1` для проверки, что используется PrimaryKey/Skip.

## 2) Data skipping indices — точечный инструмент

Per `query-index-skipping-indices`: skipping indices (`bloom_filter`, `set(N)`, `minmax`) полезны для колонок, которые не входят в `ORDER BY`, но часто участвуют в фильтрах.

Важно: сначала оптимизировать schema/key, и только потом добавлять skip-indices.

## 3) JOIN-паттерны

Per `query-join-filter-before`: фильтровать и/или агрегировать таблицы до join, чтобы не соединять лишний объем.  
Per `query-join-use-any`: когда нужен любой один матч, использовать `ANY JOIN`.  
Per `query-join-choose-algorithm`: алгоритм join выбирать по размеру и памяти (`auto`, `parallel_hash`, `partial_merge`, `full_sorting_merge`, `grace_hash`).  
Per `query-join-consider-alternatives`: в hot-path рассматривать dictionaries и денормализацию вместо частых join.

## 4) Materialized Views: incremental vs refreshable

Per `query-mv-incremental`: incremental MV эффективны для real-time pre-aggregation (`State` при записи, `Merge` при чтении).  
Per `query-mv-refreshable`: refreshable MV подходит для сложных join/кэша с допустимой stale-окном.

Выбор:

- near-real-time счетчики/фаннелы: incremental MV;
- периодический prejoin/витрина: refreshable MV.

## Чеклист для slow query

1. Проверить фильтр на соответствие `ORDER BY`-префиксу.  
2. Проверить `EXPLAIN indexes = 1`.  
3. При необходимости добавить skip-index на проблемный фильтр.  
4. Упростить join (pre-filter, ANY, dictionary/denormalization).  
5. Перенести тяжелые агрегаты в MV/предагрегации.

## Типичные вопросы на интервью

**Q: Запрос медленный — что ты проверишь первым?**
A: 1) `EXPLAIN indexes=1` — используется ли primary index? Попадает ли фильтр в префикс ORDER BY? 2) Сколько granules/rows читается — может быть full scan из-за плохого фильтра. 3) JOIN — можно ли pre-filter или заменить на dictionary? 4) Есть ли тяжёлые агрегаты, которые стоит вынести в MV?

**Q: Чем data skipping index отличается от primary index?**
A: **Primary (sparse) index** — по granules в порядке ORDER BY; работает для префикса ключа. **Skipping index** — дополнительные структуры (bloom_filter, set, minmax) по блокам для колонок **вне** ORDER BY. Skipping index — «второй шаг»: сначала правильный ORDER BY, потом скиппинг для оставшихся фильтров.

**Q: Когда использовать incremental MV, а когда refreshable?**
A: **Incremental** — для real-time pre-aggregation (счётчики, uniq, funnel). Срабатывает на каждый INSERT, стоимость O(новые строки). **Refreshable** — для тяжёлых join/prejoin, которые допускают stale-окно (обновление по расписанию). Пример: incremental — `hourly_clicks_count`; refreshable — `daily_customer_360_view`.

**Q: Как оптимизировать JOIN в ClickHouse?**
A: 1) Фильтровать/агрегировать **до** join. 2) `ANY JOIN` если нужен один матч. 3) Выбрать алгоритм по размеру (`parallel_hash` для маленькой правой стороны, `full_sorting_merge` для больших). 4) В hot-path — **dictionary** или денормализация вместо join.

## Связи

- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse Insert and Mutation Strategy](clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse: индексы и background merges](clickhouse-indexes-and-background-merges.md)
- [ClickHouse MergeTree Engines](clickhouse-merge-tree-engines.md)
- [ClickHouse + Kafka Ingestion](clickhouse-kafka-ingestion.md)
