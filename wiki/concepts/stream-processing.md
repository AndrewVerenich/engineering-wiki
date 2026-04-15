# Stream Processing

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 11)

## Суть

**Потоковая обработка** — непрерывная обработка событий с низкой задержкой. Темы: **event streams**, окна по времени, **join** потоков, семантика **exactly-once** (через идемпотентность, дедупликацию, транзакционные sink’и), сравнение с batch как обработка бесконечного набора vs ограниченного.

## Ключевые концепции

| Концепция | Суть |
|-----------|------|
| **Event log** | Immutable, append-only последовательность событий. Kafka topic — пример. Consumer'ы читают с offset, каждый в своём темпе. |
| **Windowing** | Группировка событий по времени: **tumbling** (фиксированные неперекрывающиеся окна), **hopping/sliding** (перекрываются), **session** (по активности пользователя). |
| **Event time vs processing time** | **Event time** — когда событие произошло. **Processing time** — когда система его обработала. Late events приходят с event time в прошлом — нужен **watermark**. |
| **Watermark** | Эвристика: «все события до этого timestamp уже пришли». Позволяет закрывать окна. Слишком ранний → потеря late events; слишком поздний → высокая latency. |
| **Exactly-once semantics** | Гарантия, что каждое событие обработано ровно один раз. На практике реализуется через **at-least-once + idempotent writes** или **transactional producer/consumer** (Kafka transactions). |
| **Stream-table duality** | Stream — лог изменений; table — текущее состояние. Stream → compact → table; table → CDC → stream. Kafka Streams и ksqlDB строят на этом. |

## Паттерны join в stream

| Join | Что соединяет | Пример |
|------|---------------|--------|
| **Stream-stream** | Два потока событий по ключу в окне | Clicks join Impressions в окне 1 час |
| **Stream-table** | Поток + lookup в таблицу (snapshot/changelog) | Orders join Customers (обогащение) |
| **Table-table** | Два changelog потока | KTable-KTable join в Kafka Streams |

## Типичные вопросы на интервью

**Q: Чем event time отличается от processing time и почему это важно?**
A: Event time — когда событие **реально произошло** (timestamp в данных). Processing time — когда система его **получила**. Разница: сетевые задержки, перезапуски, мобильные устройства offline. Если агрегировать по processing time — результат зависит от скорости ingestion, а не от реальности. Для корректности нужен event time + watermarks.

**Q: Что такое exactly-once и возможно ли оно?**
A: В строгом смысле impossible across system boundaries. На практике реализуется как **effectively once**: at-least-once delivery + **idempotent processing** (dedup по ключу, upsert, transactional writes). Kafka поддерживает exactly-once через transactional producer + consumer read_committed + idempotent producer.

**Q: Как обработать late-arriving events?**
A: **Watermark** определяет, когда считать окно «закрытым». Late events после watermark: 1) отбросить (потеря), 2) side output для отдельной обработки, 3) allowed lateness — окно остаётся открытым дольше и результат обновляется. Выбор зависит от бизнес-требований.

**Q: Когда использовать Kafka Streams vs Flink vs Spark Streaming?**
A: **Kafka Streams** — lightweight library, встраивается в JVM-приложение, хорошо для обогащения и простых агрегатов. **Flink** — полноценный stream engine, мощные окна, exactly-once, event time; для сложных пайплайнов. **Spark Structured Streaming** — micro-batch, удобно если уже используешь Spark для batch. В data engineering: Flink для true real-time; Spark SS для unified batch+stream.

**Q: Что такое stream-table duality?**
A: Таблица — это materialised view потока изменений (compacted log). Поток — это changelog таблицы. Kafka compacted topics — пример: log → table. CDC — table → stream. Это основа для CQRS, event sourcing, и Kafka Streams KTable.

## Связи

- [Encoding and Schema Evolution](encoding-and-schema-evolution.md) — форматы в событиях.
- [Apache Kafka](../entities/apache-kafka.md) — лог как транспорт потоков.
- [Derived Data and Systems](derived-data-and-systems.md) — unifying batch и stream.
