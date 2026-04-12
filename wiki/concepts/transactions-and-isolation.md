# Transactions and Isolation

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 7)

## Суть

**Транзакции** группируют операции с гарантиями **ACID** (в разных СУБД интерпретации различаются). **Уровни изоляции** (read committed, snapshot isolation, serializable) защищают от гонок, lost update, phantom reads по разной цене. Распределённые транзакции (2PC и альтернативы) усложняют отказоустойчивость и latency.

## Связи

- [Partitioning](partitioning.md) — координация между партициями.
- [Consistency and Consensus](consistency-and-consensus.md) — строгая сериализуемость и консенсус.
- [PostgreSQL](../entities/postgresql.md) — MVCC и изоляция на практике.
