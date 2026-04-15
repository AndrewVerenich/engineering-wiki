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

## Типичные вопросы на интервью

**Q: Почему нельзя делать частые мелкие INSERT в ClickHouse?**
A: Каждый INSERT создаёт **part** на диске. Много мелких inserts → part explosion → деградация merge-процесса → рост latency, ошибки «too many parts». Решение: батчировать (10k-100k строк) или включить **async_insert** (сервер буферизует мелкие вставки).

**Q: Почему ALTER UPDATE/DELETE — плохая идея в ClickHouse?**
A: Мутации **переписывают** parts целиком → тяжёлый I/O, блокировка merge. ClickHouse — append-optimized движок, не предназначен для row-level updates. Альтернативы: **ReplacingMergeTree** (versioned upsert), **CollapsingMergeTree** (soft delete через sign), **DROP PARTITION** для массового удаления по времени.

**Q: Что такое async insert и когда его включать?**
A: Режим, когда сервер буферизует мелкие вставки и записывает их в более крупные parts. Включать, когда клиент не может батчировать (IoT-устройства, high-frequency events). `async_insert=1, wait_for_async_insert=1` — безопасный режим с подтверждением.

**Q: Что ты проверишь, если ingest деградирует?**
A: 1) `system.parts` — количество active parts (должно быть управляемым). 2) Размер батчей — не слишком мелкие? 3) Есть ли регулярные мутации (UPDATE/DELETE)? 4) Не запущен ли `OPTIMIZE FINAL` по cron? 5) Background merges — справляются ли (нет ли merge backlog)?

## Связи

- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse Query Optimization Patterns](clickhouse-query-optimization-patterns.md)
- [ClickHouse: индексы и background merges](clickhouse-indexes-and-background-merges.md)
- [ClickHouse MergeTree Engines](clickhouse-merge-tree-engines.md)
- [ClickHouse + Kafka Ingestion](clickhouse-kafka-ingestion.md)
