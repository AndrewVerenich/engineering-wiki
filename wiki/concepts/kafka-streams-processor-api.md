# Kafka Streams Processor API

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

DSL закрывает большинство кейсов, но Processor API нужен, когда требуется точный контроль над обработкой: кастомный state access, сложная маршрутизация, punctuations и нестандартная логика времени.

## Когда DSL недостаточно

| Сценарий | Почему нужен Processor API |
|----------|----------------------------|
| Нестандартная state machine | Нужен ручной контроль чтения/записи state. |
| Тонкий контроль emission | Нужно управлять `forward()` и временем отправки. |
| Кастомные таймеры | Нужны `punctuate` по stream-time или wall-clock. |
| Сложный audit/debug | Требуется явный доступ к context метаданным. |

## Ключевые элементы

- `ProcessorContext` даёт доступ к topic/partition/offset/timestamp.
- `StateStore` подключается и управляется через lifecycle процессора.
- `PunctuationType.STREAM_TIME` реагирует на прогресс stream time.
- `PunctuationType.WALL_CLOCK_TIME` работает по системному времени.

## Типичные вопросы на интервью

**Q: Когда лучше остаться на DSL, а не идти в Processor API?**  
A: Если задача выражается стандартными transform/join/window/aggregate, DSL проще, безопаснее и легче поддерживается командой.

**Q: Что опасного в wall-clock punctuations?**  
A: Они могут не совпадать с business-time данными и приводить к неожиданной логике при паузах входного потока.

**Q: Почему Processor API часто считают более рискованным?**  
A: Больше ручного кода, выше шанс нарушить семантику времени/состояния и сложнее тестирование.

**Q: Можно ли смешивать DSL и Processor API?**  
A: Да, это частый production-подход: базовый pipeline на DSL, специальные участки на Processor API.

## Связи

- [Kafka Streams Time and Stream Time](kafka-streams-time-and-streamtime.md) — time semantics для punctuations.
- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md) — управление состоянием в custom logic.
- [Kafka Streams Testing with TopologyTestDriver](kafka-streams-testing-topology-test-driver.md) — как тестировать custom processors.
- [Kafka Streams Architecture](kafka-streams-architecture.md) — роль Processor API в topology.
