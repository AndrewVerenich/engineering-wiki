# Apache Flink

**Тип:** распределённый stream processing engine (true streaming, stateful)

## В контексте вики

В книге [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) Flink разбирается как референсная реализация stateful stream processing: настоящий event-at-a-time engine с event-time semantics, watermarks, managed state и end-to-end exactly-once через checkpointing + transactional sinks. Концепты, для которых Flink — пример: [Stream Processing](../concepts/stream-processing.md), [Apache Flink Architecture](../concepts/flink-architecture.md), [Flink Time and Watermarks](../concepts/flink-time-and-watermarks.md).

## Ключевые характеристики

| Характеристика | Суть |
|----------------|------|
| **True streaming** | Обработка **событие-за-событием**, не micro-batch. Latency измеряется миллисекундами, а не секундами как у Spark Structured Streaming. |
| **Stateful** | State — first-class citizen. Каждый оператор может держать managed state, который Flink сам шардирует по ключу, восстанавливает при сбое и масштабирует при изменении parallelism. |
| **Event time + watermarks** | Семантика времени по timestamp в самих событиях, а не по wall clock. Watermarks отделяют «уже всё пришло» от «ещё ждём». Корректность при out-of-order и late events. |
| **Exactly-once** | End-to-end через **distributed snapshots** (Chandy-Lamport) + transactional sinks (Kafka, файлы, JDBC через 2PC). |
| **DataStream API** | Низкоуровневый API на Java/Scala/Kotlin: map/filter/keyBy/window/process. Поверх него — **Table API / SQL** (декларативный). |
| **JobManager + TaskManager** | JobManager — координатор (scheduling, checkpoints). TaskManager — worker с **task slots** (единицей parallelism является slot, не CPU core). |
| **State backends** | Где хранится state: **HashMap** (in-memory, heap) для маленьких state, **RocksDB** (on-disk, embedded LSM-tree) для огромных state — десятки ГБ на оператор. |
| **Savepoints** | Версионируемый, ручной checkpoint. Используется для upgrade приложения, миграции state, A/B-релизов, переноса между кластерами. |

## Типичные вопросы на интервью

**Q: В чём принципиальное отличие Flink от Spark Structured Streaming?**
A: Flink — **true streaming**: каждое событие проходит через граф операторов сразу. Spark Structured Streaming — **micro-batch**: события буферизуются в маленькие батчи (типично секунда+) и обрабатываются как мини-job. Следствия: у Flink ниже latency и нативно event time + watermarks; у Spark проще unified batch+stream и API богаче для аналитики. Continuous Processing mode у Spark существует, но experimental и без exactly-once.

**Q: Что такое task slot и зачем он нужен?**
A: Task slot — единица **резервирования ресурсов** на TaskManager. Один slot держит **по одному параллельному инстансу каждого оператора** в pipeline (slot sharing). Если у job parallelism=4, нужно 4 slot'а, но они могут быть распределены по разному количеству TaskManager'ов. Slot изолирует память (managed memory), но **не CPU**. Это дешевле, чем выделять JVM на каждый таск.

**Q: Как Flink обеспечивает exactly-once при сбое?**
A: Через **distributed snapshots** (вариант Chandy-Lamport): JobManager периодически инжектит **checkpoint barrier** в потоки источников. Каждый оператор, увидев barrier на всех входах, делает snapshot своего state и пропускает barrier дальше. Когда checkpoint завершён глобально — он durable. При сбое: весь job рестартуется, state восстанавливается из последнего успешного checkpoint, источники откатываются на сохранённые offset'ы. Для **end-to-end** exactly-once sink должен быть либо идемпотентным, либо транзакционным (двухфазный commit, согласованный с checkpoint).

**Q: Чем отличается keyed state от operator state?**
A: **Keyed state** — привязан к ключу после `keyBy`. У каждого ключа свой инстанс ValueState/ListState/MapState/ReducingState/AggregatingState. Шардируется по `hash(key) % parallelism` через **key groups** — это даёт rescaling. **Operator state** — общий для параллельной задачи (например, offset Kafka consumer'а). Меньше распространён, чаще встречается у source/sink connector'ов.

**Q: Когда использовать RocksDB state backend, а когда HashMap?**
A: **HashMap** — если state помещается в heap (единицы ГБ), нужна минимальная latency и checkpoint'ы быстрые. **RocksDB** — если state большой (десятки ГБ — десятки ТБ), допустима небольшая latency-цена за чтение из off-heap LSM-tree, нужны **incremental checkpoints** (RocksDB пишет только новые SST-файлы между checkpoint'ами). RocksDB — стандарт для production-job'ов с большим state.

**Q: Что такое savepoint и чем отличается от checkpoint?**
A: **Checkpoint** — автоматический, периодический, ради fault tolerance. Управляется Flink, может быть incremental, удаляется при retention. **Savepoint** — ручной trigger через CLI/REST. Полный, переносимый snapshot. Используется для **осознанных операций**: апгрейд версии Flink, изменение topology, изменение parallelism, миграция между кластерами. Формат стабильный между версиями Flink.

**Q: Что такое watermark и какой trade-off у его задержки?**
A: Watermark `W(t)` — обещание: «больше событий с event time ≤ t не придёт» (точнее — «их можно игнорировать»). Окна закрываются, когда watermark проходит их конец. **Слишком ранний watermark** (низкая `outOfOrdernessBound`) → late events отбрасываются → потеря данных. **Слишком поздний** → окна закрываются с большой задержкой → высокая latency результата. Стандартный паттерн: `BoundedOutOfOrderness(Duration.ofSeconds(N))` где N выбирается по 99-95 перцентилю реального jitter событий.

**Q: Как Flink масштабируется (rescaling) без потери state?**
A: Keyed state хранится в **key groups** — фиксированное количество логических партиций (по умолчанию 128). При rescaling key groups переназначаются между параллельными инстансами оператора. Сам state перебалансируется при загрузке checkpoint'а или savepoint'а. Максимальный parallelism ограничен количеством key groups (`maxParallelism`), который **нельзя изменить после старта job** — выбирается заранее с запасом.

## Связи

- [Stream Processing](../concepts/stream-processing.md) — теория из DDIA, контекст для Flink.
- [Apache Flink Architecture](../concepts/flink-architecture.md) — JobManager, TaskManager, slots.
- [Flink State Management](../concepts/flink-state-management.md) — keyed/operator/broadcast state, backends.
- [Flink Checkpoints and Savepoints](../concepts/flink-checkpoints-and-savepoints.md) — fault tolerance.
- [Flink vs Spark Structured Streaming](../comparisons/flink-vs-spark-structured-streaming.md) — сравнение движков.
- [Apache Kafka](apache-kafka.md) — основной транспорт source/sink для Flink.
