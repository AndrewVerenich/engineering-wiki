# Stream Processing with Apache Flink

| Поле | Значение |
|------|----------|
| **Авторы** | Fabian Hueske, Vasiliki Kalavri |
| **Издательство** | O'Reilly Media |
| **Первое издание** | 2019 (Flink 1.7) |
| **Тип** | книга |

## О книге

Каноническое инженерное руководство по Apache Flink от двух коммитеров проекта. Не «как пользоваться API», а **как Flink работает под капотом**: как устроен dataflow-граф, как координируются JobManager и TaskManager'ы, как работают watermarks, окна, state backends, как Flink даёт **exactly-once** и переживает падения через checkpointing. Книга строит модель потоковой обработки от первых принципов (event time, state, semantics) и постепенно показывает, как Flink её реализует.

## Структура

### Part I. Foundations

1. Introduction to Stateful Stream Processing — зачем нужен stateful streaming, эволюция от Lambda к Kappa.
2. Stream Processing Fundamentals — dataflow-граф, parallelism, **state**, **time**, processing guarantees. См. [Flink Stateless vs Stateful Operators](../concepts/flink-stateless-vs-stateful-operators.md).
3. The Architecture of Apache Flink — JobManager, TaskManager, task slots, обмен данными, checkpointing. См. [Apache Flink Architecture](../concepts/flink-architecture.md).

### Part II. The DataStream API

4. Setting Up a Development Environment — Maven, Scala/Java, Mini Cluster.
5. The DataStream API — sources, transformations, sinks, типы и сериализация. См. [Flink DataStream API](../concepts/flink-datastream-api.md).
6. Time-Based and Window Operators — event time, **watermarks**, **windows**, ProcessFunction, joins по времени. См. [Flink Time and Watermarks](../concepts/flink-time-and-watermarks.md), [Flink Windows](../concepts/flink-windows.md), [Flink ProcessFunction](../concepts/flink-process-function.md).
7. Stateful Operators and Applications — keyed state, operator state, broadcast state, state backends, scaling, queryable state. См. [Flink State Management](../concepts/flink-state-management.md).
8. Reading from and Writing to External Systems — source/sink connectors, **end-to-end exactly-once**. См. [Flink Exactly-Once Semantics](../concepts/flink-exactly-once-semantics.md).

### Part III. Operations

9. Setting Up Flink for Streaming Applications — deployment modes (session/per-job/application), HA, конфигурация.
10. Operating Flink and Streaming Applications — REST API, **savepoints**, версионирование, мониторинг, метрики. См. [Flink Checkpoints and Savepoints](../concepts/flink-checkpoints-and-savepoints.md).

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| Архитектура (JobManager, TaskManager, slots) | [Apache Flink Architecture](../concepts/flink-architecture.md) |
| DataStream API: как писать обработку | [Flink DataStream API](../concepts/flink-datastream-api.md) |
| Stateless vs stateful операторы | [Flink Stateless vs Stateful Operators](../concepts/flink-stateless-vs-stateful-operators.md) |
| Event time и watermarks | [Flink Time and Watermarks](../concepts/flink-time-and-watermarks.md) |
| Окна (tumbling, sliding, session) | [Flink Windows](../concepts/flink-windows.md) |
| State и state backends | [Flink State Management](../concepts/flink-state-management.md) |
| Checkpoints, savepoints, recovery | [Flink Checkpoints and Savepoints](../concepts/flink-checkpoints-and-savepoints.md) |
| End-to-end exactly-once | [Flink Exactly-Once Semantics](../concepts/flink-exactly-once-semantics.md) |
| ProcessFunction и таймеры | [Flink ProcessFunction](../concepts/flink-process-function.md) |
| Flink vs Spark Structured Streaming | [Flink vs Spark Structured Streaming](../comparisons/flink-vs-spark-structured-streaming.md) |

## Сущности

- [Apache Flink](../entities/apache-flink.md)
- [Apache Kafka](../entities/apache-kafka.md) — основной source/sink в production-пайплайнах Flink.

## Связанные обзоры

- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md)
- [Stream Processing](../concepts/stream-processing.md) — теория из DDIA, на которую опирается Flink.
