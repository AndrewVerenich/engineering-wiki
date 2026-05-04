# Kafka Replication and ISR

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Kafka реплицирует каждую partition через модель leader/follower. Продюсер пишет в leader, followers подтягивают лог. Качество durability определяется не только replication factor, но и сочетанием `acks`, ISR-состава и политик leader election.

## Ключевые элементы

| Элемент | Значение |
|---------|----------|
| Leader | единственная точка записи для partition |
| Followers | реплики, копирующие данные с leader |
| ISR | in-sync replicas, которые считаются «достаточно догнавшими» leader |
| LEO | log end offset реплики |
| HW | high watermark, до которого данные считаются безопасными для чтения |

## Матрица durability

| Настройка | Поведение | Риск |
|-----------|-----------|------|
| `acks=1` | ack после записи leader | потеря данных при падении leader до репликации |
| `acks=all` + `min.insync.replicas>=2` | ack только когда запись подтверждена ISR | выше latency, но выше durability |
| `acks=0` | fire-and-forget | максимальный риск потерь |

## ISR и деградация

- ISR может сужаться при lag/сбоях реплик.
- Если ISR ниже `min.insync.replicas`, broker начинает отклонять writes при `acks=all`.
- Это важный trade-off между availability и durability.

## Unclean leader election

Если включен unclean leader election, лидером может стать out-of-sync replica. Это увеличивает доступность после аварии, но может привести к потере ранее подтвержденных данных.

## Типичные вопросы на интервью

**Q: Почему replication factor=3 сам по себе не гарантирует no data loss?**  
A: Потому что гарантия зависит от write path: при `acks=1` producer может получить ack до того, как запись уйдет на другие реплики. Нужна комбинация `acks=all` и `min.insync.replicas`, плюс корректная election-политика.

**Q: Что такое HW и зачем он нужен?**  
A: HW — максимальный offset, подтвержденный репликами ISR. Consumer reading semantics и failover safety ориентируются на него, чтобы не отдавать данные, которые могут исчезнуть при смене лидера.

**Q: Когда Kafka начинает отдавать ошибки продюсеру при `acks=all`?**  
A: Когда ISR сужается ниже `min.insync.replicas`. Это намеренное поведение: система предпочитает fail writes, а не silently терять durability.

## Связи

- [Replication](replication.md)
- [Kafka Storage and Log Internals](kafka-storage-and-log-internals.md)
- [Kafka Controller and KRaft](kafka-controller-and-kraft.md)
- [Apache Kafka](../entities/apache-kafka.md)
