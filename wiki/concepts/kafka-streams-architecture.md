# Kafka Streams Architecture

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Kafka Streams — это **библиотека в JVM-приложении**, а не отдельный кластерный runtime. Приложение описывает **topology** (DAG операторов), а затем runtime делит её на tasks и исполняет в `StreamThread`-ах. Масштабирование происходит через увеличение числа инстансов приложения и/или `num.stream.threads`.

## Исполняемая модель

| Понятие | Смысл |
|---------|-------|
| **Topology** | Граф узлов `source -> processor -> sink`, описанный DSL или Processor API. |
| **Sub-topology** | Связанная часть topology между source/sink; помогает понимать этапы и repartition. |
| **Task** | Минимальная единица параллелизма. Обычно 1 task на partition source-topic. |
| **StreamThread** | Поток исполнения, который обрабатывает набор tasks. |
| **Application instance** | JVM-процесс со своим набором StreamThread-ов и локальным state. |

## Partition-to-task assignment

- Количество tasks обычно определяется количеством partition у input topic.
- Group rebalance распределяет tasks между инстансами приложения.
- Для stateful topology у каждого task есть локальные state stores.
- При failover task переносится на другой инстанс, который восстанавливает state из changelog.

## Repartition topics и standby replicas

- `groupBy`, некоторые joins и операции, меняющие key, могут создать **internal repartition topic**.
- Repartition повышает гибкость, но добавляет сеть/диск/latency.
- `num.standby.replicas` хранит тёплые копии state на других инстансах.
- Standby не обрабатывают трафик, но ускоряют failover, потому что восстанавливать state нужно меньше.

## Типичные вопросы на интервью

**Q: Чем Kafka Streams архитектурно отличается от Flink?**  
A: Kafka Streams исполняется внутри приложения и опирается на Kafka как на транспорт/журнал состояния. Flink — отдельный кластерный engine с JobManager/TaskManager и собственным lifecycle.

**Q: Что ограничивает параллелизм Kafka Streams?**  
A: В первую очередь число partition входных топиков. Можно увеличить `num.stream.threads`, но больше реальной пользы будет только если есть больше tasks/partition.

**Q: Почему repartition topics считаются дорогими?**  
A: Они требуют дополнительной сериализации, записи/чтения из Kafka и сетевого shuffle. Это повышает latency и нагрузку на кластер.

**Q: Зачем нужны standby replicas?**  
A: Чтобы при падении инстанса новый владелец task мог быстро продолжить обработку с почти готовым локальным state, а не восстанавливать всё из changelog с нуля.

## Связи

- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md) — как хранится и восстанавливается state.
- [Kafka Streams Exactly-Once](kafka-streams-exactly-once.md) — гарантии обработки в runtime.
- [Kafka Streams vs Flink](../comparisons/kafka-streams-vs-flink.md) — сравнение deployment и execution model.
- [Kafka Topic Design and Compaction](kafka-topic-design-and-compaction.md) — как дизайн topic влияет на tasks и internal topics.
