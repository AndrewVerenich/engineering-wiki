# Kafka Controller and KRaft

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Контроллер в Kafka отвечает за cluster metadata: лидеры партиций, membership брокеров, операции reassignment и recovery после сбоев. Современный Kafka использует KRaft (Kafka Raft) вместо ZooKeeper: metadata хранится и реплицируется внутри Kafka-кворума.

## Что делает контроллер

| Обязанность | Пример |
|-------------|--------|
| Election лидеров partition | смена лидера после падения брокера |
| Управление metadata | topics, partitions, ISR state |
| Координация admin операций | partition reassignment, topic lifecycle |
| Контроль health кластера | реакции на broker up/down |

## KRaft vs ZooKeeper (коротко)

| Аспект | ZooKeeper era | KRaft era |
|--------|---------------|-----------|
| Метаданные | внешняя система ZooKeeper | встроенный Raft quorum |
| Deployment complexity | больше компонентов | проще topology |
| Failure model | split между Kafka и ZK | единая control plane модель |

## Control plane vs data plane

- **Control plane**: metadata decisions, controller quorum, leader assignments.
- **Data plane**: чтение/запись реальных message batches в partition logs.

Проблемы control plane (flapping controller, quorum issues) часто выглядят как data plane деградация (рост latency, stuck rebalances).

## Типичные вопросы на интервью

**Q: Что изменилось для эксплуатации Kafka после перехода на KRaft?**  
A: Уходит отдельный ZooKeeper-кластер и часть operational сложности. Но появляется новый фокус: устойчивость controller quorum, корректный bootstrap и мониторинг metadata log.

**Q: Почему отказ контроллера не всегда означает потерю данных?**  
A: Потому что данные лежат в partition logs на брокерах, а контроллер управляет метаданными и назначениями. Потери данных связаны в первую очередь с replication semantics, а не с самим фактом смены контроллера.

**Q: Где частая ошибка при обсуждении KRaft на интервью?**  
A: Смешивать control plane и data plane. KRaft улучшает управление метаданными, но не отменяет необходимости корректно настраивать ISR, `acks`, retention и клиентские параметры.

## Связи

- [Consistency and Consensus](consistency-and-consensus.md)
- [Kafka Replication and ISR](kafka-replication-and-isr.md)
- [Kafka Observability and Production Gotchas](kafka-observability-and-production-gotchas.md)
- [Apache Kafka](../entities/apache-kafka.md)
