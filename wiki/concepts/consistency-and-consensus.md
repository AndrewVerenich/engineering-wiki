# Consistency and Consensus

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 9)

## Суть

**Линеаризуемость** — сильная модель «как один копия», дорогая в распределённой системе. **Причинность** и **total order** связывают события без полной глобальной синхронизации. **Консенсус** (выбор лидера, согласованные решения) реализуется через алгоритмы вроде Raft/Paxos (в книге — концептуально). Связь с практикой: ZooKeeper, etcd, лидер в кластерах БД.

## Связи

- [Distributed Systems Pitfalls](distributed-systems-pitfalls.md) — почему консенсус нужен.
- [Replication](replication.md) — лидер и согласованность чтений.
