# Flink Time and Watermarks

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 2, 6)

## Суть

В streaming-системе **«сейчас» — это не одно значение**. Существуют как минимум три разные шкалы времени, и от того, какую ты выбрал, зависит корректность результата. Flink — один из немногих движков, где **event time** — first-class citizen: окна закрываются по timestamp'ам **в самих событиях**, а не по wall clock сервера. Чтобы знать, когда «достаточно подождали» опоздавших событий, Flink использует **watermarks** — специальные служебные сообщения, которые движутся по pipeline и отвечают на вопрос «можно ли уже считать, что все события до момента T пришли?». Watermark = эвристика. Слишком ранний → потеря late events. Слишком поздний → высокая задержка результата.

## Три типа времени

| Тип | Что это | Источник | Применение |
|-----|--------|----------|------------|
| **Event time** | Когда событие реально произошло (timestamp в данных) | Поле в самом событии (например, `clickedAt`) | **По умолчанию для корректных результатов**: правильное окно даже при reordering, late events, restarts, replays |
| **Processing time** | Когда оператор Flink увидел событие (его wall clock) | `System.currentTimeMillis()` в момент обработки | Когда нужна минимальная latency и не критична детерминированность (мониторинг, грубые алерты) |
| **Ingestion time** *(redundant since 1.12)* | Когда событие вошло в Flink (присвоено source'ом) | Wall clock у source-оператора | Компромисс: дешевле event time, корректнее processing time. На практике почти не используется |

### Почему event time важен

Сценарий: считаем число событий за минутный tumbling window.

| Время в источнике (event time) | Реальное прибытие (processing time) | Окно по event time | Окно по processing time |
|-------------------------------|-------------------------------------|--------------------|-----------------------|
| 12:00:30 | 12:00:31 | 12:00–12:01 | 12:00–12:01 |
| 12:00:55 | 12:01:08 (мобильный буферизовал) | 12:00–12:01 ✓ | 12:01–12:02 ✗ |
| 12:00:59 | 12:00:59 | 12:00–12:01 | 12:00–12:01 |

При rescaling, перезапуске, обработке исторических данных — processing time даёт **разные результаты** на одних и тех же входных данных. Event time — **детерминированный**.

## Watermark: определение и инвариант

**Watermark `W(t)`** — служебное сообщение в потоке, которое говорит downstream-операторам: **«события с event time ≤ t больше не появятся (или их можно проигнорировать)»**.

Инварианты:
- Watermark'и **монотонно неубывают** в каждом потоке.
- Watermark не события: они движутся между событиями как маркеры.
- Каждый event-time оператор хранит **current watermark** = последний полученный watermark.
- Окна закрываются, когда `watermark >= window.end`.

Если по pipeline идут события с event time `[10, 9, 12, 8]` и watermark `W(11)`, это значит: «8, 9 — точно в прошлом, окно `[0, 10]` можно закрывать». Если потом придёт событие с event time `7` — это **late event**.

## Как генерируются watermarks

Watermark — это **обещание**, которое делает source или watermark assigner. Самые частые стратегии:

| Стратегия | Идея | Когда использовать |
|-----------|------|---------------------|
| **`forMonotonousTimestamps()`** | watermark = (max event time seen) - 1ms; событий «из будущего» относительно watermark не бывает | Идеально упорядоченный поток (например, single-partition Kafka с гарантированным порядком записи) |
| **`forBoundedOutOfOrderness(Duration.ofSeconds(N))`** | watermark = (max event time seen) - N | **Стандарт**: разрешает разупорядочивание в пределах N секунд. N выбирается по 99-95 перцентилю реального jitter'а |
| **Пользовательская `WatermarkGenerator`** | Любая логика, например: периодические watermark на основе квантиля задержек | Сложные кейсы, переменный jitter, адаптивные стратегии |

```java
WatermarkStrategy<Event> strategy = WatermarkStrategy
    .<Event>forBoundedOutOfOrderness(Duration.ofSeconds(5))
    .withTimestampAssigner((event, ts) -> event.eventTime)
    .withIdleness(Duration.ofMinutes(1));

DataStream<Event> stream = env.fromSource(kafkaSource, strategy, "events");
```

`withTimestampAssigner` — извлечение timestamp'а из события (если source их сам не присваивает). `withIdleness` — обработка idle source'ов (см. ниже).

## Распространение watermark в графе

Watermark вычисляется отдельно для каждой партиции потока, а потом сливается:

| Узел графа | Что делает с watermark |
|-----------|------------------------|
| **Source** | Генерирует watermark per-partition (Kafka — per-partition, файлы — per-split) |
| **Один input оператор (`map`, `filter`)** | Прокидывает watermark дальше as-is |
| **Несколько входов (`union`, `connect`, `join`)** | Берёт **минимум** входных watermark'ов — иначе нарушится инвариант на slowest input |
| **Параллельные partition'ы downstream** | После shuffle: оператор собирает watermark от всех upstream subtask'ов и берёт минимум |

**Watermark = bottleneck-слайдинг.** Если хотя бы одна Kafka-партиция отстаёт — watermark всего job'а застревает на её последнем event time. Это самая частая причина «окна не закрываются».

## Idle source / idle partition

Если одна Kafka-партиция **не получает событий**, watermark'а от неё нет → downstream берёт min → весь job замораживается на её последнем времени.

**Решение:** `withIdleness(Duration.ofMinutes(N))`. Если партиция молчит >N минут, она помечается **idle** и её watermark игнорируется при вычислении минимума. Когда события возобновятся — она выходит из idle и снова влияет на минимум.

Это не серебряная пуля: late events с idle-партиции, которые опоздали относительно watermark'а от других партиций, будут отброшены.

## Late events: что с ними делать

Когда событие приходит с `eventTime < currentWatermark`, оно **late**. Стратегии:

| Стратегия | API | Trade-off |
|-----------|-----|-----------|
| **Отбросить (default)** | Ничего не делать | Потеря данных, корректные окна |
| **Allowed lateness** | `.window(...).allowedLateness(Time.minutes(10))` | Окно остаётся открытым ещё N минут после watermark; результат пересчитывается и **повторно эмиттится** (downstream должен это уметь обработать) |
| **Side output для late events** | `.sideOutputLateData(lateTag)` | Late events идут в отдельный поток для retroactive processing / аудита |
| **Подвинуть watermark позже** (`forBoundedOutOfOrderness(BIG)`) | Ждать дольше | Меньше late events, но большая задержка результата |

Чаще всего комбинируют: умеренный allowed lateness + side output для крайне опоздавших.

## Watermark в join'ах

| Тип join | Watermark |
|----------|-----------|
| **Window join** (`stream1.join(stream2).where(...).equalTo(...).window(...)`) | Окно закрывается по min(W1, W2). Состояние обоих потоков буферизуется до закрытия. |
| **Interval join** (`stream1.intervalJoin(stream2).between(-5min, +5min)`) | Каждое событие из stream1 ждёт парные события из stream2 в окне `[t-5, t+5]`. State чистится, когда watermark проходит верхнюю границу. |
| **Temporal table join** (lookup в таблицу-как-стрим) | Lookup происходит на момент event time; обе стороны должны иметь watermark |

## Trade-off ключевого параметра — задержка watermark'а

Это главный tuning-knob streaming-приложения:

| `outOfOrdernessBound` | Late events | Latency окна |
|------------------------|-------------|--------------|
| **0 секунд** (`forMonotonousTimestamps`) | Любое разупорядочивание = late | ~0 |
| **5 секунд** | Опоздавшие >5с — late | +5с к закрытию окна |
| **30 секунд** | Опоздавшие >30с — late | +30с |
| **5 минут** | Почти ничего не late | +5 мин — может быть неприемлемо |

**Как выбирать:** замерить реальный jitter `processingTime - eventTime` на прод-данных, взять 99-перцентиль и добавить запас 20-50%. Для критичных бизнес-окон (биллинг) — 99.9-перцентиль.

## Watermark vs checkpoint barrier

Часто путают, но это **разные служебные сообщения**:

| | Watermark | Checkpoint barrier |
|--|-----------|-------------------|
| **Что говорит** | «события до t уже все/можно игнорировать» | «сделай snapshot своего state прямо сейчас» |
| **Кто инициирует** | Source / WatermarkGenerator | JobManager (CheckpointCoordinator) |
| **Распространение** | Broadcast → multi-input берёт min | Aligned (по умолчанию) — оператор ждёт barrier на всех входах |
| **Связь с временем** | Только event time | Не связан со временем событий — это «контрольная точка» процесса |

См. [Flink Checkpoints and Savepoints](flink-checkpoints-and-savepoints.md).

## Типичные вопросы на интервью

**Q: Чем event time отличается от processing time и какой выбрать?**
A: **Event time** — timestamp в самих событиях (когда они произошли). **Processing time** — wall clock оператора в момент обработки. Event time даёт **детерминированные** результаты при reordering, перезапусках, replay'ях исторических данных — это нужно для корректности. Processing time даёт минимальную latency и используется для грубого мониторинга/алертов, где допустима недетерминированность. По умолчанию в production — event time.

**Q: Что такое watermark и зачем он нужен?**
A: Watermark `W(t)` — служебное сообщение в потоке, означающее «события с event time ≤ t больше не появятся (или будут считаться опоздавшими)». Это **эвристика**, а не факт — реальные late events могут прийти. Watermark позволяет окнам по event time **закрыться**: иначе непонятно, когда считать, что все события за минуту собраны. Без watermark event time semantics не работает.

**Q: Как генерируется watermark в Flink и какой главный параметр настраивается?**
A: Через `WatermarkStrategy`. Стандартная стратегия — `forBoundedOutOfOrderness(Duration.ofSeconds(N))`: watermark = (max event time seen) - N. Параметр **N** (out-of-orderness bound) — главный trade-off: малый N → быстрое закрытие окон, но больше late events; большой N → меньше late events, но высокая latency. Подбирается по 99-перцентилю реального jitter'а на прод-данных + запас 20-50%.

**Q: Почему watermark в multi-input операторе берётся как минимум?**
A: Чтобы сохранить инвариант «все события до t уже обработаны». Если один из входов отстаёт (его watermark = T1), а другой ушёл вперёд (T2 > T1), оператор не может обещать «все события до T2 пришли» — со стороны медленного входа ещё могут прийти события с временем между T1 и T2. Поэтому берётся `min(T1, T2)`. Это означает: **watermark тянется к самому медленному источнику**.

**Q: У меня окно не закрывается, что проверить в первую очередь?**
A: 1) **Idle Kafka-партиция**: одна партиция не получает событий → её watermark = -∞ → min всех watermark'ов застрял. Решение: `withIdleness(...)`. 2) **Слишком большой `outOfOrdernessBound`** — окно `[0, 60s]` закроется только при watermark ≥ 60s, т.е. когда max event time будет ≥ `60s + N`. 3) **Skew между параллельными source'ами** — один subtask читает быстрее. 4) Проверить метрику `currentInputWatermark` в Flink UI на каждом операторе.

**Q: Как правильно обрабатывать late events?**
A: По бизнес-критичности: (a) **Отбросить** (default) — если потеря единичных опоздавших не страшна. (b) **Allowed lateness** — окно остаётся открытым ещё N после watermark, результат может пересчитываться и переэмиттиться (downstream должен поддерживать update). (c) **Side output** — late events в отдельный поток для аудита или batch-доагрегации. На практике часто комбинируют: малый `allowedLateness` (1-5 мин) + side output для очень опоздавших.

**Q: Что произойдёт, если watermark начнёт идти назад?**
A: Это **запрещённое** состояние, watermark монотонно не убывает. Если WatermarkGenerator выдал watermark `W(t1)` после `W(t2 > t1)`, второй просто будет проигнорирован — Flink не пропустит регрессию. Поэтому при реализации custom watermark generator'а важно следить за монотонностью и обычно поддерживать `maxSeenTimestamp` как state.

**Q: Можно ли использовать processing time для окон в production?**
A: Можно, и иногда нужно — для технического мониторинга, алертов на тренды, метрик системы (latency, throughput) — где не важна корректность относительно event time. Но для бизнес-агрегатов (биллинг, лимиты транзакций, отчёты) processing time опасен: после рестарта или replay'а получишь другие результаты, при backfill — все события улетят в одно окно, нельзя воспроизвести инцидент.

## Связи

- [Flink Windows](flink-windows.md) — окна, которые закрываются по watermark.
- [Flink ProcessFunction](flink-process-function.md) — event-time таймеры на основе watermark.
- [Flink Checkpoints and Savepoints](flink-checkpoints-and-savepoints.md) — barrier'ы vs watermark.
- [Flink Exactly-Once Semantics](flink-exactly-once-semantics.md) — почему event time помогает детерминированности.
- [Stream Processing](stream-processing.md) — DDIA-теория event time, late events.
- [Apache Kafka](../entities/apache-kafka.md) — partition-aware watermark generation.
