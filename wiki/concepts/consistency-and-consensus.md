# Consistency and Consensus

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 9)

## Суть

**Линеаризуемость** — сильная модель «как один копия», дорогая в распределённой системе. **Причинность** и **total order** связывают события без полной глобальной синхронизации. **Консенсус** (выбор лидера, согласованные решения) реализуется через алгоритмы вроде Raft/Paxos (в книге — концептуально). Связь с практикой: ZooKeeper, etcd, лидер в кластерах БД.

## Модели согласованности

| Модель | Гарантия | Цена |
|--------|----------|------|
| **Linearizability** | «Как одна копия»: после успешной записи все чтения видят новое значение | Высокая latency (координация), невозможна при network partition (CAP) |
| **Causal consistency** | Причинно связанные события видны в правильном порядке; concurrent — в любом | Дешевле linearizability, достижима при partition |
| **Eventual consistency** | «Рано или поздно» все реплики сойдутся | Минимальная цена; stale reads возможны |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **CAP theorem** | При network partition нужно выбрать: **Consistency** (linearizable) или **Availability** (отвечаем даже без кворума). На практике partition случается → CP или AP. |
| **Total order broadcast** | Все узлы получают одни и те же сообщения в одном порядке. Эквивалентен consensus. Используется для replicated state machine. |
| **Lamport timestamp** | Logical clock: каждый узел инкрементирует счётчик; при общении — max(local, received) + 1. Даёт total order, но не causal order без доп. инфо. |
| **Raft / Paxos** | Алгоритмы consensus: выбор лидера и согласование значений среди большинства. Raft проще для понимания (leader-based, log replication). |
| **Epoch / term** | Номер «эпохи» лидера в Raft/Paxos. При выборе нового лидера epoch растёт; старый лидер с меньшим epoch отвергается. |
| **Quorum** | Большинство узлов (N/2 + 1); операция подтверждена, если quorum согласен. |

## Типичные вопросы на интервью

**Q: Объясни CAP теорему простыми словами.**
A: В распределённой системе при network partition (P — неизбежно) нужно выбрать: **C** (consistency — все видят одно и то же, но часть запросов отклоняется) или **A** (availability — все запросы обслуживаются, но могут быть stale). В реальности это спектр, а не бинарный выбор. Большинство систем — AP с tunable consistency (Cassandra) или CP (ZooKeeper, etcd).

**Q: В чём разница между linearizability и serializability?**
A: **Linearizability** — свойство одного объекта: операции выглядят как мгновенные в реальном времени. **Serializability** — свойство транзакций: набор транзакций выглядит как выполненный последовательно. Можно иметь serializable без linearizable (snapshot isolation) и linearizable без serializable (одиночные записи).

**Q: Как работает Raft?**
A: Узлы выбирают **лидера** через election (рандомный таймаут → кандидат → голосование кворумом). Лидер принимает все writes, реплицирует через **log entries**. Если кворум followers подтвердил — entry committed. При падении лидера — новые выборы с большим **term**. Committed entries никогда не теряются (safety guarantee).

**Q: Когда нужна linearizability?**
A: Выбор лидера (один лидер), уникальные constraints (одно имя пользователя), CAS-операции. Если можно допустить stale reads (аналитика, кэш) — eventual consistency дешевле.

**Q: Зачем нужен ZooKeeper/etcd?**
A: Consensus-сервис: хранение конфигурации, leader election, distributed locks, service discovery. Kafka использует ZooKeeper (или KRaft в новых версиях) для координации брокеров и partition leadership. ClickHouse Keeper — аналог для ClickHouse кластера.

## Связи

- [Distributed Systems Pitfalls](distributed-systems-pitfalls.md) — почему консенсус нужен.
- [Replication](replication.md) — лидер и согласованность чтений.
