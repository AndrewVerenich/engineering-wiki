# CDC: Debezium → Kafka → ClickHouse

## Контур

```
OLTP (WAL/binlog) -> Debezium connector -> Kafka topics -> ClickHouse landing -> dbt -> marts
```

Debezium читает журнал транзакций БД и публикует изменения как события в Kafka, без тяжелых full-poll запросов к источнику.

## Что Debezium дает в аналитической архитектуре

- **Change stream как источник истины изменений**, а не периодический dump.
- **Почти real-time ingestion** при сохранении истории изменений.
- **Развязка OLTP и DWH**: аналитика не давит транзакционную БД.
- **Replay-возможность** через Kafka retention (в пределах политики хранения).

## Как устроено событие Debezium (envelope)

Типично приходят поля:

- `before` — состояние строки до изменения;
- `after` — состояние после изменения;
- `op` — тип операции (`c` create, `u` update, `d` delete, `r` snapshot-read);
- `ts_ms` — время события;
- `source` — метаданные источника (db/schema/table/LSN и т.п.).

Для аналитики обычно:

1. парсят envelope в landing-слой;
2. сохраняют полезные метаданные (`op`, source timestamp, ingestion timestamp);
3. решают delete-семантику (soft/hard, tombstone handling).

## Snapshot vs Streaming фаза

Debezium часто работает в двух фазах:

1. **Initial snapshot** — начальная выгрузка текущего состояния таблиц (`op='r'`).
2. **Streaming** — дальнейшие изменения из WAL/binlog (`c/u/d`).

Практически важно не смешать эти фазы при дедупе и не задвоить данные при переинициализации.

## Ключи, порядок, exactly-once ожидания

- Kafka гарантирует порядок **внутри partition**, не глобально.
- Для одной строки корректность держится на стабильном **ключе сообщения** (обычно PK источника).
- Debezium + Kafka + ClickHouse в реальности чаще дают **at-least-once ingestion**, поэтому в DWH нужен идемпотентный слой.

Идемпотентность обычно делается в staging:

`row_number() over (partition by business_key order by source_ts desc, ingestion_ts desc) = 1`.

## Deletes и tombstones

При delete обычно приходит:

- событие `op='d'` с `before` (или ключом);
- опционально tombstone-сообщение для компакции Kafka.

В аналитике нужно заранее решить политику:

- **Hard delete** в витринах;
- **Soft delete** флагом `is_deleted`;
- хранение истории удалений в audit-слое.

## Schema evolution и совместимость

Изменения колонок в OLTP неизбежны: add/drop/rename/type change.

Надежная схема:

1. landing-слой допускает эволюцию;
2. staging нормализует контракт полей;
3. downstream модели (dims/facts) меняются контролируемо и тестируются.

Связь: [Encoding and Schema Evolution](encoding-and-schema-evolution.md).

## Типичные анти-паттерны

- Считать CDC поток “exactly once by default”.
- Игнорировать `op` и merge-ить `before/after` как попало.
- Отсутствие dedup-логики в staging.
- Отсутствие стратегии для deletes и late/out-of-order events.
- Full refresh витрин без контроля replay-окна.

## Operational checklist

1. Мониторить lag коннектора и lag потребителей Kafka.  
2. Контролировать DDL изменения источника и контракты полей.  
3. Хранить source метаданные в landing (операция, ts, ключ).  
4. Иметь runbook: restart connector, replay диапазона, backfill.  
5. Покрыть staging тестами на uniqueness/freshness/nullability.

## Связи

- [ClickHouse + Kafka Ingestion](clickhouse-kafka-ingestion.md)
- [dbt: слои проекта](dbt-project-layers.md)
- [Staging and Presentation Layers](staging-and-presentation-layers.md)
- [Encoding and Schema Evolution](encoding-and-schema-evolution.md)
