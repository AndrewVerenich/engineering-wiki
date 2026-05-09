# Kafka Streams Joins

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Kafka Streams поддерживает несколько join-моделей, и выбор зависит от того, объединяем ли мы события со событиями, поток с текущим состоянием или две материализованные таблицы. Ключевой инженерный момент: **co-partitioning и корректная модель времени**.

## Виды joins

| Join | Что объединяет | Особенности |
|------|----------------|------------|
| **KStream-KStream** | Два event stream | Почти всегда windowed join, требует окно и grace. |
| **KStream-KTable** | Поток + таблица состояния | Lookup по key, типичный enrichment. |
| **KStream-GlobalKTable** | Поток + полная копия таблицы | Не требует co-partitioning, но дороже по памяти. |
| **KTable-KTable** | Две changelog-таблицы | Join как обновление materialized state. |

## Co-partitioning и internal topics

- Для stream/table joins важно совпадение key-space и partitioning.
- Если key меняется (`selectKey`, `groupBy`), Streams создаёт repartition topic.
- Неверный ключ приводит к пропускам join или data skew.
- Foreign-key join для KTable полезен, но повышает сложность состояния и трафика.

## Типичные ошибки

- Window слишком маленькое: теряются поздние совпадения.
- Неверный serdes: join-ключи не декодируются консистентно.
- Использование `GlobalKTable` на большом справочнике: перегрузка инстансов.

## Типичные вопросы на интервью

**Q: Почему KStream-KStream join обычно windowed?**  
A: Потому что два независимых потока приходят с задержками и out-of-order. Без окна невозможно корректно и ограниченно по ресурсам определить, какие события считаются совпавшими.

**Q: Когда стоит выбирать KStream-GlobalKTable join?**  
A: Когда reference-таблица небольшая и хочется избежать строгого co-partitioning между потоком и таблицей.

**Q: Что значит co-partitioning practically?**  
A: Одинаковый ключевой space, совместимый partitioner и одинаковое число partitions там, где это требуется для join.

**Q: Какие trade-offs у foreign-key join KTable-KTable?**  
A: Удобная модель для реляционных связей, но больше internal state и traffic, сложнее прогнозировать latency и стоимость восстановления.

## Связи

- [Kafka Streams: KStream, KTable, GlobalKTable](kafka-streams-kstream-and-ktable.md) — исходная модель типов.
- [Kafka Streams Windowing](kafka-streams-windowing.md) — окна для stream-stream joins.
- [Kafka Streams State Stores and Changelog](kafka-streams-state-stores-and-changelog.md) — материализация join state.
- [Partitioning](partitioning.md) — базовая теория ключей и шардирования.
