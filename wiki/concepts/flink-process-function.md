# Flink ProcessFunction

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 6, 7)

## Суть

`ProcessFunction` и `KeyedProcessFunction` — low-level API Flink для логики, которую нельзя выразить простыми map/reduce/window операторами. Ключевые возможности: произвольный state, event-time/processing-time timers, side outputs, доступ к timestamp/watermark контексту.

## Когда нужен ProcessFunction

| Сценарий | Почему high-level API не хватает |
|----------|----------------------------------|
| Rule engine с тайм-аутами | нужны timers + per-key state machine |
| Дедупликация с TTL | нужен кастомный MapState и cleanup |
| Сложные последовательности событий | условные переходы между состояниями |
| Late data routing | side output по контексту watermark |

## Основной lifecycle

```java
public class FraudFn extends KeyedProcessFunction<String, Event, Alert> {
  private transient ValueState<Integer> fails;

  @Override
  public void open(Configuration parameters) {
    fails = getRuntimeContext().getState(
      new ValueStateDescriptor<>("fails", Integer.class));
  }

  @Override
  public void processElement(Event e, Context ctx, Collector<Alert> out) throws Exception {
    Integer n = fails.value();
    n = (n == null) ? 1 : n + 1;
    fails.update(n);
    ctx.timerService().registerEventTimeTimer(e.ts + 5 * 60_000L);
  }

  @Override
  public void onTimer(long ts, OnTimerContext ctx, Collector<Alert> out) throws Exception {
    Integer n = fails.value();
    if (n != null && n >= 3) out.collect(new Alert(ctx.getCurrentKey(), n));
    fails.clear();
  }
}
```

## Таймеры

| Таймер | Когда срабатывает | Назначение |
|--------|-------------------|------------|
| Event-time | когда watermark >= timer timestamp | бизнес-логика по времени событий |
| Processing-time | по wall clock TaskManager | тех. таймауты, housekeeping |

Важно: event-time timers детерминированы при replay; processing-time — нет.

## Типичные вопросы на интервью

**Q: Почему KeyedProcessFunction обычно предпочтительнее ProcessFunction?**  
A: Потому что даёт keyed state и timers per-key после `keyBy`, что нужно почти для любой business-логики. Обычный ProcessFunction без keyBy ограничен operator-state моделью и хуже масштабируется по ключам.

**Q: В чём риск processing-time timers?**  
A: Они привязаны к wall clock конкретного TaskManager, поэтому после restart/replay поведение может отличаться. Для бизнес-критичных правил и окон лучше event-time timers, синхронизированные watermark'ами.

**Q: Как избежать memory leak в ProcessFunction?**  
A: Всегда предусматривать очистку state: onTimer + TTL + clear при terminal condition. Иначе per-key state растёт бесконечно, checkpoint size увеличивается, recovery замедляется.

## Связи

- [Flink Stateful vs Stateless Operators](flink-stateless-vs-stateful-operators.md)
- [Flink State Management](flink-state-management.md)
- [Flink Time and Watermarks](flink-time-and-watermarks.md)
- [Flink Windows](flink-windows.md)
