# ClickHouse Insert and Mutation Strategy

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

## 1) Размер батчей и частота вставок

Per `insert-batch-size`: целевой размер batch обычно 10k-100k строк на `INSERT` (минимум около 1k).  
Причина: каждый `INSERT` создает part; слишком мелкие вставки ведут к part explosion и деградации merge-процесса.

Per `insert-format-native`: для максимальной производительности предпочтителен `Native` format.

## 2) Async insert для частых мелких сообщений

Per `insert-async-small-batches`: когда клиентское батчирование ограничено, включать:

- `async_insert = 1`
- `wait_for_async_insert = 1` (безопасный режим подтверждения)

Это позволяет серверу буферизовать мелкие вставки и записывать более крупными parts.

## 3) Избегать тяжелых мутаций как регулярного механизма

Per `insert-mutation-avoid-update`: не строить потоковые/частые обновления через `ALTER TABLE ... UPDATE`; для upsert-паттернов использовать versioned insert + движки вроде ReplacingMergeTree.

Per `insert-mutation-avoid-delete`: не использовать `ALTER TABLE ... DELETE` как обычный cleanup-механизм; предпочитать:

- `DROP PARTITION` для массового удаления по времени,
- lightweight `DELETE` для умеренных сценариев,
- модели soft-delete (например sign/version-подходы), если частые удаления — норма.

## 4) OPTIMIZE FINAL — только для редких спецслучаев

Per `insert-optimize-avoid-final`: не запускать `OPTIMIZE TABLE ... FINAL` после каждого батча.  
Обычный режим — доверять background merges. Для дедупликации ReplacingMergeTree в чтении чаще нужен `FINAL` в `SELECT`, а не `OPTIMIZE FINAL`.

## Операционный чеклист ingest

1. Контролировать количество active parts в `system.parts`.  
2. Держать батчи в диапазоне 10k-100k, где возможно.  
3. Для high-frequency small writes включать async insert.  
4. Свести UPDATE/DELETE мутации к редким исключениям.  
5. Не автоматизировать `OPTIMIZE FINAL` в регулярных DAG/cron.

## Связи

- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse Query Optimization Patterns](clickhouse-query-optimization-patterns.md)
- [ClickHouse: индексы и background merges](clickhouse-indexes-and-background-merges.md)
- [ClickHouse MergeTree Engines](clickhouse-merge-tree-engines.md)
- [ClickHouse + Kafka Ingestion](clickhouse-kafka-ingestion.md)
