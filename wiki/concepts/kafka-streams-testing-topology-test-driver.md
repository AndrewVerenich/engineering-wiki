# Kafka Streams Testing with TopologyTestDriver

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

`TopologyTestDriver` позволяет тестировать topology локально, детерминированно и без запуска полноценного Kafka-кластера. Это ключ к быстрым unit/integration-like тестам stateful логики.

## Что тестировать

| Тип теста | Цель |
|-----------|------|
| **Stateless steps** | Проверка transform/filter/map логики. |
| **Stateful aggregates** | Проверка materialized state и changelog-поведения. |
| **Windowed logic** | Проверка времени, late events, suppress. |
| **Join-пайплайны** | Проверка co-partitioning assumptions и результатов join. |

## Практика

- Использовать `TestInputTopic` и `TestOutputTopic` для сценариев ввода/вывода.
- Явно управлять timestamps в тестах, особенно для окон и stream-time поведения.
- Проверять не только output, но и состояние store, если это критично для логики.
- Для end-to-end smoke полезно добавить отдельные integration tests с EmbeddedKafka/Testcontainers.

## Типичные вопросы на интервью

**Q: Почему одного integration testing недостаточно?**  
A: Он медленный, нестабилен и хуже локализует ошибки. `TopologyTestDriver` даёт быстрый feedback на уровне бизнес-логики.

**Q: Что чаще всего ломают в тестах Kafka Streams?**  
A: Время и ключи: забывают управлять timestamps и неверно задают key, из-за чего результат join/окон не совпадает с ожиданиями.

**Q: Нужно ли тестировать state stores напрямую?**  
A: Да, для stateful кейсов это полезно, иначе можно пропустить ошибки materialization при корректном внешнем output на малом наборе данных.

**Q: Когда всё же нужен реальный Kafka в тестах?**  
A: Для проверки сериализации/конфигов, security, rebalance и operational сценариев, которые TestDriver не эмулирует полностью.

## Связи

- [Kafka Streams Processor API](kafka-streams-processor-api.md) — тестирование кастомной логики.
- [Kafka Streams Windowing](kafka-streams-windowing.md) — сценарии с timestamp и late events.
- [Kafka Streams Joins](kafka-streams-joins.md) — проверка корректности join-паттернов.
- [Kafka Streams Interactive Queries and Deployment](kafka-streams-interactive-queries-and-deployment.md) — что проверять дополнительно в e2e.
