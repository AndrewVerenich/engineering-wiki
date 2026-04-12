# dbt Learning Roadmap

**Источник:** [Личный план изучения dbt](../sources/dbt-learning-plan.md)

## Этап 1 — Core dbt

**Цель:** понимать, как dbt строит DAG и компилирует SQL.

- Структура проекта: `models`, `seeds`, `snapshots`, `tests`.
- Зависимости: `ref`, `source`, граф моделей.
- Materializations: `view`, `table`, `incremental`, `ephemeral`.
- Jinja + SQL как базовая техника переиспользования.

**Выход этапа:** мини-проект `raw -> staging -> marts`.

## Этап 2 — Modeling в dbt

**Цель:** перенести dimensional modeling в практику dbt.

- Слои: `staging`, `intermediate`, `marts`.
- Naming conventions и единый стиль моделей.
- Явная фиксация grain для каждой факт-таблицы.
- Surrogate keys и стабильные ключи join.
- SCD Type 2 через snapshots.

## Этап 3 — Quality framework

**Цель:** сделать dbt инструментом контроля данных, а не только трансформаций.

- Built-in tests: `not_null`, `unique`, `relationships`, `accepted_values`.
- Custom tests и domain checks.
- Freshness checks.
- Data contracts на критичных моделях.

## Этап 4 — Incremental и performance

**Цель:** перейти от full rebuild к экономным прод-запускам.

- Incremental facts с `updated_at`/watermark-логикой.
- Стратегии backfill.
- Минимизация сканов и стоимости пересборки.
- Согласование с физикой хранилища (partitioning/ordering).

## Этап 5 — Макросы и DRY

**Цель:** убрать дублирование и повысить поддерживаемость.

- Макросы surrogate keys.
- Макросы audit-колонок (`loaded_at`, `updated_at`, `run_id`).
- Переиспользуемые SQL-шаблоны и генерация выражений.

## Этап 6 — Production

**Цель:** воспроизводимый lifecycle “PR -> test -> deploy”.

- Окружения: `dev / staging / prod`.
- CI/CD для dbt.
- State-aware запуски и slim CI.
- Защитные quality gates перед deploy.

## Связи

- [dbt in Portfolio: Applied Patterns](../concepts/dbt-portfolio-applied-patterns.md)
- [dbt: слои проекта](../concepts/dbt-project-layers.md)
- [ELT vs ETL (на практике)](../comparisons/elt-vs-etl-practice.md)
