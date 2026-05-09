# Kafka Streams vs Kafka Consumer

**Контекст:** оба подхода читают данные из Kafka, но уровень абстракции и встроенные гарантии разные: low-level Consumer API против high-level stream processing DSL/runtime.

## Главная разница

| Критерий | Kafka Streams | Kafka Consumer API |
|----------|---------------|--------------------|
| Уровень абстракции | DSL + runtime для topology | Низкоуровневое чтение/коммит offset |
| Stateful processing | Встроено (stores + changelog) | Нужно реализовать вручную |
| Joins/windows | Готовые primitives | Реализация вручную |
| Exactly-once | Встроенный режим `exactly_once_v2` | Нужно строить вручную (сложно) |
| Гибкость кастомной логики | Высокая, но в рамках Streams model | Максимальная, но дороже в разработке |

## Когда что выбирать

| Сценарий | Лучший выбор | Почему |
|----------|--------------|--------|
| Нужны joins/aggregates/windows/state | **Kafka Streams** | Меньше custom-кода, выше надёжность. |
| Простейший poll-process-write без state | **Consumer API** | Меньше runtime overhead. |
| Требуется строгий контроль над loop/protocol | **Consumer API** | Максимально явное управление. |
| Быстрое развитие stream-фич в сервисе | **Kafka Streams** | Богатая DSL-модель и встроенные гарантии. |

## Типичные вопросы на интервью

**Q: Почему нельзя просто взять Consumer API и написать всё руками?**  
A: Можно, но стоимость владения резко растёт: state, joins, time/window semantics, rebalance safety и EOS становятся вашей ответственностью.

**Q: Когда Consumer API всё же правильный выбор?**  
A: Для простых интеграционных воркеров без сложной stream-логики, где важен полный контроль и минимальная зависимость от фреймворка.

**Q: Какой главный риск у самописного stateful processing на Consumer API?**  
A: Несогласованность между offset/state/output при сбоях, что ведёт к дублям или потерям без продуманной транзакционной схемы.

**Q: Что даёт Streams для команды разработки?**  
A: Быстрее реализация бизнес-фич, меньше инфраструктурного «клея» и выше предсказуемость поведения под нагрузкой.

## Связи

- [Kafka Consumer Internals and Offsets](../concepts/kafka-consumer-internals-and-offsets.md)
- [Kafka Streams Architecture](../concepts/kafka-streams-architecture.md)
- [Kafka Streams Exactly-Once](../concepts/kafka-streams-exactly-once.md)
- [Kafka Backend Patterns: Outbox, DLQ, Retry](../concepts/kafka-backend-patterns-outbox-dlq-retry.md)
