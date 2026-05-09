# Kafka Streams: KStream, KTable, GlobalKTable

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Основная модель Kafka Streams строится на различии между **потоком событий** (`KStream`) и **таблицей текущего состояния** (`KTable`). Понимание этой duality критично: один и тот же Kafka topic можно интерпретировать как event log или как changelog таблицы.

## Типы и их поведение

| Тип | Семантика | Когда использовать |
|-----|-----------|-------------------|
| **KStream** | Каждая запись — отдельный факт, append-only обработка. | Event-driven пайплайны, audit, enrichment потоков. |
| **KTable** | Upsert/DELETE (через tombstone) по ключу, хранит последнее значение. | Materialized state, агрегаты, stream-table joins. |
| **GlobalKTable** | Полная копия таблицы на каждом инстансе. | Небольшие reference-справочники без co-partitioning. |

## Stream-table duality

- Topic как `KStream`: видим все изменения по времени.
- Тот же topic как `KTable`: видим только текущее состояние по ключам.
- Changelog таблицы можно обратно развернуть в stream изменений.
- Tombstone (`key + null`) в `KTable` означает удаление записи.

## Практические trade-offs

- `KTable` требует state store и changelog, поэтому имеет стоимость по диску/восстановлению.
- `GlobalKTable` упрощает join, но дублирует данные на каждый инстанс.
- Неверный выбор типа приводит к логическим багам (например, double-counting при агрегатах на `KStream` без дедупликации).

## Типичные вопросы на интервью

**Q: Почему нельзя всегда использовать только KStream?**  
A: Потому что многие задачи требуют текущего состояния по ключу (lookup, upsert, compact view). На `KStream` это придётся строить вручную через state stores.

**Q: Когда GlobalKTable лучше KTable?**  
A: Когда reference-данные маленькие и часто нужны join с потоками, где co-partitioning неудобен или невозможен.

**Q: Что значит tombstone в KTable?**  
A: Это запись с `null`-value, которая удаляет ключ из materialized state и в compacted topic.

**Q: Какие риски у GlobalKTable?**  
A: Рост памяти/диска на каждом инстансе, более долгий старт/restore и повышенная нагрузка при обновлении справочника.

## Связи

- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md) — физическое хранение KTable state.
- [Kafka Streams Joins](kafka-streams-joins.md) — выбор join-модели для KStream/KTable/GlobalKTable.
- [Stream Processing](stream-processing.md) — фундаментальная stream-table duality.
- [Kafka Streams Architecture](kafka-streams-architecture.md) — как типы встраиваются в topology.
