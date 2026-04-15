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

## Типичные вопросы на интервью

**Q: Зачем разделять dbt-проект на слои?**
A: 1) **Разделение ответственности**: staging — контракт с источником, intermediate — бизнес-логика, marts — бизнес-витрины. 2) **Тестируемость**: quality gate между слоями. 3) **Lineage**: понятно, откуда данные. 4) **Переиспользование**: dimension доступна нескольким marts. Без слоёв — спагетти-SQL, невоспроизводимость, хрупкие витрины.

**Q: Что такое staging-модели в dbt?**
A: 1:1 с источником. Задачи: переименования, типизация, дедуп (для CDC — `row_number()` по PK), фиксация контракта полей. Staging — **не** место для бизнес-логики. Если источник меняет схему — правим staging, downstream не ломается.

**Q: Как устроены тесты в dbt?**
A: Generic tests: `not_null`, `unique`, `relationships`, `accepted_values` — в schema.yml. Custom tests — SQL, возвращающий «плохие» строки. Freshness tests — на источниках (последнее обновление). Запускаются как quality gate: `dbt test --select tag:marts` после `dbt run`.

**Q: Что такое incremental модель и когда её использовать?**
A: Модель, которая обрабатывает только **новые/изменённые** строки (по `updated_at` или watermark). Экономит время и ресурсы на больших таблицах. Когда использовать: facts и таблицы с большим объёмом, где full rebuild дорогой. Когда НЕ использовать: маленькие dimensions, сложная логика с зависимостями от всех данных.

## Связи

- [Conformed Dimensions](conformed-dimensions.md) — согласование измерений между marts
