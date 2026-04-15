# Spark Batch + Medallion (HDFS)

## Идея

Сырьё в **HDFS** (Parquet), обработка **Spark** по этапам:

- **Bronze** — нормализация сырых логов, базовая очистка.
- **Silver** — дедуп, join к справочникам, обогащение.
- **Load to ClickHouse** — загрузка raw-слоя в OLAP (JDBC / batch insert), часто с **`ingest_batch_id`** как партиция для **идемпотентности** (`DROP PARTITION` перед повторной загрузкой батча).

Это вариант **medallion** / zone-архитектуры: от «как пришло» к «как анализируем», дальше — **dbt** в ClickHouse для звезды и витрин.

## Где Spark vs dbt

- **Spark:** тяжёлые распределённые преобразования на файлах в HDFS, join больших объёмов.
- **dbt (в ClickHouse):** SQL-трансформации внутри OLAP, витрины, тесты — [ELT vs ETL](../comparisons/elt-vs-etl-practice.md).

## Типичные вопросы на интервью

**Q: Что такое medallion architecture?**
A: Послойная организация данных: **Bronze** (сырые, «как пришло»), **Silver** (очищенные, дедуплицированные, обогащённые), **Gold** (бизнес-витрины для BI). Каждый слой — отдельный уровень качества и ответственности. Это эволюция staging/presentation Kimball для data lake.

**Q: Когда Spark, а когда dbt?**
A: **Spark** — для тяжёлых распределённых преобразований на файлах (HDFS/S3): большие join'ы, ML, semi-structured data. **dbt** — для SQL-трансформаций **внутри** DWH (ClickHouse, Snowflake). Часто комбинируются: Spark готовит raw → ClickHouse, dbt строит витрины внутри ClickHouse.

**Q: Как обеспечить идемпотентность при загрузке из Spark в ClickHouse?**
A: Паттерн: `ingest_batch_id` как partition key → `DROP PARTITION(batch_id)` перед повторной загрузкой. Альтернатива: ReplacingMergeTree с version column. Ключевое: повторная загрузка не должна создавать дублей.

## Связи

- [Batch Processing](batch-processing.md)
- [dbt Project Layers](dbt-project-layers.md)
