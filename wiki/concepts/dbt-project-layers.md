# dbt: слои проекта

## Типичная раскладка

| Слой | Роль | Примеры |
|------|------|---------|
| **staging** | 1:1 с источником, переименования, типы, **дедуп CDC** | `stg_customers` |
| **intermediate** | join’ы, бизнес-правила, подготовка к dims/facts | `int_transactions_enriched` |
| **dimensions** | измерения звезды, **surrogate keys** | `dim_customer` |
| **facts** | факты, grain, FK на измерения | `fct_transaction` |
| **marts** | витрины для доменов/метрик | `mart_daily_revenue` |

Связь с [Dimensional Modeling (Kimball)](dimensional-modeling-kimball.md): presentation-слой выражен в **dimensions / facts / marts**.

## Теги и оркестрация

Прогон по тегам (`tag:staging`, …) позволяет Airflow/DAG выполнять слои **последовательно** и вешать **quality gate** (`dbt test`) после marts.

## Тесты

`not_null`, `unique`, `relationships`, `accepted_values`, кастомные generic-тесты — часть контракта качества между слоями.

## Как применено в портфолио

- Слои запускались последовательно (stage -> intermediate -> core -> marts) с quality gate после построения.
- В `staging` закреплялись типизация и dedup-логика для “последней версии” записи.
- В core-слое фиксировались grain фактов и surrogate keys для устойчивых join.
- В marts выносились бизнес-метрики и domain-oriented витрины.

Детализация обучения и применения:

- [dbt Learning Roadmap](../overviews/dbt-learning-roadmap.md)
- [dbt in Portfolio: Applied Patterns](dbt-portfolio-applied-patterns.md)

## Связи

- [Conformed Dimensions](conformed-dimensions.md) — согласование измерений между marts
