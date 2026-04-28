# Flink Windows

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 6)

## Суть

**Окно (window)** в Flink — это логический контейнер событий, ограниченный временем или условием, на котором выполняется агрегирование. В streaming нет «конца данных», поэтому без окна агрегат типа `COUNT(*)` бесконечен. Окна позволяют сказать: «посчитай метрику за 5 минут», «по сессии пользователя», «каждые 10 секунд за последние 1 минуту». Корректность окон по event time зависит от [watermark](flink-time-and-watermarks.md): окно закрывается, когда watermark проходит его конец.

## Базовая модель окна

Оконный оператор определяется 5 элементами:

| Элемент | Что задаёт |
|---------|------------|
| **Window assigner** | Какому окну принадлежит событие (tumbling/sliding/session/global) |
| **Key** | После `keyBy` окна считаются отдельно по каждому ключу |
| **Trigger** | Когда вычислять результат (при закрытии окна, по количеству, по времени, кастомно) |
| **Evictor** *(редко)* | Какие элементы удалить из окна перед/после вычисления |
| **Function** | Как считать результат (`reduce`, `aggregate`, `process`) |

Форма в API:

```java
stream
  .keyBy(e -> e.userId)
  .window(TumblingEventTimeWindows.of(Time.minutes(5)))
  .aggregate(new SumAgg(), new WindowResultFn());
```

## Типы окон

### Tumbling window

| Свойство | Значение |
|----------|----------|
| Размер | фиксированный (`size`) |
| Перекрытие | нет |
| Пример | `[12:00-12:05), [12:05-12:10), ...` |
| Когда использовать | классические time buckets: трафик/минуту, revenue/час |

Плюсы: простота, одна запись события в одно окно, легко масштабируется.  
Минусы: граничный эффект (события около границы попадают в разные окна).

### Sliding window

| Свойство | Значение |
|----------|----------|
| Размер | `size` |
| Шаг | `slide` |
| Перекрытие | да, если `slide < size` |
| Пример | size=10m, slide=1m → каждую минуту считаем последние 10 минут |
| Когда использовать | moving average, rate-of-change, скользящие SLI/SLO |

Плюсы: «гладкие» метрики, меньше скачков.  
Минусы: одно событие может попасть в много окон → рост state и CPU.

### Session window

| Свойство | Значение |
|----------|----------|
| Размер | динамический |
| Правило | новая сессия начинается после `sessionGap` тишины |
| Пример | `EventTimeSessionWindows.withGap(Time.minutes(30))` |
| Когда использовать | user sessionization, clickstream, fraud patterns |

Особенность: соседние окна могут **merge**-иться, если пришли события, заполняющие gap.

## Время окон: event vs processing time

| Режим | Когда окно закрывается | Риск |
|------|-------------------------|------|
| **Event time** | Когда `watermark >= window.end` | Нужен правильный watermark; late events |
| **Processing time** | По wall clock оператора | Недетерминированность при replay/restart |

Для бизнес-агрегатов почти всегда нужен event time.

## Функции вычисления окна

| Функция | Когда использовать | Память |
|---------|--------------------|--------|
| `reduce` | ассоциативные агрегаты с одинаковым IN/OUT (`sum`, `max`) | минимальная |
| `aggregate` | сложные агрегаты с accumulator (`avg`, custom stats) | минимальная |
| `process` (`ProcessWindowFunction`) | нужен metadata окна (`windowStart`, `windowEnd`), полный набор элементов, side output | высокая (часто буферизует больше) |
| `aggregate + ProcessWindowFunction` | часто лучший компромисс: инкрементальный агрегат + metadata | средняя |

Анти-паттерн: использовать только `ProcessWindowFunction` там, где достаточно `aggregate`.

## Triggers

По умолчанию для event-time окон trigger срабатывает один раз при закрытии окна watermark'ом.  
Можно задать кастомный trigger:

| Trigger | Применение |
|---------|------------|
| `EventTimeTrigger` | стандарт для event-time |
| `ProcessingTimeTrigger` | периодические early results |
| `CountTrigger` | «каждые N событий» |
| Custom trigger | early + final firing, сложные правила |

Практика: early results + final correction для UX-дэшбордов.

## Allowed lateness и late events

```java
OutputTag<Event> lateTag = new OutputTag<>("late") {};

stream
  .keyBy(Event::userId)
  .window(TumblingEventTimeWindows.of(Time.minutes(5)))
  .allowedLateness(Time.minutes(2))
  .sideOutputLateData(lateTag)
  .aggregate(new SumAgg());
```

| Механизм | Что делает |
|----------|------------|
| `allowedLateness` | окно остаётся «живым» ещё N после nominal close |
| Late within lateness | окно пересчитывается, результат эмиттится повторно |
| Too-late | уходит в side output или отбрасывается |

Downstream должен быть готов к update/ретракциям (upsert sink, dedup key).

## Window joins

| Join | Что хранится в state | Когда закрывается |
|------|----------------------|-------------------|
| Window join | буферы обеих сторон внутри окна | при watermark >= window.end |
| Interval join | буфер событий по диапазону `[t-a, t+b]` | по watermark верхней границы |

Главный риск — state blow-up при большом окне и высокой кардинальности ключей.

## Производительность и anti-patterns

| Проблема | Причина | Что делать |
|----------|---------|------------|
| Очень большой state | sliding window с маленьким шагом | увеличить шаг, перейти на incremental aggregation |
| Окна «не закрываются» | watermark застрял на idle партиции | `withIdleness(...)`, проверка `currentInputWatermark` |
| Дубликаты результатов | allowed lateness + repeated firing | idempotent sink / upsert semantics |
| Высокая latency | слишком большой out-of-orderness | подобрать bound по p99 задержек |

## Типичные вопросы на интервью

**Q: Чем tumbling, sliding и session windows отличаются на практике?**  
A: Tumbling — фиксированные неперекрывающиеся окна, событие попадает ровно в одно окно; дешёвый и понятный выбор для отчётности. Sliding — окна перекрываются (`size > slide`), одно событие участвует в нескольких окнах; лучше для «скользящих» метрик, но дороже по state/CPU. Session — динамический размер, окно закрывается после периода неактивности; идеален для user behavior, но сложнее для capacity planning, потому что размер окна заранее неизвестен.

**Q: Почему в production чаще используют `aggregate`, а не `ProcessWindowFunction`?**  
A: `aggregate` и `reduce` инкрементальные: Flink хранит accumulator, а не все события окна — меньше память и GC pressure. `ProcessWindowFunction` часто вынуждает держать больше данных и обходить элементы окна целиком. Его включают, когда нужна metadata окна (`window start/end`), side outputs или нестандартная логика emit.

**Q: Что происходит с late events в event-time окнах?**  
A: После закрытия окна watermark'ом событие с `eventTime < currentWatermark` считается late. Если задан `allowedLateness`, окно может быть пересчитано и результат отправлен повторно (update). Если событие опоздало сильнее lateness, оно уходит в side output (`sideOutputLateData`) или отбрасывается. Поэтому sink должен поддерживать upsert/идемпотентность.

**Q: Когда processing-time окна допустимы?**  
A: Когда важна минимальная latency и допустима недетерминированность: технические дэшборды, оперативные алерты, internal observability. Для финансовых расчётов, биллинга, лимитов, продуктовой аналитики processing-time опасен — replay/restart даст иные результаты.

**Q: Как выбрать размер окна и шаг для sliding window?**  
A: Исходить из бизнес-метрики и стоимости. `size` определяет «контекст» (например, 10 минут для rate), `slide` — частоту обновления. Стоимость примерно пропорциональна `size/slide`: чем меньше шаг, тем больше окон на событие. Часто начинают с `size=10m, slide=1m` и смотрят CPU/state, затем уменьшают частоту обновления до приемлемой.

## Связи

- [Flink Time and Watermarks](flink-time-and-watermarks.md) — когда окно закрывается.
- [Flink Stateless vs Stateful Operators](flink-stateless-vs-stateful-operators.md) — окна как stateful операторы.
- [Flink ProcessFunction](flink-process-function.md) — low-level альтернативы окнам.
- [Flink State Management](flink-state-management.md) — где хранится window state.
- [Stream Processing](stream-processing.md) — общая теория windowing.
