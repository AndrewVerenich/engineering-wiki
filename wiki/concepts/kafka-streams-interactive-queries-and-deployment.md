# Kafka Streams Interactive Queries and Deployment

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Kafka Streams можно использовать как stateful microservice: topology поддерживает materialized state, а приложение отдаёт его через API (Interactive Queries). Это удобно для low-latency read path, но требует аккуратного deployment и маршрутизации запросов.

## Interactive Queries

| Компонент | Назначение |
|-----------|------------|
| **Queryable store** | Доступ к локальному materialized state в процессе. |
| **Metadata discovery** | Поиск инстанса-владельца ключа (`StreamsMetadata`/metadata API). |
| **Remote forwarding** | Если ключ не локальный, запрос проксируется на нужный инстанс. |

## Deployment-практики

- Масштабировать по числу partitions и объёму state, а не только по CPU.
- Держать стабильные `application.id` и совместимые serdes между релизами.
- Использовать rolling restart с контролем restore lag.
- Отдельно мониторить время восстановления state после ребаланса.

## Ключевые метрики

- `process-rate`, `process-latency-avg`.
- `commit-latency-avg`.
- `records-lag-max` по входным topic.
- размер локальных state stores и restore throughput.

## Типичные вопросы на интервью

**Q: Что главное в Interactive Queries с точки зрения архитектуры?**  
A: Это distributed key-value read layer поверх локальных state stores. Нужно уметь находить владельца ключа и маршрутизировать запросы между инстансами.

**Q: Почему IQ не всегда лучше внешней БД?**  
A: IQ хорошо работает для данных, которые уже materialize в Streams. Для сложных ad-hoc запросов, глобальных сканов и независимого lifecycle часто удобнее отдельная БД.

**Q: Какой риск при rolling deploy stateful Streams-сервиса?**  
A: Массовый rebalance и длительный restore state могут ухудшить latency. Нужен поэтапный rollout и capacity reserve.

**Q: Какие метрики первыми смотреть при деградации?**  
A: Lag, processing/commit latency и состояние restore. Они быстро показывают, bottleneck в input, compute или state recovery.

## Связи

- [Kafka Streams Architecture](kafka-streams-architecture.md) — tasks/threads/deployment model.
- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md) — откуда берётся queryable state.
- [Kafka Streams Testing with TopologyTestDriver](kafka-streams-testing-topology-test-driver.md) — тестирование core-логики до deploy.
- [Kafka Observability and Production Gotchas](kafka-observability-and-production-gotchas.md) — общие production-практики Kafka-экосистемы.
