# Flink Exactly-Once Semantics

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 8)

## Суть

В Flink exactly-once достигается на уровне движка через checkpointed state и replay input'а. Но реальная бизнес-гарантия зависит от **внешних систем**: source и sink. Поэтому корректнее мыслить как **end-to-end effectively once**: Flink гарантирует консистентную обработку, а ты обеспечиваешь идемпотентный/транзакционный выход.

## Семантики доставки

| Семантика | Что гарантирует | Риск |
|----------|------------------|------|
| At-most-once | событие не более 1 раза | потеря данных |
| At-least-once | событие минимум 1 раз | дубликаты |
| Exactly-once (engine) | state/output pipeline-consistent | сложность sink integration |

## Как Flink реализует exactly-once

1. Source offsets включены в checkpoint.
2. Operator state snapshot'ится консистентно.
3. При recovery source читается с checkpointed offsets.
4. Stateful operators продолжают из последнего snapshot.

Внутри Flink это исключает «двойной учёт» состояния.

## End-to-end аспекты (source/sink)

| Компонент | Что нужно |
|-----------|-----------|
| Source | replayable чтение с точкой позиции (Kafka offsets, file positions) |
| Sink | transactional commit или idempotent upsert |

Паттерны:
- Kafka -> Flink -> Kafka: transactional sink + read_committed consumer.
- Kafka -> Flink -> DB: upsert by primary key / dedup key.
- Kafka -> Flink -> object storage: atomic commit files after checkpoint.

## Типичные вопросы на интервью

**Q: Почему “exactly-once” часто называют маркетинговым термином?**  
A: Потому что между независимыми системами абсолютный one-shot трудно гарантировать без распределённой транзакции. На практике строят effectively once: checkpoint + replay + idempotent/transactional sink. Это инженерно достаточно для бизнеса.

**Q: Что нужно, чтобы получить end-to-end exactly-once в Kafka pipeline?**  
A: Replayable source, checkpointing во Flink, transactional Kafka sink (`DeliveryGuarantee.EXACTLY_ONCE`), и downstream consumers в режиме `read_committed`. Иначе потребитель увидит uncommitted/duplicate records.

**Q: Если sink не поддерживает транзакции, что делать?**  
A: Делать idempotent writes: upsert по deterministic key, dedup table, versioning (last-write-wins с sequence/timestamp), external exactly-once key store. Тогда повторная отправка после recovery не меняет итоговое состояние.

## Связи

- [Flink Checkpoints and Savepoints](flink-checkpoints-and-savepoints.md)
- [Flink DataStream API](flink-datastream-api.md)
- [Apache Kafka](../entities/apache-kafka.md)
- [Stream Processing](stream-processing.md)
