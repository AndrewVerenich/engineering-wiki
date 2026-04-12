# Reliability, Scalability, Maintainability

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 1)

## Суть

Три свойства, которыми описывают «хорошие» data-intensive приложения:

- **Reliability** — система продолжает корректно работать при сбоях (железо, ПО, человек).
- **Scalability** — справляться с ростом нагрузки разумными средствами (масштабирование **up** или **out**, узкие места).
- **Maintainability** — операции, простота эволюции: наблюдаемость, управляемость, простота для новых людей.

## Связи

- [Storage and Retrieval](storage-and-retrieval.md) — узкие места часто в хранении и чтении.
- [Replication](replication.md), [Partitioning](partitioning.md) — типичные ответы на масштабирование.
- [Distributed Systems Pitfalls](distributed-systems-pitfalls.md) — почему «надёжность» в распределёнке сложнее.
