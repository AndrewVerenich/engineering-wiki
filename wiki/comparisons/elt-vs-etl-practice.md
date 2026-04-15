# ELT vs ETL (на практике)

## Суть

| | **ELT** | **ETL / transform вне DWH** |
|---|---------|------------------------------|
| Где тяжёлая логика | В **аналитическом хранилище** (SQL, например **dbt** поверх ClickHouse / Snowflake / BigQuery) | В **Spark**, Flink, внешних сервисах **до** загрузки в хранилище |
| Типичный сценарий | CDC или batch уже **в OLAP**; модель и витрины — SQL в репозитории | Сырые файлы в **data lake** (HDFS, S3), тяжёлые join’ы и очистка в кластере, затем load |

Оба подхода часто **сочетаются**: распределённая обработка готовит широкие Parquet-таблицы, а **dbt** задаёт семантику звезды и витрин уже внутри хранилища.

## Типичные вопросы на интервью

**Q: В чём разница между ETL и ELT?**
A: **ETL**: Extract → Transform (вне DWH, например Spark) → Load в DWH. **ELT**: Extract → Load (raw) в DWH → Transform внутри DWH (SQL, dbt). Ключевое: **где** происходит тяжёлая трансформация — снаружи или внутри хранилища.

**Q: Почему ELT стал популярнее?**
A: 1) Облачные DWH (BigQuery, Snowflake, ClickHouse) достаточно мощные для трансформаций. 2) SQL-first подход (dbt) — понятнее аналитикам, version control, тесты. 3) Raw данные сохраняются в DWH — можно перестроить витрины без повторной выгрузки из источника. 4) Меньше инфраструктуры (не нужен Spark-кластер для SQL-трансформаций).

**Q: Когда всё-таки нужен ETL (трансформация снаружи)?**
A: 1) Тяжёлые join'ы на **сырых файлах** (HDFS/S3) — Spark лучше. 2) Semi-structured/unstructured data (парсинг логов, ML feature engineering). 3) Данные слишком большие для загрузки в DWH целиком. 4) Complex transformations, не выразимые в SQL. На практике ETL + ELT часто **сочетаются**: Spark (bronze → silver) + dbt (silver → gold).

## Связи

- [dbt Project Layers](../concepts/dbt-project-layers.md)
- [Spark Batch + Medallion](../concepts/spark-batch-medallion-hdfs.md)
- [Staging and Presentation Layers](../concepts/staging-and-presentation-layers.md)
