# Flink Stateless vs Stateful Operators

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 2, 7)

## Суть

В Flink каждый оператор — **либо stateless, либо stateful**, и это различие определяет всё: что переживает рестарт, как работает rescaling, нужен ли checkpoint, как Flink даёт exactly-once. **Stateless** оператор обрабатывает событие, опираясь только на само событие (`map`, `filter`, `flatMap`). **Stateful** помнит что-то между событиями: счётчики, окна, последнее значение, паттерны (`reduce`, `aggregate`, `window`, `process` с state, joins). State — first-class citizen: Flink сам шардирует его по ключу, чекпоинтит и восстанавливает при сбое.

## Stateless операторы

| Оператор | Что делает | Память |
|----------|-----------|--------|
| `map` | `T → R` | Не хранит ничего между событиями |
| `flatMap` | `T → Collection<R>` (один в ноль/много) | Не хранит |
| `filter` | `T → Boolean` | Не хранит |
| `union` | Объединение потоков одного типа | Не хранит |
| `rebalance`, `rescale`, `keyBy`, `broadcast` | Перепартицирование | Не хранит — это распределение, не вычисление |

Свойства:
- **Идемпотентны** при повторной обработке (если `f` детерминирована).
- **Параллелятся бесплатно** — каждый subtask работает независимо.
- **Rescaling тривиальный** — нет state, который нужно перебалансировать.
- **Checkpoint содержит только offset'ы source'ов**, никакого state самого оператора.

Если pipeline целиком stateless — Flink почти неотличим от любого другого framework'а. Вся ценность Flink — в state.

## Stateful операторы

### Что значит «stateful»

Stateful оператор хранит **managed state** — структуры данных, которыми управляет Flink: они автоматически:
- участвуют в **checkpoint'ах** (durable snapshot);
- **шардируются** по ключу через **key groups** (см. [Flink State Management](flink-state-management.md));
- **перебалансируются** при rescaling;
- доступны через типизированные API (`ValueState`, `ListState`, `MapState`, `ReducingState`, `AggregatingState`).

Не путать с **обычным полем класса** оператора — оно тоже «состояние», но Flink о нём не знает: оно НЕ чекпоинтится, теряется при сбое, не перебалансируется. Это анти-паттерн.

### Категории stateful операторов

| Тип оператора | API | Что хранит |
|---------------|-----|-----------|
| **Reduce / Aggregate** | `keyedStream.reduce(...)`, `.aggregate(...)` | Текущее значение свёртки по каждому ключу (`ReducingState`, `AggregatingState`) |
| **Windowed** | `.window(...).aggregate/reduce/process(...)` | Буфер событий или accumulator на каждое (key, window) — см. [Flink Windows](flink-windows.md) |
| **ProcessFunction со state** | `.process(new KeyedProcessFunction<...>)` с зарегистрированным `ValueState`/`ListState`/`MapState` | Произвольное состояние по ключу — см. [Flink ProcessFunction](flink-process-function.md) |
| **CEP / pattern matching** | `CEP.pattern(...)` | NFA (non-deterministic finite automaton) состояние для каждого ключа |
| **Joins по времени** | `intervalJoin`, оконные joins | Буфер событий из обеих сторон в окне |
| **Stateful sources** | `KafkaSource` и т.п. | Offset'ы, прочитанные splits |
| **Stateful sinks** | `KafkaSink` (transactional), `FileSink` | Pending транзакции, незавершённые файлы |

## Stateless vs Stateful: сравнение

| | Stateless | Stateful |
|--|-----------|----------|
| **Что нужно для recovery** | Только source offset'ы | Offset'ы + полный state оператора |
| **Размер checkpoint'а** | Малый | От МБ до десятков ТБ — основной фактор времени checkpoint'а |
| **Rescaling** | Тривиален — увеличить parallelism | Требует перераспределения state по новому числу subtask'ов; ограничен `maxParallelism` |
| **Latency на событие** | Минимальная | Зависит от state backend'а (HashMap < RocksDB) |
| **Когда использовать** | Преобразования, фильтры, обогащение через broadcast или async I/O | Окна, дедупликация, joins, паттерны, скользящие агрегаты, машины состояний |

## Когда состояние **обязательно**

| Задача | Почему нужен state |
|--------|-------------------|
| **Дедупликация событий** | Помнить уже виденные ID за окно (TTL'ом) |
| **Sessions / sessionization** | Накапливать события пользователя до тайм-аута неактивности |
| **Алертинг по последовательностям** | "3 неудачных входа за 5 минут" — счётчик + временное окно |
| **Joins двух потоков** | Буфер из обеих сторон в окне |
| **Обогащение из медленно меняющегося справочника** | Broadcast state с правилами/lookup table |
| **CDC материализация** | Latest value по ключу (changelog → table) |
| **Скользящие метрики** | sum/avg/percentile в окне |
| **Watermark generation per-partition** | Внутреннее состояние watermark assigner'а |

## Где **не нужен** state

| Задача | Решение |
|--------|---------|
| Парсинг JSON/Avro в типизированный объект | `map` |
| Фильтрация по полю | `filter` |
| Обогащение через внешний сервис per-event | `AsyncDataStream` (асинхронные lookup'ы), без state в Flink |
| Простой routing по типу события | `flatMap` + side outputs |

## Keyed vs non-keyed state

State доступен в двух режимах:

| | Keyed state | Operator state |
|--|-------------|----------------|
| **Поток** | После `keyBy` — `KeyedStream` | Любой `DataStream` |
| **Изоляция** | По ключу — каждый ключ имеет свой state | По параллельному инстансу subtask'а |
| **Rescaling** | Через **key groups** (логические партиции — переназначаются при изменении parallelism) | Через специальные стратегии: `even-split`, `union list state`, `broadcast` |
| **Применение** | 95% случаев: счётчики/окна/паттерны по ключу | Source connector'ы (offset'ы), Sink (pending транзакции), broadcast state |

Подробнее — в [Flink State Management](flink-state-management.md).

## Жизненный цикл state в stateful операторе

```
RuntimeContext → getState(...) — регистрация state в open()
event in → processElement → state.value() / state.update() / state.add()
checkpoint barrier → snapshot state в state backend → ack JobManager'у
restart → getState(...) → автоматически восстанавливается из последнего checkpoint'а
```

Регистрация state — обязательно в `open()` через `getRuntimeContext().getState(StateDescriptor)`. Если регистрировать в `processElement()` — каждое событие будет создавать новый descriptor (анти-паттерн).

## TTL для state

Без явного TTL state будет расти бесконечно. Flink поддерживает **State TTL**:

```java
StateTtlConfig ttl = StateTtlConfig
    .newBuilder(Time.hours(24))
    .setUpdateType(UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateVisibility.NeverReturnExpired)
    .cleanupIncrementally(100, true)
    .build();

ValueStateDescriptor<Long> desc = new ValueStateDescriptor<>("counter", Long.class);
desc.enableTimeToLive(ttl);
```

Стратегии очистки:
- **incremental** — на каждое чтение/запись чистится N записей (для HashMap state).
- **rocksdb compaction filter** — встраивается в фоновую компакцию RocksDB (для большого state).

Без TTL единственный способ удалить — явно `state.clear()` в логике.

## Типичные вопросы на интервью

**Q: Что значит «оператор stateful» и почему это важно?**
A: Stateful оператор помнит что-то между событиями (счётчики, окна, last value, паттерны). State хранится в **managed state** Flink (`ValueState`/`ListState`/`MapState`/...) — Flink его чекпоинтит, шардирует по key groups, восстанавливает при сбое и перераспределяет при rescaling. Это позволяет writing pipelines, переживающие падения без потери данных и без внешних БД для state.

**Q: В чём разница между обычным полем оператора и managed state?**
A: Поле класса (например, `private long counter`) живёт только в памяти JVM текущего subtask'а: оно **не чекпоинтится**, теряется при сбое, не перераспределяется при rescaling, и в keyed-операторах оно одно на все ключи (что обычно неверно). Managed state — это `ValueState<Long>` зарегистрированный через `getRuntimeContext().getState(...)`: Flink хранит его per-key, чекпоинтит и восстанавливает.

**Q: Какие операторы в Flink stateless, а какие stateful?**
A: **Stateless**: `map`, `flatMap`, `filter`, `union`, операции перепартицирования (`keyBy`/`rebalance`/`broadcast`). **Stateful**: `reduce`, `aggregate` после `keyBy`, любые `window` операции, `process`/`KeyedProcessFunction` если регистрирует state, `intervalJoin`, CEP паттерны, transactional sinks. Также source connector'ы хранят offset'ы — это тоже state.

**Q: Что такое keyed state и почему он лучше operator state в большинстве случаев?**
A: Keyed state шардируется автоматически: каждый ключ имеет свой инстанс state, события идут в правильный subtask через `hash(key) % parallelism`. Это даёт линейный scaling и простой rescaling через **key groups**. Operator state требует явных стратегий redistribution (even-split, union, broadcast) и применяется в основном в connector'ах (source offsets, sink pending transactions).

**Q: Как контролировать рост state?**
A: Главный инструмент — **State TTL**: автоматическое истечение записей по времени с настраиваемой стратегией очистки (incremental для heap, RocksDB compaction filter для on-disk). Также: явный `state.clear()` в логике (например, после закрытия окна), правильный выбор state backend'а (RocksDB для большого state), мониторинг размера через метрики и periodic savepoint'ы для аудита.

**Q: Можно ли использовать обычные коллекции (HashMap) вместо managed state?**
A: Технически да, но это **анти-паттерн**: HashMap в поле оператора не чекпоинтится, теряется при сбое, не перераспределяется при rescaling и в keyed-операторе один на все ключи (нужно вручную делать `Map<Key, Value>` — но тогда теряешь автоматический ключевой шардинг). Единственный валидный кейс — кэш read-only данных, который можно перестроить (например, перечитанные на старте справочники).

**Q: Что произойдёт со stateful job'ом, если я увеличу `parallelism` с 4 до 8?**
A: При перезапуске из savepoint'а (или checkpoint'а с `--allowNonRestoredState`) Flink перераспределит **key groups** между новыми 8 subtask'ами вместо 4 — каждый subtask теперь будет отвечать за половину прежнего количества ключей. Это работает прозрачно, **если** `maxParallelism` (количество key groups) ≥ нового parallelism. `maxParallelism` фиксируется на старте job'а, поэтому его выбирают с запасом (по умолчанию `max(128, ceil(parallelism * 1.5))`).

## Связи

- [Flink State Management](flink-state-management.md) — типы state, backends, key groups.
- [Flink Windows](flink-windows.md) — главный пример stateful операторов.
- [Flink ProcessFunction](flink-process-function.md) — как регистрировать и использовать state.
- [Flink Checkpoints and Savepoints](flink-checkpoints-and-savepoints.md) — как state переживает сбой.
- [Flink DataStream API](flink-datastream-api.md) — где живут эти операторы в API.
