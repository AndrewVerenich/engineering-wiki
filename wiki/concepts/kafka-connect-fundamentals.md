# Kafka Connect Fundamentals

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Kafka Connect — framework для стандартных source/sink интеграций без написания «клеевого» кода для каждого пайплайна. Он удобен для DE, когда нужен управляемый lifecycle connectors, offset tracking и наблюдаемость на уровне платформы.

## Базовая модель

| Компонент | Роль |
|-----------|------|
| Worker | runtime, исполняющий connectors/tasks |
| Connector | конфигурация source или sink integration |
| Task | параллельный execution unit |
| Offset storage | прогресс source/sink чтения |

## Режимы работы

| Режим | Когда подходит |
|-------|----------------|
| Standalone | dev/small локальные сценарии |
| Distributed | production, масштабирование и отказоустойчивость |

## Важные части прод-конфига

- Converters (Avro/Protobuf/JSON) и совместимость схем.
- SMT (single-message transforms) для простых преобразований.
- Error handling + DLQ для bad records.
- Тюнинг task parallelism и backpressure.

## Типичные вопросы на интервью

**Q: Когда лучше использовать Kafka Connect, а не писать custom consumer/sink?**  
A: Когда есть типовая интеграция и нужен быстрый запуск с минимальным custom code, стандартным мониторингом и централизованным управлением конфигами. Custom код оправдан при сложной бизнес-логике трансформации.

**Q: Почему DLQ в Connect важен?**  
A: Потому что в реальном потоке всегда есть malformed или несовместимые записи. Без DLQ connector может падать или silently пропускать ошибки, что разрушает data quality.

**Q: Где часто ошибаются при запуске Connect в production?**  
A: Недооценивают schema governance, не настраивают error policy, не планируют capacity для task scaling и не отделяют коннекторы разных SLA по кластерам/worker pools.

## Связи

- [Kafka in DE Pipelines](kafka-in-de-pipelines.md)
- [Kafka Schema Registry and Evolution](kafka-schema-registry-and-evolution.md)
- [CDC: Debezium → Kafka → ClickHouse](cdc-debezium-analytics-pipeline.md)
- [Apache Kafka](../entities/apache-kafka.md)
