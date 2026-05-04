# Kafka Storage and Log Internals

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Kafka хранит данные как append-only log, разбитый на сегменты. Это позволяет дешевые последовательные записи, эффективный replay и предсказуемую модель retention. Производительность во многом обеспечивается не «магией Kafka», а сочетанием структуры лога, page cache ОС и batched network I/O.

## Как устроен log на диске

| Компонент | Что хранит | Зачем |
|-----------|------------|-------|
| Log segment (`*.log`) | батчи сообщений | базовый append-only файл partition |
| Offset index (`*.index`) | offset -> position | быстрый seek по offset |
| Time index (`*.timeindex`) | timestamp -> position | time-based lookup/retention |

## Segment rolling и retention

| Настройка | Эффект | Trade-off |
|-----------|--------|-----------|
| `segment.bytes` | размер сегмента до roll | меньше файлов vs более грубая очистка |
| `segment.ms` | roll по времени | стабильность lifecycle при низком трафике |
| `retention.ms` | хранение по времени | replay window vs storage cost |
| `retention.bytes` | хранение по размеру | контроль диска vs историчность |

## Почему Kafka быстрый на I/O

- Append-only writes уменьшают random disk seeks.
- Send path использует page cache ОС и zero-copy (`sendfile`) в типовом read path.
- Producer batching и compression снижают syscalls и сетевые накладные расходы.

## Tiered storage (обзор)

Tiered storage разгружает локальный диск брокеров, перенося «холодные» сегменты во внешнее хранилище. Это увеличивает retention window без линейного роста локальных NVMe затрат, но добавляет latency для холодных чтений и усложняет ops-профиль.

## Типичные вопросы на интервью

**Q: Почему Kafka не хранит сообщения в одной большой файле topic?**  
A: Потому что сегментация упрощает retention и recovery: можно удалять старые куски целиком, быстрее делать startup и изолировать повреждения. Один гигантский файл усложняет индексирование и lifecycle-операции.

**Q: Как размер сегмента влияет на работу кластера?**  
A: Слишком маленькие сегменты увеличивают количество файлов, метаданных и cleanup overhead. Слишком большие — делают retention и compaction менее точными и удлиняют операции восстановления. Выбор зависит от throughput и SLA на replay window.

**Q: Почему page cache критичен для Kafka?**  
A: Kafka в значительной степени полагается на кеш ОС для «горячих» сегментов. Если page cache недостаточен, чтение/репликация упираются в физический диск, и p99 latency растет.

## Связи

- [Storage and Retrieval](storage-and-retrieval.md)
- [Kafka Replication and ISR](kafka-replication-and-isr.md)
- [Kafka Topic Design and Compaction](kafka-topic-design-and-compaction.md)
- [Apache Kafka](../entities/apache-kafka.md)
