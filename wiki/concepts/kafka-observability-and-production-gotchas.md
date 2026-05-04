# Kafka Observability and Production Gotchas

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

Kafka редко «падает сразу». Чаще система деградирует: lag растет, ISR сужается, rebalance storms ломают throughput, hot partitions убивают p99. Senior-подход — ловить такие режимы до аварии через метрики и runbooks.

## Что мониторить в первую очередь

| Слой | Ключевые сигналы |
|------|------------------|
| Producer | request-latency, retries, error-rate, record-send-rate |
| Consumer | lag, rebalance rate, poll latency, commit failures |
| Broker | ISR shrinks/expands, under-replicated partitions, request queue, disk usage |
| Cluster | controller health, partition reassign duration, offline partitions |

## Частые production gotchas

| Проблема | Симптом | Реакция |
|----------|---------|---------|
| Hot partition | один consumer перегружен | пересмотреть key strategy/num partitions |
| Rebalance storms | частые stop/start processing | tune session/poll, cooperative rebalance |
| ISR shrink | write errors при `acks=all` | проверить lag/disk/network брокеров |
| Lag explosion | stale data в consumers | scale consumers + optimize processing path |

## Операционные ошибки

- Reset offsets без dry-run и без documented replay window.
- Rolling restart без контроля ISR/under-replicated partitions.
- Смешение критичных и не критичных workload в одном topic/cluster без quotas.

## Типичные вопросы на интервью

**Q: Почему lag — это не просто метрика производительности?**  
A: Потому что lag напрямую связан с бизнес-SLA свежести данных. Даже при «живом» сервисе высокий lag делает систему функционально устаревшей для клиентов и аналитики.

**Q: Что обычно вызывает rebalance storms?**  
A: Нестабильные consumers (GC pauses, crash loops), слишком агрессивные timeout/poll настройки, flapping deployment, network jitter. Последствие — постоянные перераспределения и просадка throughput.

**Q: Как безопасно делать rolling restart брокеров?**  
A: По одному брокеру, с контролем ISR и under-replicated partitions перед переходом к следующему; без этого легко нарушить durability и вызвать каскадные проблемы.

## Связи

- [Kafka Replication and ISR](kafka-replication-and-isr.md)
- [Kafka Consumer Internals and Offsets](kafka-consumer-internals-and-offsets.md)
- [Kafka Controller and KRaft](kafka-controller-and-kraft.md)
- [Apache Kafka](../entities/apache-kafka.md)
