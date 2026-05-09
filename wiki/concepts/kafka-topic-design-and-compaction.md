# Kafka Topic Design and Compaction

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Топик-дизайн в Kafka — это инженерный выбор между параллелизмом, ordering, storage cost и replay-потребностями. Ошибки на этом уровне (неправильный key, число partitions, compaction policy) дорого исправлять в production.

## Выбор числа partitions

| Фактор | Влияние |
|--------|---------|
| Throughput | больше partitions -> выше параллелизм |
| Ordering | порядок гарантирован только в пределах одной partition |
| Rebalance cost | больше partitions -> дольше rebalance и ops overhead |
| Future scaling | увеличивать partitions можно, но это меняет key mapping |

## Retention vs compaction

| Модель | Для чего | Особенности |
|--------|----------|-------------|
| Time/size retention | event history/replay window | старые данные удаляются целыми сегментами |
| Log compaction | latest state by key | сохраняются последние значения и tombstones |

## Key design

- Стабильный business key нужен для ordering per entity.
- Плохой key ведет к hot partition и skew.
- Изменение числа partitions меняет hash-mapping, поэтому порядок по ключу в старых/новых данных может разойтись.

## Tombstones и cleaner

Compacted topics используют tombstone records для удаления ключа. Cleaner работает асинхронно, поэтому удаление не мгновенное; это важно при построении stateful consumers.

Kafka Streams активно использует compacted internal topics как changelog для state stores, поэтому политика compaction/retention напрямую влияет на скорость восстановления state после failover.

## Типичные вопросы на интервью

**Q: Почему нельзя бездумно увеличивать partitions у существующего topic?**  
A: Потому что mapping `key -> partition` изменится для новых сообщений. Это может нарушить assumptions о порядке/локальности обработки и повлиять на stateful consumers.

**Q: Когда нужен compacted topic, а когда обычный retention?**  
A: Compacted topic нужен для changelog/latest-value кейсов (state sync, CDC latest image). Обычный retention — для истории событий и replay-пайплайнов, где важна полная последовательность.

**Q: Что чаще всего ломают в topic design?**  
A: 1) Слишком мало partitions для будущего роста. 2) Нестабильный key. 3) Отсутствие явной retention policy. 4) Смешивание разных semantics (audit history и latest state) в одном topic.

## Связи

- [Partitioning](partitioning.md)
- [Kafka Storage and Log Internals](kafka-storage-and-log-internals.md)
- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md)
- [Kafka In DE Pipelines](kafka-in-de-pipelines.md)
- [Apache Kafka](../entities/apache-kafka.md)
