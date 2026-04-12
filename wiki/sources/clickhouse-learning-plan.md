# Личный план изучения ClickHouse (через dbt)

| Поле | Значение |
|------|----------|
| **Тип** | личный план обучения |
| **Фокус** | ClickHouse как DWH backend + analytics engineering |
| **Контекст** | dbt Core, data modeling, production pipelines |

## Стратегия

План выстроен как трек **dbt + ClickHouse**:

- analytics engineering как дисциплина;
- production-grade data pipelines;
- data modeling поверх column-store (ClickHouse).

## Этапы (roadmap)

1. **Core dbt (1-2 недели):** архитектура dbt, DAG, materializations, refs/sources, Jinja+SQL.  
2. **Data Modeling (2-3 недели):** layered architecture, grain, surrogate keys, SCD2 snapshots.  
3. **Testing & Quality (1-2 недели):** built-in tests, custom tests, contracts, freshness.  
4. **Incremental & Performance (2 недели):** incremental logic, merge/overwrite, partitioning, backfill.  
5. **Jinja & Macros (2 недели):** DRY, генерация SQL, reusable macros.  
6. **Production (2-3 недели):** CI/CD, окружения, state-based runs, slim CI.

## Производные страницы

- [ClickHouse Learning Track (dbt-oriented)](../overviews/clickhouse-learning-track.md)
- [ClickHouse Official Documentation](clickhouse-official-documentation.md)
- [dbt Learning Roadmap](../overviews/dbt-learning-roadmap.md)
