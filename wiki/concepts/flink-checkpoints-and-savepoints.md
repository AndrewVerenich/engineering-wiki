# Flink Checkpoints and Savepoints

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 7, 10)

## Суть

Fault tolerance во Flink строится на **asynchronous distributed snapshots**: система периодически фиксирует consistent state всех операторов и source offsets. Если TaskManager падает, job рестартуется и продолжает с последнего успешного snapshot.  
`Checkpoint` — автоматический механизм для recovery.  
`Savepoint` — ручной snapshot для controlled operations (deploy/upgrade/migration).

## Checkpoint

| Свойство | Значение |
|----------|----------|
| Кто запускает | JobManager (CheckpointCoordinator) |
| Частота | `execution.checkpointing.interval` |
| Назначение | автоматический recovery |
| Содержимое | operator state + keyed state + source positions |
| Жизненный цикл | retention ограничен политикой |

Ключевые настройки:
- `minPauseBetweenCheckpoints`
- `checkpointTimeout`
- `maxConcurrentCheckpoints`
- `tolerableCheckpointFailureNumber`

## Savepoint

| Свойство | Значение |
|----------|----------|
| Кто запускает | вручную (CLI/REST/UI) |
| Назначение | upgrade/rescale/migration |
| Формат | переносимый, стабильный между версиями |
| Удаление | вручную по решению оператора |

Команда (пример): `flink savepoint <jobId> s3://.../savepoints`.

## Checkpoint vs Savepoint

| | Checkpoint | Savepoint |
|--|-----------|-----------|
| Авто/ручной | авто | ручной |
| Частота | регулярно | по операции |
| Оптимизация | может быть incremental | чаще полный, переносимый |
| Цель | fast recovery | planned changes |

## Exactly-once и checkpoints

Checkpoint фиксирует консистентную точку для state + input offsets.  
Для end-to-end exactly-once sink должен быть transactional или idempotent.  
Классический паттерн: `TwoPhaseCommitSinkFunction` коммитит внешнюю транзакцию только после успешного checkpoint completion.

## Типичные вопросы на интервью

**Q: Чем checkpoint отличается от savepoint?**  
A: Checkpoint — автоматический и частый snapshot для аварийного восстановления. Savepoint — ручной и осознанный snapshot для изменений (deploy, upgrade, rescale). Checkpoint оптимизируется под скорость recovery, savepoint — под переносимость и контроль.

**Q: Что произойдёт при падении TaskManager?**  
A: JobManager обнаружит failure, перезапустит затронутые таски (или весь job в зависимости от strategy), поднимет state из последнего успешного checkpoint и восстановит source offsets. Внешний sink должен быть согласован с checkpoint semantics, иначе возможны дубликаты.

**Q: Почему checkpoints могут “тормозить” job?**  
A: Большой state, медленный storage (S3/HDFS), частые checkpoints, backpressure и alignment overhead. Лечат: увеличить interval, включить incremental checkpoint (RocksDB), оптимизировать state volume/TTL, использовать unaligned checkpoints для сильного backpressure.

## Связи

- [Flink Exactly-Once Semantics](flink-exactly-once-semantics.md)
- [Flink State Management](flink-state-management.md)
- [Apache Flink Architecture](flink-architecture.md)
