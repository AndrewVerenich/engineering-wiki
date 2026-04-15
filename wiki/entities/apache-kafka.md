# Apache Kafka

**Тип:** распределённый брокер сообщений / **log**-платформа

## В контексте вики

В [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) Kafka иллюстрирует **журнал с партициями**, репликацию логов, потоковую обработку и развязку производителей и потребителей.

## Ключевые характеристики

| Характеристика | Суть |
|----------------|------|
| **Topic / Partition** | Topic — логический канал. Partition — единица параллелизма и ordering. Сообщения внутри partition — строго упорядочены по offset. |
| **Producer** | Пишет в topic; выбирает partition по ключу (`hash(key) % partitions`) или round-robin. `acks=all` для durability. |
| **Consumer / Consumer Group** | Consumer group — набор consumer'ов, читающих topic. Одна partition → один consumer в группе. Rebalance при добавлении/удалении consumer'ов. |
| **Offset** | Позиция consumer'а в partition. Consumer коммитит offset; при рестарте — продолжает с последнего committed offset. |
| **Replication** | Каждая partition имеет leader + ISR (in-sync replicas). `min.insync.replicas` + `acks=all` → no data loss при сбое одного брокера. |
| **Retention** | По времени или размеру. Compacted topics — хранят последнее значение по ключу (snapshot). |
| **KRaft** | Замена ZooKeeper для metadata management (с Kafka 3.x). Kafka сама управляет consensus через Raft. |

## Типичные вопросы на интервью

**Q: Как Kafka гарантирует порядок сообщений?**
A: Порядок гарантирован **внутри partition**. Глобального порядка между partitions нет. Для ordering по entity — использовать entity ID как partition key (все события одной сущности в одной partition).

**Q: Что такое consumer group и как работает rebalance?**
A: Consumer group — набор consumer'ов, совместно читающих topic. Каждая partition назначается ровно одному consumer'у в группе. При добавлении/удалении consumer'а — **rebalance**: partitions перераспределяются. Во время rebalance — кратковременная остановка обработки. Cooperative rebalance (incremental) — минимизирует stop-the-world.

**Q: Что такое acks=all и min.insync.replicas?**
A: `acks=all` — producer ждёт подтверждения от **всех ISR** (in-sync replicas). `min.insync.replicas=2` — broker отклоняет writes, если ISR < 2. Комбинация: гарантия, что данные записаны минимум на 2 реплики. Trade-off: latency выше, но data loss при single-broker failure исключён.

**Q: Чем compacted topic отличается от обычного?**
A: Обычный — retention по времени/размеру, старые сообщения удаляются целиком. **Compacted** — Kafka хранит **последнее значение по ключу**; старые версии удаляются при compaction. Применение: changelog (CDC), latest state of entity, конфигурации. По сути — distributed key-value store с историей.

**Q: Как Kafka используется в data engineering pipeline?**
A: 1) **CDC transport**: Debezium → Kafka → DWH. 2) **Event bus**: микросервисы публикуют события, analytics consumer'ы читают. 3) **Buffer/decoupling**: между ingestion и processing (Spark/Flink/ClickHouse Kafka Engine). 4) **Stream processing**: Kafka Streams, ksqlDB, Flink читают и пишут в Kafka.

**Q: Что такое exactly-once в Kafka?**
A: **Idempotent producer** (`enable.idempotence=true`) — дедуп на уровне broker (no duplicates при retry). **Transactional producer** — атомарная запись в несколько partitions/topics. **Consumer** — `read_committed` для чтения только committed сообщений. Вместе — effectively once semantics **внутри** Kafka. За пределами (sink) — нужен idempotent consumer.

## Связи

- [Replication](../concepts/replication.md)
- [Partitioning](../concepts/partitioning.md)
- [Stream Processing](../concepts/stream-processing.md)
- [CDC: Debezium → Kafka → ClickHouse](../concepts/cdc-debezium-analytics-pipeline.md)
- [ClickHouse + Kafka Ingestion](../concepts/clickhouse-kafka-ingestion.md)