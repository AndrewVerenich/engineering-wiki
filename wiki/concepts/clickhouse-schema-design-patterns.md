# ClickHouse Schema Design Patterns

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

Ниже — практический конспект по правилам из официальных best practices.

## 1) ORDER BY и primary key проектируются до создания таблицы

Per `schema-pk-plan-before-creation`: `ORDER BY` для MergeTree нельзя безопасно "переопределить" без миграции данных.  
Per `schema-pk-prioritize-filters`: ключ должен соответствовать частым `WHERE`-фильтрам.  
Per `schema-pk-cardinality-order`: порядок колонок в ключе — от низкой кардинальности к высокой.

Быстрый шаблон:

- сначала категория/тип события;
- затем дата (`toDate(ts)` или `event_date`);
- затем более высококардинальные идентификаторы.

Per `schema-pk-filter-on-orderby`: запросы должны фильтровать префикс `ORDER BY`, иначе индекс почти не поможет.

## 2) Типы данных: нативные, компактные, осмысленные

Per `schema-types-native-types`: не использовать `String` "для всего", а выбирать нативные типы (`UInt*`, `DateTime`, `UUID`, `Decimal`, `Bool`).  
Per `schema-types-minimize-bitwidth`: выбирать минимально достаточную битность.  
Per `schema-types-lowcardinality`: для повторяющихся строк (<10k уникальных) использовать `LowCardinality(String)`.  
Per `schema-types-avoid-nullable`: `Nullable` применять только когда `NULL` имеет явный бизнес-смысл, иначе `DEFAULT`.

## 3) Partitioning — про lifecycle, не про магическое ускорение

Per `schema-partition-lifecycle`: партиционирование в первую очередь для retention/архивации (`TTL`, `DROP PARTITION`).  
Per `schema-partition-low-cardinality`: кардинальность партиций должна оставаться ограниченной (обычно 100-1000 значений).  
Per `schema-partition-query-tradeoffs`: pruning помогает только подходящим фильтрам; запросы через много партиций могут стать тяжелее.  
Per `schema-partition-start-without`: если lifecycle-требования не ясны, можно начать без partitioning.

## 4) JSON и полуструктурированные поля

Per `schema-json-when-to-use`: JSON-тип использовать для реально динамической схемы, а не вместо нормального моделирования типизированных полей.

## Чеклист перед созданием таблицы

1. Зафиксировать топ-фильтры и SLA-запросы.  
2. Выбрать `ORDER BY` по фильтрам и кардинальности.  
3. Проставить нативные типы, `LowCardinality`, минимальную битность.  
4. Решить, нужен ли `Nullable` и partitioning.  
5. Для динамики — точечно использовать JSON.

## Связи

- [ClickHouse Insert and Mutation Strategy](clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](clickhouse-query-optimization-patterns.md)
- [Storage and Retrieval](storage-and-retrieval.md)
