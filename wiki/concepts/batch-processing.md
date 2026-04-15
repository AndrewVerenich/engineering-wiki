# Batch Processing

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 10)

## Суть

**Пакетная обработка** — обход больших наборов данных офлайн (MapReduce, распределённые join’ы, сортировки). Отличие от OLTP: высокая пропускная способность, высокая задержка, **идемпотентность** и **recomputation** как паттерны. Связь с **dataflow** и материализацией производных наборов.

## Ключевые концепции

| Концепция | Суть |
|-----------|------|
| **MapReduce** | Два этапа: Map (извлечь ключ-значение) → Shuffle (группировка по ключу) → Reduce (агрегация). Параллелизм по partition. Kleppmann: «Unix pipes для распределённых систем». |
| **Sort-merge join** | Обе стороны сортируются по ключу → merge-pass. Эффективен для больших данных (sequential I/O). В Spark — SortMergeJoin. |
| **Broadcast join** | Маленькая таблица рассылается на все executor'ы → join без shuffle. В Spark: `broadcast()`. |
| **Idempotent recomputation** | При сбое — перезапуск задачи с теми же входами даёт тот же результат. Нет side-effects → простой retry, без координации. |
| **Materialization** | Промежуточные результаты пишутся на диск (HDFS/S3) → следующий этап читает. Fault-tolerant, но I/O overhead. В Spark: RDD lineage + checkpoint. |

## MapReduce vs Spark

| | **MapReduce** | **Spark** |
|---|---------------|-----------|
| **Модель** | Map → Reduce, запись промежуточных на диск | DAG операторов, in-memory pipeline |
| **Производительность** | Медленнее (disk I/O между этапами) | Быстрее (memory, pipelining) |
| **Fault tolerance** | Перезапуск map/reduce task | RDD lineage → recomputation потерянных partitions |
| **API** | Низкоуровневый (Java) | Высокоуровневый (DataFrame, SQL, Catalyst optimizer) |
| **Сейчас** | Legacy | De-facto стандарт для batch |

## Типичные вопросы на интервью

**Q: Чем batch отличается от stream processing?**
A: **Batch** — обработка ограниченного (bounded) набора данных; высокая пропускная способность, высокая latency (минуты-часы). **Stream** — обработка неограниченного (unbounded) потока; низкая latency (секунды), но сложнее гарантии (ordering, exactly-once). На практике часто комбинируются (Lambda/Kappa architecture).

**Q: Как Spark обеспечивает fault tolerance?**
A: Через **RDD lineage**: каждый RDD помнит, как был вычислен из предыдущего. При потере partition — пересчитывает из lineage. Для длинных цепочек — **checkpoint** (запись на диск) обрывает lineage и ускоряет recovery.

**Q: Что такое data skew и как с ним бороться в Spark?**
A: Одна partition получает непропорционально много данных → один executor работает дольше всех. Решения: salted key (добавить random suffix, потом re-aggregate), broadcast join для маленькой стороны, adaptive query execution (AQE) в Spark 3+.

**Q: Почему idempotency важна в batch?**
A: Batch job может упасть на середине. При перезапуске нужно, чтобы повторная обработка не создала дубликаты и не испортила данные. Паттерн: overwrite output partition целиком (не append), или DROP PARTITION + reload в DWH.

**Q: Когда batch, а когда stream?**
A: Batch — когда latency часов допустима и важна throughput (ETL, отчёты, тренировка ML). Stream — когда нужна реакция в реальном времени (мониторинг, fraud detection, live dashboards). Часто оба: stream для fast path, batch для корректировки и full recompute.

## Связи

- [Derived Data and Systems](derived-data-and-systems.md) — batch как источник производных представлений.
- [Stream Processing](stream-processing.md) — контраст и объединение (lambda/kappa — в derived-data).
- [OLTP vs OLAP](../comparisons/oltp-vs-olap.md) — OLAP и аналитические batch-нагрузки.
