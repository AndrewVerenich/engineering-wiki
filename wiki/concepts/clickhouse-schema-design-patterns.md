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

## Типичные вопросы на интервью

**Q: Как ты выбираешь ORDER BY для таблицы в ClickHouse?**
A: Исходя из **самых частых WHERE-фильтров**. Порядок колонок — от **низкой кардинальности к высокой** (сначала event_type, потом date, потом user_id). Причина: sparse index работает по префиксу; если фильтр не на префикс — full scan. Менять ORDER BY потом дорого (миграция данных).

**Q: Зачем LowCardinality(String) и когда его использовать?**
A: Dictionary encoding для строк с малым числом уникальных значений (<10k). Вместо хранения полных строк — хранятся ID из словаря. Экономия памяти и диска, ускорение фильтрации. Использовать для: status, country, event_type. Не использовать для: user_id, UUID, email.

**Q: Зачем Nullable избегать и когда он нужен?**
A: Nullable добавляет отдельную bitmap-колонку для хранения NULL-маски → overhead на storage и query. Использовать только когда NULL имеет **явный бизнес-смысл** (например, `deleted_at IS NULL` = не удалён). Для «нет значения» — лучше DEFAULT (пустая строка, 0, epoch).

**Q: Как устроено partitioning в ClickHouse и чем оно отличается от ORDER BY?**
A: Partitioning — логическая группировка для **lifecycle**: TTL, DROP PARTITION, архивация. ORDER BY — для **отсечения данных при запросах** (sparse index). Partition pruning помогает только если фильтр совпадает с partition expression. Не путать: partitioning — не замена правильного ORDER BY.

## Связи

- [ClickHouse Insert and Mutation Strategy](clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](clickhouse-query-optimization-patterns.md)
- [Storage and Retrieval](storage-and-retrieval.md)
