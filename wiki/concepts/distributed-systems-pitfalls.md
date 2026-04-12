# Distributed Systems Pitfalls

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 8)

## Суть

В распределённой среде нельзя полагаться на «идеальную» сеть или часы: **задержки непредсказуемы**, **часы неточны**, процессы **паузятся** (GC, virtualization). **Частичные сбои** — норма. Отсюда таймауты, fencing, версионирование, и осторожность с «разумными» предположениями о времени и порядке событий.

## Связи

- [Consistency and Consensus](consistency-and-consensus.md) — как строить порядок и согласие при этих ограничениях.
- [Replication](replication.md) — задержки репликации и разрыв с реальностью.
