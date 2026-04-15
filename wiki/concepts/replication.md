# Replication

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 5)

## Суть

**Репликация** — копии данных на нескольких узлах: для отказоустойчивости и масштабирования чтения. Модели обновления: **single-leader**, **multi-leader**, **leaderless**. Ключевые темы: синхронная vs асинхронная репликация, **replication lag**, чтение собственных записей, монотонное чтение, consistent prefix. Конфликты при multi-leader требуют явной стратегии.

## Модели репликации

| Модель | Как работает | Плюсы | Минусы |
|--------|-------------|-------|--------|
| **Single-leader** | Все writes на одного лидера; followers читают лог | Простота, нет write-конфликтов | Лидер — bottleneck и SPOF (до failover) |
| **Multi-leader** | Writes на любой из лидеров; асинхронная синхронизация | Resilience при partition, geo-distributed writes | Конфликты при concurrent writes (нужна стратегия: LWW, merge, manual) |
| **Leaderless** | Writes и reads на кворум узлов (R + W > N) | Нет единой точки отказа, продолжает при partition | Сложнее гарантии; read repair, anti-entropy; stale reads возможны |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **Synchronous replication** | Лидер ждёт подтверждения от фолловера перед ack клиенту. Durability, но latency. |
| **Asynchronous replication** | Лидер отвечает сразу; фолловеры догоняют. Быстрее, но риск потери при failover. |
| **Replication lag** | Задержка между записью на лидере и появлением на фолловере. Проблема для read-heavy систем. |
| **Read-after-write consistency** | Пользователь видит свои записи. Решение: читать со slave только если прошло достаточно времени, или читать своё с лидера. |
| **Monotonic reads** | Пользователь не «откатывается» назад при последовательных чтениях. Решение: sticky session к одной реплике. |
| **Failover** | Автоматическая смена лидера при его падении. Риски: split brain, потеря unreplicated writes. |
| **Split brain** | Два узла считают себя лидером → конфликтующие writes, потеря данных. Решение: fencing tokens, quorum. |

## Типичные вопросы на интервью

**Q: Чем отличается синхронная и асинхронная репликация?**
A: Синхронная: лидер ждёт ack от реплики → данные не потеряются при failover, но выше latency и один медленный follower тормозит всех. Асинхронная: лидер отвечает сразу → ниже latency, но при падении лидера unreplicated writes теряются. На практике часто **semi-synchronous**: один follower sync, остальные async.

**Q: Что такое replication lag и какие проблемы он вызывает?**
A: Задержка между записью и её появлением на реплике. Проблемы: пользователь не видит своё обновление (read-after-write), видит «откат» данных (non-monotonic reads), видит нарушенный порядок связанных записей (consistent prefix). Решения: sticky sessions, read-from-leader для своих writes, версионирование.

**Q: Как работает failover и что может пойти не так?**
A: При падении лидера followers выбирают нового (через consensus или timeout). Проблемы: **split brain** (оба считают себя лидером), потеря unreplicated writes, клиенты продолжают писать в старого лидера. Нужны: fencing tokens, epoch numbers, мониторинг.

**Q: Когда использовать leaderless (Dynamo-style)?**
A: Когда нужна высокая доступность при network partition, допустима eventual consistency, и нагрузка — write-heavy с простыми key-value операциями. Примеры: Cassandra, DynamoDB. Для data engineering: Kafka использует ISR-подход (in-sync replicas) — ближе к single-leader с кворумом.

**Q: Как Kafka реализует репликацию?**
A: Каждая partition имеет leader и ISR (in-sync replicas). Producer пишет в leader; реплики тянут. `acks=all` — ждать подтверждения от всех ISR. При падении лидера — новый из ISR. Не leaderless и не multi-leader, а single-leader per partition.

## Связи

- [Partitioning](partitioning.md) — репликация и партиции часто комбинируются.
- [Consistency and Consensus](consistency-and-consensus.md) — согласованность реплик.
- [Apache Kafka](../entities/apache-kafka.md) — репликация логов партиций.
