# PostgreSQL

**Тип:** реляционная СУБД (open source)

## В контексте вики

Часто упоминается в [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) как пример **реляционной модели**, индексов, транзакций и MVCC. Внутреннее устройство детально разобрано в [PostgreSQL 17 изнутри](../sources/postgresql-internals.md) (Егор Рогов): MVCC, буферный кеш, блокировки, планировщик запросов, типы индексов.

## Ключевые характеристики

| Характеристика | Суть |
|----------------|------|
| **MVCC** | Multi-Version Concurrency Control: каждая транзакция видит свой snapshot; writers не блокируют readers. Версии строк по xmin/xmax. |
| **WAL (Write-Ahead Log)** | Журнал для crash recovery и репликации. Все изменения — сначала в WAL, потом на диск. Основа для streaming replication и CDC (Debezium). |
| **Индексы** | B-tree (по умолчанию), GIN (full-text, JSON), GiST (geo), BRIN (block range для time-series), hash. |
| **Транзакции** | Полный ACID; уровни: Read Committed (default), Repeatable Read (через MVCC), Serializable (SSI). |
| **Расширяемость** | Extensions: PostGIS, pg_stat_statements, TimescaleDB, pgvector. Stored procedures (PL/pgSQL). Foreign data wrappers. |
| **Репликация** | Streaming replication (physical), logical replication (по таблицам), Patroni/pg_auto_failover для HA. |

## Типичные вопросы на интервью

**Q: Как работает MVCC в PostgreSQL?**
A: Каждая строка имеет `xmin` (ID транзакции, создавшей строку) и `xmax` (ID транзакции, удалившей/обновившей). При UPDATE создаётся **новая версия** строки (новый xmin), старая помечается xmax. Транзакция видит только строки, где `xmin < snapshot` и `xmax > snapshot` (или нет xmax). VACUUM очищает «мёртвые» версии.

**Q: Что такое VACUUM и зачем он нужен?**
A: Удаляет «мёртвые» tuple (старые версии строк после UPDATE/DELETE). Без VACUUM — table bloat (таблица растёт, не освобождая место). Autovacuum — фоновый процесс. VACUUM FULL — полная перезапись таблицы (тяжёлая операция, блокирует). Мониторить: `pg_stat_user_tables.n_dead_tup`.

**Q: Какие типы индексов есть и когда какой?**
A: **B-tree** — по умолчанию; range queries, equality, ORDER BY. **GIN** — для массивов, JSONB, full-text search. **GiST** — геопространственные, range types. **BRIN** — для больших таблиц с естественным порядком (time-series); компактный, но не точный. **Hash** — только equality (редко лучше B-tree).

**Q: Чем Read Committed отличается от Repeatable Read в PostgreSQL?**
A: **Read Committed** — каждый statement видит **последний committed** snapshot (может видеть разные данные в одной транзакции). **Repeatable Read** — транзакция видит snapshot **на момент начала** (consistent read). Repeatable Read может отклонить транзакцию при serialization failure (нужен retry).

**Q: Как PostgreSQL используется в контексте data engineering?**
A: Как **OLTP-источник** для CDC (Debezium читает WAL). Для небольших аналитических задач — с расширениями (TimescaleDB). Как backend для **metadata** (Airflow, dbt, Hive Metastore). Не подходит для тяжёлой OLAP-аналитики — для этого ClickHouse/BigQuery.

## Связи

- [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md) — внутреннее устройство: xmin/xmax, snapshots, VACUUM, HOT, буферный кеш.
- [SQL Transactions, Locking and Isolation](../concepts/sql-transactions-locking-standard.md) — блокировки, 2PL, deadlocks, уровни изоляции по стандарту.
- [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md) — план запроса, EXPLAIN, простые и составные индексы.
- [Normalization and Normal Forms](../concepts/normalization-and-normal-forms.md) — нормальные формы (1NF–5NF).
- [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md) — сравнение реализаций MVCC.
- [Data Models and Query Languages](../concepts/data-models-and-query-languages.md)
- [Storage and Retrieval](../concepts/storage-and-retrieval.md)
- [Transactions and Isolation](../concepts/transactions-and-isolation.md)
- [CDC: Debezium → Kafka → ClickHouse](../concepts/cdc-debezium-analytics-pipeline.md)