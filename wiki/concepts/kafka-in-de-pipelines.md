# Kafka in DE Pipelines

**Источник:** [Kafka in Action](../sources/kafka-in-action.md), [Kafka Official Documentation](../sources/kafka-official-documentation.md)

## Суть

В data engineering Kafka чаще всего играет роль durable transport layer между источниками изменений и аналитическими/stream processing системами. Ключевые вопросы: replay window, schema contracts, idempotent sink и контролируемый backfill.

## Типовая роль Kafka в DE

| Роль | Пример |
|------|--------|
| CDC transport | Debezium -> Kafka -> DWH |
| Ingestion buffer | producer services -> Kafka -> ClickHouse/Flink/Spark |
| Replay source | восстановление витрин из retention window |

## Sink-паттерны

| Паттерн | Когда использовать |
|---------|-------------------|
| Append sink | event history и audit |
| Upsert sink | latest-state витрины |
| Idempotent load | at-least-once delivery и retries |

## Backfill и replay

- Планировать retention так, чтобы покрывать realistic replay/backfill окно.
- Делать dry-run на тестовой группе consumers перед массовым offset reset.
- Хранить source metadata (event time, key, partition, offset) в landing.

## Schema contracts

Schema drift в Kafka быстро ломает downstream модели. Нужно сочетание registry governance, staged rollout и тестов на совместимость в DE pipeline.

## Типичные вопросы на интервью

**Q: Почему Kafka удобен как транспорт для CDC?**  
A: Потому что дает буферизацию, replay и развязку источника/приемников. OLTP не зависит от скорости DWH-потребителей, а pipeline переживает кратковременные сбои за счет lag-window.

**Q: Что важнее в DE pipeline: exactly-once или idempotent sink?**  
A: На практике чаще критичнее idempotent sink, потому что внешние границы редко поддерживают идеальный end-to-end EOS. Надежный upsert/dedup обычно дает более предсказуемый результат.

**Q: Какую ошибку часто допускают с retention?**  
A: Ставят слишком короткий retention, после чего при длительном инциденте невозможно корректно replay-нуть данные и приходится делать дорогой backfill из первичного источника.

## Связи

- [CDC: Debezium → Kafka → ClickHouse](cdc-debezium-analytics-pipeline.md)
- [ClickHouse + Kafka Ingestion](clickhouse-kafka-ingestion.md)
- [Kafka Connect Fundamentals](kafka-connect-fundamentals.md)
- [Apache Kafka](../entities/apache-kafka.md)
