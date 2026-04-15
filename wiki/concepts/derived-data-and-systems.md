# Derived Data and Systems

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 12)

## Суть

**Производные данные** — индексы, кэши, представления, поисковые индексы, строящиеся из источника истины. Книга обсуждает **materialized views**, согласование batch и stream, идеи вроде **unbundling** (разные системы для разных задач с общим логом). Связь с архитектурой data-intensive приложений в целом.

## Ключевые концепции

| Концепция | Суть |
|-----------|------|
| **System of record (source of truth)** | Авторитетная версия данных; при конфликте — выигрывает. Обычно OLTP-БД или immutable event log. |
| **Derived data** | Индексы, кэши, materialized views, search индексы, DWH витрины — всё, что можно **пересоздать** из source of truth. |
| **Unbundling** | Вместо одной монолитной БД — специализированные системы для разных задач (OLTP, search, analytics, cache), связанные через **log** (Kafka). Каждая система — derived view. |
| **Lambda architecture** | Batch layer (полные пересчёты) + speed layer (stream для свежих данных) + serving layer (merge). Проблема: две кодовые базы для одной логики. |
| **Kappa architecture** | Только stream layer; replay с начала лога для recomputation. Проще, но требует retentive log (Kafka с длинным retention или compaction). |

## Типичные вопросы на интервью

**Q: Что такое derived data и приведи примеры?**
A: Данные, которые можно полностью пересоздать из source of truth. Примеры: поисковый индекс (Elasticsearch) из OLTP; materialized view в ClickHouse из raw facts; кэш (Redis) из БД; DWH витрина из staging. Если derived данные потеряны — их можно восстановить; source of truth — нельзя.

**Q: Объясни разницу Lambda vs Kappa architecture.**
A: **Lambda**: batch layer для точных результатов (часы latency) + speed layer для приблизительных real-time. Serving layer merge'ит. Минус: дублирование логики. **Kappa**: только stream; для full recompute — replay log с начала. Проще, но нужен long-retention log. На практике многие системы — гибрид: stream для fast path + периодический batch для коррекции.

**Q: Что значит «unbundling the database»?**
A: Традиционная БД объединяет storage, indexing, query, transactions. Unbundling — разделение: Kafka (log), ClickHouse (analytics), Elasticsearch (search), Redis (cache). Каждый оптимален для своей задачи. Kafka-лог — «позвоночник», связывающий системы. Сложность: eventual consistency, operational overhead.

**Q: Как обеспечить consistency между derived systems?**
A: Через **single source of truth** (event log) + **deterministic consumers**. Если каждая система потребляет одинаковый лог в одинаковом порядке — в итоге converge к одному состоянию (eventual consistency). Для strict consistency — consensus, distributed transactions (дорого). На практике: idempotent consumers + monitoring lag.

## Связи

- [Batch Processing](batch-processing.md), [Stream Processing](stream-processing.md) — движки производных данных.
- [Reliability, Scalability, Maintainability](reliability-scalability-maintainability.md) — сопровождение цепочек ETL/ELT.
