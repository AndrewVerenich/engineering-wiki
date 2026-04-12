# Личный план изучения dbt (roadmap)

| Поле | Значение |
|------|----------|
| **Тип** | личный план обучения |
| **Фокус** | analytics engineering + production-grade pipelines |
| **Контекст** | dbt Core, ClickHouse, применение в портфолио |

## Цель плана

Изучить dbt не как “SQL-утилиту”, а как каркас для:

- layered data modeling;
- контроля качества данных;
- воспроизводимых production-пайплайнов;
- CI/CD-процесса для аналитики.

## Этапы плана

1. **Core dbt (1-2 недели):** архитектура, DAG, materializations, refs/sources, Jinja+SQL.  
2. **Data Modeling (2-3 недели):** layered architecture, grain, surrogate keys, SCD2 через snapshots.  
3. **Testing & Quality (1-2 недели):** built-in tests, custom tests, freshness, contracts.  
4. **Incremental & Performance (2 недели):** incremental models, merge/overwrite стратегии, backfill.  
5. **Jinja & Macros (2 недели):** DRY, reusable SQL, макросы ключей/аудита.  
6. **Production (2-3 недели):** CI/CD, окружения, state/slim CI.

## Производные страницы

- [dbt Learning Roadmap](../overviews/dbt-learning-roadmap.md)
- [dbt in Portfolio: Applied Patterns](../concepts/dbt-portfolio-applied-patterns.md)
- [dbt: слои проекта](../concepts/dbt-project-layers.md)
