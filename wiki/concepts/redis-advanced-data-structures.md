# Redis Advanced Data Structures

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (data types, streams, probabilistic structures)

## Суть

Базовые структуры Redis (String/Hash/List/Set/ZSet) часто знают на уровне junior/middle. Senior-уровень начинается там, где ты понимаешь **внутреннее представление, асимптотику и стоимость в памяти** и можешь выбрать структуру под конкретный SLA по latency/accuracy/cost.

Эта страница дополняет обзор [Redis Data Structures and Modeling](redis-data-structures-and-modeling.md) и фокусируется на deep-dive для production-инженерии.

## Sorted Set internals

| Свойство | Деталь | Практический эффект |
|----------|--------|---------------------|
| Внутренняя модель | skiplist + hashtable | быстрый rank/range + O(1) lookup member |
| Типовые операции | `ZADD`, `ZRANGE`, `ZRANGEBYSCORE` | обычно O(log N) + размер выборки |
| Сценарии | leaderboard, delay queue, sliding windows | универсальный индекс по score/time |

## HyperLogLog

| Параметр | Значение |
|----------|----------|
| Точность | ~0.81% относительной ошибки |
| Память | около 12 KB на key |
| Команды | `PFADD`, `PFCOUNT`, `PFMERGE` |

Use-case: approximate distinct count (DAU/MAU/UV), когда важна компактность, а не абсолютная точность.

## Bitmaps и Bitfields

| Инструмент | Для чего | Trade-off |
|------------|----------|-----------|
| Bitmap (`SETBIT`, `BITCOUNT`) | флаги и presence по ID/time | очень компактно, но нужен стабильный mapping |
| Bitfield (`BITFIELD`) | packed counters/flags | контроль memory layout, но сложнее читать/поддерживать |

## Geo (на базе ZSet)

Geo-команды используют geohash-индексацию поверх sorted set:

- `GEOADD` для записи координат;
- `GEOSEARCH`/`GEORADIUS` для поиска рядом;
- хорошо подходит для «ближайшие N объектов» при умеренной точности.

## Streams и consumer groups

| Компонент | Роль |
|-----------|------|
| `XADD` | append-only запись событий в stream |
| `XREADGROUP` | чтение группой потребителей |
| PEL (Pending Entries List) | трекинг доставленных, но не подтверждённых сообщений |
| `XACK` | подтверждение обработки |
| `XAUTOCLAIM` | re-claim «зависших» сообщений после таймаута |

Ключевая идея: Streams добавляют persisted messaging semantics поверх Redis, но это не полный аналог Kafka-партиций и экосистемы.

## Матрица выбора структуры

| Задача | Лучшая структура | Почему |
|--------|------------------|--------|
| Top-N / rank / score | ZSet | порядок + range/rank запросы |
| Distinct users estimate | HyperLogLog | фиксированный маленький memory footprint |
| Feature flags по user-id | Bitmap/Bitfield | плотная упаковка битов |
| Nearby search | Geo | geohash индекс из коробки |
| Reliable background processing | Streams + consumer groups | pending tracking + ack |

## Типичные вопросы на интервью

**Q: Почему ZSet реализован через skiplist + hashtable, а не одной структурой?**  
A: Hashtable даёт быстрый доступ к member для update/delete, skiplist даёт упорядоченный обход и range по score. Комбинация закрывает оба класса операций без полного сканирования.

**Q: Когда HyperLogLog нельзя использовать?**  
A: Когда нужна точная cardinality (финансовая отчётность, billing per unique event) или когда погрешность ~0.81% неприемлема. Тогда выбирают точные структуры (Set/внешняя аналитика) и платят памятью.

**Q: Streams в Redis заменяют Kafka?**  
A: Частично, для небольших/средних workloads и простой архитектуры. Но у Kafka сильнее масштабирование по partition log, replay-экосистема и долговременный retention. Redis Streams удобнее как встроенный broker в backend-контур, когда нужна низкая операционная сложность.

**Q: Почему bitmap может быть опасным при неверном дизайне?**  
A: Потому что размер bitmap определяется максимальным offset. Если использовать sparse огромные ID без mapping/compaction, можно не заметить memory blow-up.

## Связи

- [Redis Data Structures and Modeling](redis-data-structures-and-modeling.md)
- [Redis Pub/Sub and Streams](redis-pubsub-and-streams.md)
- [Redis](../entities/redis.md)
- [Data Models and Query Languages](data-models-and-query-languages.md)
