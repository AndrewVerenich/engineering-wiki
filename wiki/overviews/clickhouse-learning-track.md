# ClickHouse Learning Track (dbt-oriented)

Этот трек фокусируется на применении ClickHouse как аналитического хранилища в сочетании с dbt.

## Этап 1 — Core dbt на ClickHouse

- Разобрать структуру dbt-проекта (`models`, `seeds`, `snapshots`, `tests`).
- Понять DAG и `ref/source`.
- Освоить materializations: `view`, `table`, `incremental`, `ephemeral`.
- Практика: мини-проект `raw -> staging -> marts` на `dbt-clickhouse`.

## Этап 2 — Data modeling для витрин

- Layered architecture: `staging`, `intermediate`, `marts`.
- Naming conventions и явный grain.
- Surrogate keys.
- Практика: `fact_orders`, `dim_users`, `dim_products`, SCD2 через snapshots.

## Этап 3 — Тестирование и качество

- Built-in tests (`not_null`, `unique`, `relationships`, `accepted_values`).
- Custom tests и schema tests.
- Freshness checks и data contracts.

## Этап 4 — Incremental и производительность

- Incremental facts c `updated_at`-логикой.
- Merge/overwrite стратегии и backfill.
- Оптимизация сканов и отказ от частых full rebuild.
- Согласование dbt-моделей с физикой ClickHouse (ordering/partitioning/parts).

## Этап 5 — Jinja и макросы

- DRY-подход.
- Макросы surrogate key и audit-колонок.
- Reusable SQL-шаблоны.

## Этап 6 — Production

- CI/CD: `PR -> test -> deploy`.
- Окружения `dev/staging/prod`.
- State-based runs и slim CI.

## Связи

- [ClickHouse Schema Design Patterns](../concepts/clickhouse-schema-design-patterns.md)
- [ClickHouse Insert and Mutation Strategy](../concepts/clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](../concepts/clickhouse-query-optimization-patterns.md)
- [dbt Learning Roadmap](dbt-learning-roadmap.md)
