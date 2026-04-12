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
