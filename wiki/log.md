# Log

## [INIT] Wiki created

---

## [INGEST] Designing Data-Intensive Applications

- Страница источника: `wiki/sources/designing-data-intensive-applications.md`
- Добавлены концепты по главам 1–12 (см. index → Concepts)
- Сущности: PostgreSQL, Apache Kafka
- Обзор: Data Engineering Fundamentals
- Сравнение: OLTP vs OLAP
- Обновлены `index.md`, cross-links между страницами

---

## [INGEST] The Data Warehouse Toolkit (Kimball) — dimensional modeling

- Источник: `wiki/sources/the-data-warehouse-toolkit.md` (книга: *The Data Warehouse Toolkit*, Kimball & Ross)
- Концепты: dimensional modeling hub, star schema & grain, dimension tables, fact table types, SCD, conformed dimensions, advanced patterns
- Обзор для собеседований: `wiki/overviews/dimensional-modeling-interview-prep.md`
- Сравнение: `wiki/comparisons/kimball-vs-inmon.md`
- Обновлены `index.md`, `data-engineering-fundamentals.md`, `oltp-vs-olap.md`, cross-links

---

## [UPDATE] Два слоя DW (staging / presentation)

- Концепт: `wiki/concepts/staging-and-presentation-layers.md`
- Обновлены: `dimensional-modeling-kimball.md`, источник Kimball, interview prep, `data-engineering-fundamentals.md`, `index.md`

---

## [UPDATE] Удалены источники по отдельным кодовым проектам

- Страницы-источники и обзор, привязанные к конкретным репозиториям, убраны; в вики остались концепты и сравнения без ссылок на проекты (`elt-vs-etl-practice` переписан нейтрально).

---

## [INGEST] ClickHouse Official Documentation

- Источник: `wiki/sources/clickhouse-official-documentation.md`
- Добавлены концепты: schema design patterns, insert/mutation strategy, query optimization patterns
- Обновлены `clickhouse-kafka-ingestion.md`, `index.md`, `data-engineering-fundamentals.md`
- Ingest выполнен на основе best-practices rules (schema / query / insert)

---

## [UPDATE] ClickHouse internals: indexes, merges, engines

- Добавлены детальные концепты:
  - `wiki/concepts/clickhouse-indexes-and-background-merges.md`
  - `wiki/concepts/clickhouse-under-the-hood.md`
  - `wiki/concepts/clickhouse-merge-tree-engines.md`
- Обновлены cross-links в `clickhouse-official-documentation.md`, `index.md`, `data-engineering-fundamentals.md`, query/insert концептах

---

## [INGEST] dbt learning plan + portfolio application

- Источник: `wiki/sources/dbt-learning-plan.md` (личный roadmap по этапам 1-6)
- Обзор: `wiki/overviews/dbt-learning-roadmap.md`
- Концепт применения: `wiki/concepts/dbt-portfolio-applied-patterns.md`
- Обновлен `dbt-project-layers.md` секцией “Как применено в портфолио”
- Обновлены `index.md` и `data-engineering-fundamentals.md`

---

## [INGEST] ClickHouse learning plan (dbt-oriented)

- Источник: `wiki/sources/clickhouse-learning-plan.md`
- Обзор: `wiki/overviews/clickhouse-learning-track.md`
- Обновлены `index.md`, `data-engineering-fundamentals.md`, `clickhouse-official-documentation.md`
- Зафиксирован отдельный learning track для ClickHouse как DWH backend в связке с dbt

---

## [UPDATE] Debezium section expanded

- Существенно расширен `wiki/concepts/cdc-debezium-analytics-pipeline.md`
- Добавлены: envelope структура, snapshot vs streaming, ordering/idempotency, deletes/tombstones, schema evolution, anti-patterns, operational checklist

---

## [INGEST] Реляционные базы данных в примерах (Куликов)

- Источник: `wiki/sources/relational-databases-in-examples.md`
- Концепт: `wiki/concepts/normalization-and-normal-forms.md` — нормальные формы (1NF–5NF), функциональные зависимости, денормализация
- Концепт: `wiki/concepts/sql-query-execution-and-indexes.md` — pipeline выполнения, cost-based optimizer, EXPLAIN, простые/составные/покрывающие/частичные/functional индексы, leftmost prefix, стратегии JOIN
- Обновлены: `storage-and-retrieval.md`, `data-models-and-query-languages.md`, `postgresql.md` (entity), `data-engineering-fundamentals.md`, `index.md`

---

## [INGEST] SQL: The Complete Reference (Groff, Weinberg, Oppel)

- Источник: `wiki/sources/sql-complete-reference.md`
- Концепт: `wiki/concepts/sql-transactions-locking-standard.md` — транзакционная модель SQL, блокировки (S/X/intent), гранулярность, lock escalation, 2PL, deadlocks, уровни изоляции по стандарту, сравнение lock-based vs MVCC, реализации в PostgreSQL/MySQL/SQL Server/Oracle
- Обновлены: `transactions-and-isolation.md`, `postgresql-mvcc-internals.md`, `postgresql.md` (entity), `data-engineering-fundamentals.md`, `index.md`

---

## [INGEST] PostgreSQL 17 изнутри (Егор Рогов)

- Источник: `wiki/sources/postgresql-internals.md`
- Концепт: `wiki/concepts/postgresql-mvcc-internals.md` — MVCC (xmin/xmax, snapshots, VACUUM, HOT, freeze, блокировки строк, буферный кеш, WAL, выполнение запросов, типы индексов)
- Сравнение: `wiki/comparisons/postgresql-vs-mysql-mvcc.md` — PostgreSQL vs MySQL InnoDB (heap vs undo log, bloat, UPDATE cost, Serializable, clustered index)
- Обновлены: `postgresql.md` (entity), `transactions-and-isolation.md`, `data-engineering-fundamentals.md`, `index.md`

---

## [CLEANUP] Удалены личные планы обучения из sources

- Удалены: `wiki/sources/clickhouse-learning-plan.md`, `wiki/sources/dbt-learning-plan.md`
- Причина: личные планы — не источники (книги, статьи, документация); sources должен содержать только внешние материалы
- Почищены ссылки в `index.md`, `data-engineering-fundamentals.md`, `clickhouse-learning-track.md`, `dbt-learning-roadmap.md`, `dbt-portfolio-applied-patterns.md`

---

## [UPDATE] Interview Q&A enrichment across all existing topics

- Добавлены секции «Типичные вопросы на интервью» (Q&A) во все существующие concept-страницы, entity-страницы и comparison-страницы
- Тонкие DDIA-концепты расширены: ключевые термины, таблицы сравнения, trade-offs (reliability, data-models, storage, encoding, replication, partitioning, transactions, distributed-pitfalls, consistency, batch, stream, derived-data)
- Kimball-концепты дополнены interview Q&A (dimensional-modeling, staging-layers, star-schema, dimension-tables, fact-table-types, conformed-dimensions, advanced-patterns)
- ClickHouse-концепты дополнены interview Q&A (schema-design, insert-mutation, query-optimization, indexes-merges, under-the-hood, merge-tree-engines, cdc-debezium)
- Entity-страницы (postgresql, apache-kafka) существенно расширены: характеристики, термины, interview Q&A
- Comparison-страницы дополнены interview Q&A (oltp-vs-olap, kimball-vs-inmon, elt-vs-etl)
- dbt и Spark concepts дополнены interview Q&A

---
