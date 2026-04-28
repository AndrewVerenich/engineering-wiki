# Flink vs Spark Structured Streaming

**Контекст:** оба движка решают задачу потоковой обработки, но архитектурно делают это по-разному: Flink — true streaming engine, Spark Structured Streaming — micro-batch поверх Spark SQL runtime.

## Главная разница

| | Flink | Spark Structured Streaming |
|--|------|----------------------------|
| Модель выполнения | event-at-a-time (record by record) | micro-batch (по trigger) |
| Задержка | миллисекунды-десятки мс | обычно сотни мс - секунды |
| Event time/watermarks | нативно, first-class | поддерживается, но привязано к batch trigger |
| Stateful processing | очень гибко (KeyedProcessFunction, timers) | есть, но менее low-level гибкость |
| Exactly-once | strong engine semantics + checkpointed state | exactly-once для supported sinks/modes |
| Экосистема | DataStream/Table/SQL, Flink-native connectors | сильная экосистема Spark SQL/DataFrame/ML |

## Когда что выбирать

| Сценарий | Лучший выбор | Почему |
|----------|--------------|--------|
| Реал-тайм алерты < 1 сек | **Flink** | ниже latency и сильный контроль над временем |
| Unified batch + stream в одном стеке lakehouse | **Spark SS** | одна платформа и DataFrame API |
| Сложные state machines и timers | **Flink** | KeyedProcessFunction и event-time timers |
| BI/ETL команда уже живёт в Spark | **Spark SS** | ниже порог входа и общая инфраструктура |

## Типичные вопросы на интервью

**Q: Почему Flink обычно даёт меньшую latency?**  
A: Потому что обрабатывает события по одному, без ожидания формирования следующего micro-batch. Spark Structured Streaming планирует и выполняет небольшие batch jobs по trigger interval, что добавляет scheduling overhead и нижнюю границу latency.

**Q: Значит ли это, что Spark хуже для streaming?**  
A: Нет. Для многих data engineering задач latency в секундах приемлема, а выгода от единой Spark-платформы (batch+stream+SQL) выше. Выбор определяется SLO по задержке, сложностью stateful-логики и командной экспертизой.

**Q: Где сложнее обеспечить корректность при late events?**  
A: Оба поддерживают event time и watermarks. Flink даёт более fine-grained контроль (timers, ProcessFunction, боковые потоки late events), Spark — более декларативный путь через Structured APIs. Для кастомной логики late/retractions Flink обычно удобнее.

## Связи

- [Stream Processing](../concepts/stream-processing.md)
- [Flink Time and Watermarks](../concepts/flink-time-and-watermarks.md)
- [Flink State Management](../concepts/flink-state-management.md)
- [Flink Exactly-Once Semantics](../concepts/flink-exactly-once-semantics.md)
