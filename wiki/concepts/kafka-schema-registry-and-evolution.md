# Kafka Schema Registry and Evolution

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

В Kafka schema evolution — это не просто «добавить поле». Нужно одновременно сохранить совместимость producer/consumer, избежать silent data corruption и поддержать rolling deploy. Schema Registry делает эти контракты явными и проверяемыми.

## Базовые сущности

| Понятие | Суть |
|---------|------|
| Subject | namespace для версий схем |
| Compatibility mode | правила допустимых изменений |
| Serializer/Deserializer | запись/чтение payload с schema id |
| Registry | централизованный контроль схем |

## Subject naming strategies

| Strategy | Когда полезна | Риск |
|----------|----------------|------|
| TopicNameStrategy | простые topic-level контракты | сложнее reuse schemas между topics |
| RecordNameStrategy | reuse по record type | выше риск смешения эволюций без дисциплины |
| TopicRecordNameStrategy | компромисс topic + record | чуть сложнее governance |

## Compatibility levels (упрощенно)

| Режим | Что гарантирует |
|-------|-----------------|
| BACKWARD | новый consumer читает старые данные |
| FORWARD | старый consumer читает новые данные |
| FULL | обе стороны совместимы |
| TRANSITIVE варианты | совместимость проверяется не только с последней версией |

## Типичные эволюции

- Добавление optional field с default обычно безопасно.
- Удаление обязательного поля часто ломает backward compatibility.
- Переименование поля без migration strategy обычно breaking change.

## Типичные вопросы на интервью

**Q: Почему Schema Registry обязателен в больших Kafka-системах?**  
A: Потому что без централизованной проверки совместимости schema drift обнаруживается слишком поздно: consumers падают или молча читают некорректные данные. Registry с compatibility checks сдвигает риск на этап публикации схемы.

**Q: Чем отличается BACKWARD от FULL compatibility на практике?**  
A: BACKWARD защищает миграцию consumers вперёд, но не гарантирует, что старые consumers поймут новые сообщения. FULL нужен в rolling deploy средах, где старые и новые версии приложения одновременно живут в проде.

**Q: Где чаще всего делают ошибку в schema governance?**  
A: В смешении разных domain events в одном subject без четкого naming strategy и ownership. Это приводит к конфликтующим эволюциям и постоянным breaking changes.

## Связи

- [Encoding and Schema Evolution](encoding-and-schema-evolution.md)
- [Kafka In DE Pipelines](kafka-in-de-pipelines.md)
- [CDC: Debezium → Kafka → ClickHouse](cdc-debezium-analytics-pipeline.md)
- [Apache Kafka](../entities/apache-kafka.md)
