# Partitioning

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 6)

## Суть

**Партиционирование (шардирование)** распределяет данные по узлам по ключу. Важны: выбор ключа, **rebalancing** при добавлении узлов, запросы, затрагивающие несколько партиций (**scatter/gather**), и вторичные индексы (локальные vs глобальные).

## Стратегии партиционирования

| Стратегия | Принцип | Плюсы | Минусы |
|-----------|---------|-------|--------|
| **По ключу (hash)** | `hash(key) % N` | Равномерное распределение | Range-запросы по ключу → scatter/gather |
| **По диапазону (range)** | Диапазоны ключей на разные узлы | Эффективные range-сканы | Hot spots при неравномерном распределении |
| **Compound** | Hash по одной части, range по другой | Баланс: равномерность + range | Сложнее проектирование |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **Shard / partition** | Подмножество данных на одном узле; термины часто синонимы (partition — Kafka, shard — MongoDB/Elasticsearch). |
| **Hot spot** | Перегруженная партиция — весь трафик идёт на один узел. Причина: плохой выбор ключа (timestamp как ключ — все пишут в «последнюю» партицию). |
| **Rebalancing** | Перемещение данных между узлами при добавлении/удалении. Проблема: объём перемещения и downtime. Подходы: consistent hashing, fixed number of partitions, dynamic splitting. |
| **Scatter/gather** | Запрос, затрагивающий все партиции (нет подходящего ключа в фильтре). Дорого: latency = max(all partitions). |
| **Secondary index (local vs global)** | **Local** — каждая партиция содержит свой индекс (write простой, read — scatter). **Global** — индекс по всем данным, сам разбит на партиции (read быстрый, write — multi-partition update). |

## Типичные вопросы на интервью

**Q: Как выбрать ключ партиционирования?**
A: Исходить из паттернов запросов: ключ должен быть в самых частых фильтрах, чтобы запросы попадали в одну/несколько партиций (а не scatter/gather). При этом кардинальность ключа должна обеспечить равномерное распределение. В Kafka — обычно business entity ID; в ClickHouse — часто `(event_type, toDate(ts))`.

**Q: Что такое hot spot и как с ним бороться?**
A: Когда одна партиция получает непропорционально много данных (например timestamp-based partitioning и все пишут «сейчас»). Решения: добавить salt/random suffix к ключу (ценой scatter при чтении), compound key, или hash-based partitioning.

**Q: Чем отличается partitioning в Kafka от partitioning в ClickHouse?**
A: **Kafka**: partition — единица параллелизма и ordering; данные распределяются по partition key, consumer per partition. **ClickHouse**: partition — логическая группа для lifecycle (TTL, DROP PARTITION); основной механизм отсечения — `ORDER BY` + sparse index, а не partition pruning.

**Q: Как работает rebalancing при добавлении узлов?**
A: Нужно перераспределить данные. Проблема — объём перемещения. Подходы: **consistent hashing** (минимум перемещений), **fixed number of partitions** (partition → node mapping меняется, данные не дробятся), **dynamic splitting** (partition делится при росте). В Kafka: reassign partitions через admin tool; в ClickHouse: sharding через Distributed table + ручное управление.

**Q: Что такое scatter/gather и почему это проблема?**
A: Запрос без подходящего partition key → координатор отправляет запрос на все партиции и ждёт все ответы. Latency = max(все узлы). Throughput падает линейно. В аналитических системах (ClickHouse) это менее критично (всё равно scan), но в OLTP — сильно бьёт по latency.

## Связи

- [Replication](replication.md) — каждая партиция обычно реплицируется.
- [Transactions and Isolation](transactions-and-isolation.md) — транзакции между партициями дороже.
- [Apache Kafka](../entities/apache-kafka.md) — партиции топиков.
