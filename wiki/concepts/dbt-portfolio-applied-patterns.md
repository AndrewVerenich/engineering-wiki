# dbt in Portfolio: Applied Patterns

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

## Типичные вопросы на интервью

**Q: Как ты организовал dbt-проект на практике?**
A: Четыре слоя: staging (1:1 с источником, dedup), intermediate (join'ы и бизнес-правила), core (facts + dimensions с surrogate keys), marts (domain-витрины). Запуск послойно по тегам с quality gate (`dbt test`) между слоями. Incremental facts с watermark для оптимизации.

**Q: Как ты обеспечиваешь качество данных в dbt?**
A: Generic tests (`not_null`, `unique`, `relationships`) на всех ключах и FK. Custom tests на domain-invariants. Freshness tests на источниках. Pipeline: «test before publish» — если tests fail, marts не обновляются. CI: slim/state-aware runs на PR.

**Q: Почему surrogate keys, а не natural keys, в dbt-проекте?**
A: 1) Стабильность join'ов при смене source PK. 2) Поддержка SCD Type 2 (каждая версия dimension — свой surrogate). 3) Integer join быстрее composite string. В dbt — макрос `surrogate_key` (обычно hash) для детерминированных ключей.

## Связи

- [dbt Learning Roadmap](../overviews/dbt-learning-roadmap.md)
- [dbt: слои проекта](dbt-project-layers.md)
- [Staging and Presentation Layers](staging-and-presentation-layers.md)
