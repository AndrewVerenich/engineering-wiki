# dbt in Portfolio: Applied Patterns

**Источник:** [Личный план изучения dbt](../sources/dbt-learning-plan.md)

Страница фиксирует, как roadmap dbt был применен в portfolio-пайплайнах на практике.

## 1) Layered architecture как стандарт

Применено разделение:

- `staging` — очистка, типизация, дедуп;
- `intermediate` — обогащение и бизнес-правила;
- `dimensions/facts` — core-модель;
- `marts` — бизнес-витрины.

Это дает предсказуемый lineage и упрощает сопровождение.

## 2) Dimensional modeling в dbt

Применено:

- явное задание grain для фактов;
- `fact + dimensions` в core-слое;
- surrogate keys для стабильных join;
- SCD Type 2 (через snapshots и/или версионность измерений).

## 3) Quality gates

На критичных моделях покрытие тестами:

- `not_null`, `unique`, `relationships`, `accepted_values`;
- кастомные domain tests;
- freshness и checks на данных источника.

Практический эффект: сбои ловятся до BI-слоя и до потребителей метрик.

## 4) Incremental модели и backfill

Применено:

- incremental facts с `updated_at`/watermark;
- стратегия выборочного backfill по окнам;
- ограничение full rebuild только на исключительных изменениях.

Результат: дешевле и быстрее прод-запуски при росте объема данных.

## 5) Макросы и переиспользование

Использованы DRY-паттерны:

- макросы surrogate key;
- макросы audit-колонок;
- переиспользуемые фрагменты SQL для единообразия логики.

## 6) Оркестрация и production-flow

Применены практики:

- послойный запуск моделей по тегам;
- quality gate после построения слоев;
- pipeline-логика “test before publish”;
- подготовка к slim/state-aware CI.

## Что это дает на собеседовании

Можно уверенно объяснить:

1. почему слои в dbt = инженерная дисциплина, а не просто стиль;  
2. как grain/surrogate keys влияют на корректность метрик;  
3. как incremental + quality gates удерживают надежность при росте данных.

## Связи

- [dbt Learning Roadmap](../overviews/dbt-learning-roadmap.md)
- [dbt: слои проекта](dbt-project-layers.md)
- [Staging and Presentation Layers](staging-and-presentation-layers.md)
