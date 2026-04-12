# ClickHouse + Kafka Ingestion

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

## Паттерн

```
Kafka Topic → Kafka Engine table → Materialized View → MergeTree (целевая таблица)
```

- **Kafka Engine** — виртуальный consumer: данные **не хранятся** для произвольных SELECT; батч читается и передаётся дальше.
- **Materialized View** в ClickHouse — **триггер на INSERT**: при появлении строк в источнике MV выполняет преобразование и пишет в **MergeTree** (или другой движок).

Так обеспечивается устойчивый поток **Kafka → колоночное хранилище** с контролем схемы и типов в MV.

## Типичные шаги в MV

- Приведение типов (например Unix timestamp float → `DateTime64`).
- Парсинг envelope (в CDC — JSON Debezium, см. [CDC pipeline](cdc-debezium-analytics-pipeline.md)).

## Инкрементальная агрегация

Цепочка **Fact (MergeTree) → Materialized Views → SummingMergeTree / AggregatingMergeTree**:

- MV срабатывает на **новый INSERT** в факт — стоимость **O(новые строки)** на батч.
- **SummingMergeTree** — аддитивные суммы при фоновых merge.
- **AggregatingMergeTree** — состояния `uniqState`, квантили и т.д. ([Star Schema](star-schema-and-grain.md) — additive vs non-additive).

## Интервью-формулировки

- Почему не читают напрямую из Kafka Engine? — таблица **не предназначена** для аналитических запросов по полному объёму.
- Что даёт MV поверх Kafka Engine? — **персистентная** MergeTree-таблица + явная трансформация.

## Связи

- [Stream Processing](stream-processing.md)
- [Replication](replication.md) (лог Kafka как реплицируемый журнал)
- [ClickHouse Schema Design Patterns](clickhouse-schema-design-patterns.md)
- [ClickHouse Insert and Mutation Strategy](clickhouse-insert-and-mutation-strategy.md)
