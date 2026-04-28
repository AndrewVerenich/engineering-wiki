# Flink State Management

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 7)

## Суть

В Flink состояние — встроенная часть runtime, а не «самописный HashMap в операторе». Это даёт ключевые свойства production-streaming: state автоматически сохраняется в checkpoint'ах, восстанавливается после падений и перераспределяется при rescaling. Главные вопросы: какой тип state выбрать, где он хранится (backend), как контролировать рост (TTL), и как избежать hot key/state blow-up.

## Типы state

| Тип | Где доступен | Для чего |
|-----|--------------|----------|
| `ValueState<T>` | только после `keyBy` | одно значение на ключ (последний статус, счётчик) |
| `ListState<T>` | keyed/operator | список значений (буфер событий, union state) |
| `MapState<K,V>` | keyed | ассоциативный state (дедуп по eventId, per-feature stats) |
| `ReducingState<T>` | keyed | инкрементальная свёртка с ReduceFunction |
| `AggregatingState<IN,OUT>` | keyed | кастомный accumulator и output |
| `BroadcastState<K,V>` | broadcast side в `KeyedBroadcastProcessFunction` | правила, справочники, конфиги |

## Keyed state vs operator state

| | Keyed state | Operator state |
|--|-------------|----------------|
| Разделение | по key groups | по subtask |
| Rescaling | автоматический ребаланс key groups | explicit redistribution strategy |
| Применение | 90% бизнес-логики | source/sink connector internals |

## State backends

| Backend | Где лежит state | Плюсы | Минусы |
|---------|------------------|-------|--------|
| **HashMapStateBackend** | JVM heap | низкая latency, просто | ограничен heap, большие checkpoint'ы |
| **EmbeddedRocksDBStateBackend** | off-heap + локальный диск | огромный state (десятки/сотни ГБ), incremental checkpoints | выше latency на доступ, сложнее тюнинг |

Практика:
- маленький state и latency-критично -> HashMap;
- большой state и стабильность при росте -> RocksDB.

## Key groups и rescaling

Keyed state делится на фиксированное число **key groups** (`maxParallelism`).  
Subtask получает набор key groups; при изменении `parallelism` mapping меняется.

Почему важно:
- `maxParallelism` задаётся заранее и ограничивает будущий scaling;
- слишком маленький -> упираешься в scaling ceiling;
- слишком большой -> больше metadata overhead.

## TTL и очистка state

Без TTL state растёт бесконечно.

```java
StateTtlConfig ttl = StateTtlConfig
  .newBuilder(Time.days(7))
  .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
  .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
  .build();
```

Рекомендации:
- всегда ставить TTL, если логика это допускает;
- не полагаться только на cleanup during read — делать явный `state.clear()` на terminal state;
- мониторить размер state и время checkpoint.

## Типичные вопросы на интервью

**Q: Почему нельзя хранить состояние в полях класса оператора?**  
A: Такое состояние не managed Flink'ом: оно не входит в checkpoint, теряется при падении и не перераспределяется при rescaling. Managed state (`ValueState`, `MapState`...) автоматически snapshot/recovery/rescale-safe.

**Q: Когда выбирать RocksDB backend?**  
A: Когда объём keyed state уже не помещается в heap или ожидается рост: высококардинальные ключи, длинные окна, dedup по большим периодам, interval joins. RocksDB даёт масштаб по диску и incremental checkpoints, но с ценой в latency.

**Q: Что такое key groups и зачем нужен `maxParallelism`?**  
A: Key groups — логические партиции ключевого пространства. `maxParallelism` задаёт их количество и тем самым верхнюю границу для будущего parallelism. При rescaling Flink перекидывает key groups между subtask'ами, поэтому состояние не теряется.

**Q: Как бороться с ростом state?**  
A: TTL + clear terminal state + уменьшение окна/retention + борьба с hot keys + переход на приблизительные структуры (например, HyperLogLog, Bloom) там, где приемлема оценка. Также проверять skew и не держать «вечные» ключи без бизнес-необходимости.

## Связи

- [Flink Stateless vs Stateful Operators](flink-stateless-vs-stateful-operators.md)
- [Flink Checkpoints and Savepoints](flink-checkpoints-and-savepoints.md)
- [Flink Windows](flink-windows.md)
- [Flink ProcessFunction](flink-process-function.md)
