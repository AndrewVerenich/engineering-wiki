# Batch Processing

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 10)

## Суть

**Пакетная обработка** — обход больших наборов данных офлайн (MapReduce, распределённые join’ы, сортировки). Отличие от OLTP: высокая пропускная способность, высокая задержка, **идемпотентность** и **recomputation** как паттерны. Связь с **dataflow** и материализацией производных наборов.

## Связи

- [Derived Data and Systems](derived-data-and-systems.md) — batch как источник производных представлений.
- [Stream Processing](stream-processing.md) — контраст и объединение (lambda/kappa — в derived-data).
- [OLTP vs OLAP](../comparisons/oltp-vs-olap.md) — OLAP и аналитические batch-нагрузки.
