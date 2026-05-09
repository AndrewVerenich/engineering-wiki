# Kafka Streams Time and Stream Time

**Источник:** [Kafka Streams in Action, Second Edition](../sources/kafka-streams-in-action.md)

## Суть

Kafka Streams опирается на timestamp каждого record и продвигает время обработки через понятие **stream time** (максимальный observed event-time в task). Это влияет на закрытие окон, punctuations и обработку late events.

## Модели времени

| Модель | Что означает |
|--------|--------------|
| **Event time** | Когда событие произошло в бизнес-домене. |
| **Processing time** | Когда запись была обработана приложением. |
| **Ingestion time** | Когда запись попала в Kafka (зависит от producer/broker config). |
| **Stream time** | Локальный прогресс event-time внутри task (обычно max seen timestamp). |

## TimestampExtractor

- Отвечает за то, как извлекается timestamp из record.
- Неправильный extractor ломает окна и join semantics.
- Частый production-паттерн: fallback на metadata timestamp, если payload timestamp невалиден.

## Kafka Streams vs Flink watermarks

- Kafka Streams не использует отдельный watermark protocol как Flink.
- Прогресс времени выводится из потока данных task (stream time).
- Это упрощает модель, но требует аккуратной работы с late data и равномерностью входного потока.

## Типичные вопросы на интервью

**Q: Что такое stream time в Kafka Streams?**  
A: Это локальный прогресс event-time для task, основанный на уже увиденных событиях. Он управляет срабатыванием некоторых time-based операций.

**Q: Чем это отличается от watermarks во Flink?**  
A: Во Flink watermark — отдельный сигнал в dataflow. В Kafka Streams прогресс времени выводится из самих событий, без отдельного протокола watermark.

**Q: Почему неправильный TimestampExtractor опасен?**  
A: Он может сдвинуть события в неверные окна, создать ложные late events и нарушить корректность агрегаций и joins.

**Q: Когда processing time всё ещё полезен?**  
A: Для operational-триггеров и задач, где бизнес-время не критично. Но аналитические метрики обычно должны опираться на event time.

## Связи

- [Kafka Streams Windowing](kafka-streams-windowing.md) — окна и late events.
- [Kafka Streams Processor API](kafka-streams-processor-api.md) — stream-time punctuations.
- [Flink Time and Watermarks](flink-time-and-watermarks.md) — альтернативная time-модель.
- [Stream Processing](stream-processing.md) — теория event time и late data.
