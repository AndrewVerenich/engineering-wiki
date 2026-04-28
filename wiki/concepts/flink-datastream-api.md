# Flink DataStream API

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 5)

## Суть

**DataStream API** — основной программный интерфейс Flink: ты описываешь pipeline как последовательность преобразований над `DataStream<T>`. Flink собирает этот код в логический dataflow-граф (см. [Apache Flink Architecture](flink-architecture.md)), оптимизирует и распределяет по кластеру. Программа всегда строится из трёх частей: **source** (откуда читаем) → **transformations** (что делаем) → **sink** (куда пишем). Пока не вызван `env.execute()` — ничего не выполняется, только строится граф.

## Скелет программы

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(4);
env.enableCheckpointing(60_000);

DataStream<Event> source = env
    .fromSource(kafkaSource, WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(5)),
                "kafka-events");

DataStream<Alert> alerts = source
    .filter(e -> e.amount > 1000)
    .keyBy(Event::userId)
    .window(TumblingEventTimeWindows.of(Time.minutes(5)))
    .aggregate(new SumAggregator())
    .filter(sum -> sum.total > 10_000)
    .map(sum -> new Alert(sum.userId, sum.total));

alerts.sinkTo(kafkaSink);

env.execute("fraud-detection");
```

Каждый шаг возвращает `DataStream<T>`. Пока не вызван `execute()` — это просто построение графа в памяти драйвера.

## Sources

| Тип | Назначение | Гарантии |
|-----|-----------|----------|
| `KafkaSource` | Чтение из Kafka, основной production-источник | Exactly-once при checkpointing (offset'ы — часть state) |
| `FileSource` | Чтение файлов в bounded или streaming режиме (мониторинг новых файлов) | Exactly-once |
| `fromCollection`, `fromElements` | Тестовые/dev источники из коллекции в драйвере | n/a |
| `socketTextStream` | Чтение с TCP-сокета | At-most-once (без checkpointing offset'а) |
| Custom `SourceFunction` / новый `Source` API | Свои интеграции (HTTP-pull, gRPC stream, etc.) | Зависит от реализации |

**Новый `Source` API (FLIP-27)** разделяет **SplitEnumerator** (на JobManager — раздаёт партиции/файлы) и **SourceReader** (на TaskManager — читает данные). Это унифицированная модель batch+stream и lazy split discovery. Старый `SourceFunction` deprecated.

## Базовые трансформации (stateless)

| Оператор | Сигнатура | Применение |
|----------|-----------|-----------|
| `map` | `T → R` | Преобразование одного события в одно |
| `flatMap` | `T → Collection<R>` | Один → ноль/много (split, parse в несколько) |
| `filter` | `T → Boolean` | Отбрасывание |
| `keyBy(KeySelector)` | `DataStream<T> → KeyedStream<T, K>` | Перепартицирование по ключу. После — доступны keyed state и keyed windows |
| `union` | Объединение нескольких потоков **одного типа**. Без сортировки/упорядочивания | Слияние событий из разных Kafka-топиков |
| `connect` + `CoMap`/`CoFlatMap` | Соединение **двух разнотипных потоков** в `ConnectedStreams<A, B>`. Можно держать общий state | Подмешивание правил/конфигов в основной поток |

## Keyed-операции (stateful)

После `keyBy` поток становится `KeyedStream` и доступны:

| Оператор | Что делает |
|----------|-----------|
| `reduce(ReduceFunction)` | Инкрементальная свёртка по ключу. Хранит «текущее значение» в keyed state. |
| `aggregate(AggregateFunction)` | Обобщённая свёртка с разными типами accumulator'а и output'а. Эффективнее `reduce` для сложных агрегатов. |
| `process(KeyedProcessFunction)` | Низкоуровневый: доступ к keyed state, **timers** (event-time и processing-time), side outputs. См. [Flink ProcessFunction](flink-process-function.md). |
| `window(...)` | Группировка событий по времени/количеству. См. [Flink Windows](flink-windows.md). |

## Sinks

| Sink | Гарантии |
|------|----------|
| `KafkaSink` | At-least-once или **exactly-once** через transactional producer (двухфазный commit) |
| `FileSink` (Hadoop, S3) | Exactly-once через staging-файлы и атомарный commit при checkpoint'е |
| `JdbcSink` | At-least-once по умолчанию; exactly-once через **XA transactions** (если БД поддерживает) |
| `print()` | Только для отладки. Без гарантий |
| Custom `SinkFunction` / `Sink` API | Зависит от реализации; для exactly-once нужен **TwoPhaseCommitSinkFunction** |

См. подробнее в [Flink Exactly-Once Semantics](flink-exactly-once-semantics.md).

## Сериализация и типы

Flink не использует Java-сериализацию (медленная и хрупкая). У каждого `DataStream<T>` есть **TypeInformation** — либо выводится из generic'ов автоматически, либо задаётся явно через `.returns(Types.POJO(...))`.

| Тип | Сериализатор |
|-----|-------------|
| **POJO** (public, no-args constructor, getters/setters или public fields) | Эффективный собственный POJO serializer; поддерживает schema evolution на добавление/удаление полей |
| **Tuple** (`Tuple2<A, B>`, ..., `Tuple25`) | Самый быстрый, фиксированная структура |
| **Avro / Protobuf / Thrift** | Через подключаемые сериализаторы |
| **Kryo (fallback)** | Для сложных классов; **медленный**, без schema evolution. Стараются избегать |
| **Scala case class / Kotlin data class** | Tuple-подобный сериализатор через scala-extensions / явная регистрация |

**Type erasure pitfall:** в Java generic'и стираются. Если `flatMap` возвращает `DataStream<Tuple2<String, Integer>>` и Flink не может вывести тип — добавляй `.returns(TypeInformation.of(new TypeHint<Tuple2<String, Integer>>(){}))`.

## Side outputs

Способ выпустить из одного оператора **несколько разнотипных потоков**:

```java
OutputTag<String> lateTag = new OutputTag<>("late") {};

SingleOutputStreamOperator<Event> main = source.process(new ProcessFunction<>() {
    public void processElement(Event e, Context ctx, Collector<Event> out) {
        if (isLate(e)) ctx.output(lateTag, e.toString());
        else out.collect(e);
    }
});

DataStream<String> late = main.getSideOutput(lateTag);
```

Применение: late events в окнах, dead-letter queue, отдельный поток для метрик/аудита.

## Iterations и async I/O

| Возможность | Когда нужна |
|-------------|------------|
| `IterativeStream` | Циклические dataflow — итеративные алгоритмы (graph processing, ML). В streaming — редко. |
| `AsyncDataStream.unorderedWait` / `orderedWait` | Асинхронные внешние вызовы (HTTP, gRPC, БД lookup) без блокировки subtask'а. Важно для обогащения с lookup в внешний сервис. |

## Параллелизм

| Уровень | Как задаётся | Приоритет |
|---------|-------------|-----------|
| Глобальный | `env.setParallelism(N)` | Самый низкий |
| На оператор | `.map(...).setParallelism(N)` | Перекрывает глобальный |
| На source/sink | Часто стоит ставить ниже, чем на трансформации (Kafka source ограничен числом партиций) |
| `maxParallelism` | `env.setMaxParallelism(N)` | Верхняя граница для rescaling, нельзя изменить после старта job'а с тем же savepoint |

`maxParallelism` определяет количество **key groups** (см. [Flink State Management](flink-state-management.md)) — выбирается заранее с запасом (по умолчанию `max(128, ceil(parallelism * 1.5))`, но можно явно `2048` для job'ов с потенциалом сильного rescaling).

## Table API и SQL поверх DataStream

Flink также предоставляет:
- **Table API** — Java/Scala fluent DSL: `table.filter(...).groupBy(...).select(...)`.
- **SQL** — стандартный ANSI SQL поверх потоков с расширениями (`MATCH_RECOGNIZE`, windowed aggregations).

Под капотом — Apache Calcite парсит и оптимизирует, переводит в DataStream-операторы. Два API совместимы и часто смешиваются (низкоуровневая логика на DataStream, агрегаты на SQL).

## Типичные вопросы на интервью

**Q: Что делает `env.execute()` и почему без него ничего не работает?**
A: `execute()` отправляет построенный к этому моменту граф трансформаций в кластер. До этого все вызовы `map`/`filter`/`keyBy` только **строят StreamGraph в памяти драйвера** — они lazy. Без `execute()` граф просто останется неотправленным, job не стартует. Это аналогично lazy-evaluation в Spark.

**Q: В чём разница между `keyBy`, `rebalance` и `broadcast`?**
A: **`keyBy(K)`** — события с одинаковым ключом гарантированно попадают в один subtask следующего оператора (`hash(key) % parallelism`). Это даёт partitioning и keyed state. **`rebalance`** — round-robin между downstream subtask'ами, без привязки к ключу; используется для борьбы со skew. **`broadcast`** — каждое событие копируется во все downstream subtask'и; дорого, используется для рассылки правил/конфигов в broadcast state.

**Q: Зачем нужны side outputs и чем они отличаются от обычного второго `DataStream`?**
A: Side output позволяет выпустить из **одного** оператора несколько потоков **разных типов** — без необходимости делать второй проход или дублировать логику. Главные применения: late events в windowing (отдельный поток для опоздавших), dead-letter queue (события с ошибками валидации), потоки для аудита/метрик. Без side outputs пришлось бы дважды читать source или писать сложную flatMap.

**Q: В чём разница между ReduceFunction, AggregateFunction и ProcessFunction?**
A: **`ReduceFunction<T>`** — `(T, T) → T`, инкрементальная свёртка с одинаковым типом входа и выхода (например, max). **`AggregateFunction<IN, ACC, OUT>`** — отдельный тип accumulator'а, эффективнее для среднего/std/percentile (накапливаешь sum+count, отдаёшь sum/count). **`ProcessFunction`** — низкоуровневый API с доступом к state, timers, side outputs; использовать, когда нужна логика, не выражаемая через reduce/aggregate.

**Q: Что такое TypeInformation и зачем она?**
A: Flink не использует Java-сериализацию — у каждого потока должен быть известен тип, чтобы выбрать эффективный сериализатор. `TypeInformation` — это метаданные типа, которые Flink выводит из generic'ов или просит указать явно через `.returns(...)`. Без этого Flink упадёт в Kryo fallback (медленный, без schema evolution). Самые быстрые сериализаторы — Tuple и POJO.

**Q: Как правильно обогащать поток данными из внешней системы?**
A: Если данные **меняются редко** — broadcast state (CoFlatMap или KeyedBroadcastProcessFunction). Если данные **большие, но локализуемые по ключу** — keyed state с CDC-обновлениями через connect. Если данные **точечные lookup в БД/HTTP** — `AsyncDataStream.unorderedWait`, чтобы не блокировать subtask. Синхронный lookup в `map` — анти-паттерн: убивает throughput.

**Q: Что такое operator chaining и как им управлять?**
A: По умолчанию Flink сливает соседние операторы с одинаковым parallelism и forward-распределением в один runtime task — чтобы передавать данные без сериализации. Управление: `.disableChaining()` разрывает цепочку перед оператором; `.startNewChain()` начинает новую; `env.disableOperatorChaining()` отключает глобально. Используется для: точных метрик per-operator, изоляции тяжёлых операторов (с внешними вызовами), отладки.

## Связи

- [Apache Flink Architecture](flink-architecture.md) — куда отправляется граф из `execute()`.
- [Flink Stateless vs Stateful Operators](flink-stateless-vs-stateful-operators.md) — какие трансформации хранят state.
- [Flink Windows](flink-windows.md) — оконные операции на `KeyedStream`.
- [Flink ProcessFunction](flink-process-function.md) — низкоуровневое API для timers и side outputs.
- [Flink State Management](flink-state-management.md) — что лежит за `aggregate`/`reduce`/`process`.
- [Encoding and Schema Evolution](encoding-and-schema-evolution.md) — почему сериализация и эволюция схем важны.
