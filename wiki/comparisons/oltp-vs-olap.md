# OLTP vs OLAP

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (в т.ч. контраст рабочих и аналитических нагрузок)

## Суть

| | **OLTP** | **OLAP** |
|---|----------|----------|
| Назначение | Операционные транзакции (заказы, платежи) | Аналитика, отчёты, агрегации |
| Запросы | Много мелких, точечных | Редкие тяжёлые, сканирующие |
| Модель данных | Нормализованная (часто) | Звезда/снежинка, колоночные хранилища |
| Задержка | Низкая для одной операции | Выше, допустим batch |

Обе перспективы связаны с [Batch Processing](../concepts/batch-processing.md) (часто OLAP-данные наполняются пакетно) и [Derived Data and Systems](../concepts/derived-data-and-systems.md).

Витрины и OLAP часто строятся по **dimensional modeling** (Kimball): [Dimensional Modeling (Kimball)](../concepts/dimensional-modeling-kimball.md).

## Типичные вопросы на интервью

**Q: В чём ключевая разница между OLTP и OLAP?**
A: **OLTP** — много мелких транзакций (INSERT/UPDATE/SELECT по PK), low latency, нормализованная модель, row-based storage. **OLAP** — редкие тяжёлые запросы (агрегации, сканы), columnar storage, денормализованная модель (star schema). OLTP — для приложений, OLAP — для аналитики.

**Q: Можно ли использовать PostgreSQL для OLAP?**
A: Для небольших объёмов — да (с расширениями типа TimescaleDB, columnar extensions). Для серьёзной аналитики — нет: row-based storage, нет vectorized execution, нет columnar compression. Лучше: ClickHouse, BigQuery, Snowflake, DuckDB (для embedded).

**Q: Как данные попадают из OLTP в OLAP?**
A: 1) **ETL/ELT batch** — периодическая выгрузка (pg_dump, SELECT + INSERT). 2) **CDC** — Debezium читает WAL и публикует изменения в Kafka → OLAP. 3) **Dual write** — приложение пишет в оба (анти-паттерн: consistency проблемы). Рекомендуется: CDC или batch с idempotency.

## Связи

- [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md)
- [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md)