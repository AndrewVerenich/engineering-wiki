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

## Связи

- [Batch Processing](batch-processing.md)
- [dbt Project Layers](dbt-project-layers.md)
