# ELT vs ETL (на практике)

## Суть

| | **ELT** | **ETL / transform вне DWH** |
|---|---------|------------------------------|
| Где тяжёлая логика | В **аналитическом хранилище** (SQL, например **dbt** поверх ClickHouse / Snowflake / BigQuery) | В **Spark**, Flink, внешних сервисах **до** загрузки в хранилище |
| Типичный сценарий | CDC или batch уже **в OLAP**; модель и витрины — SQL в репозитории | Сырые файлы в **data lake** (HDFS, S3), тяжёлые join’ы и очистка в кластере, затем load |

Оба подхода часто **сочетаются**: распределённая обработка готовит широкие Parquet-таблицы, а **dbt** задаёт семантику звезды и витрин уже внутри хранилища.

## Связи

- [dbt Project Layers](../concepts/dbt-project-layers.md)
- [Spark Batch + Medallion](../concepts/spark-batch-medallion-hdfs.md)
- [Staging and Presentation Layers](../concepts/staging-and-presentation-layers.md)
