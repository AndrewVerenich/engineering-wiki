# Derived Data and Systems

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 12)

## Суть

**Производные данные** — индексы, кэши, представления, поисковые индексы, строящиеся из источника истины. Книга обсуждает **materialized views**, согласование batch и stream, идеи вроде **unbundling** (разные системы для разных задач с общим логом). Связь с архитектурой data-intensive приложений в целом.

## Связи

- [Batch Processing](batch-processing.md), [Stream Processing](stream-processing.md) — движки производных данных.
- [Reliability, Scalability, Maintainability](reliability-scalability-maintainability.md) — сопровождение цепочек ETL/ELT.
