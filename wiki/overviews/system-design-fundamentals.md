# System Design Fundamentals

**Каркас зоны** [System Design](../system-design/README.md). Раздел в стадии наполнения — методология и framework собеседования будут добавлены по мере чтения источников (см. [backlog](../backlog.md) → System Design).

## Зачем эта страница существует

Аналог `java-backend-fundamentals.md` и `data-engineering-fundamentals.md`, но для зоны System Design. Здесь живут:

- **Framework собеседования** — пошаговый алгоритм решения SD-задачи в условиях интервью (45–60 минут).
- **Capacity estimation** — приёмы оценки на салфетке.
- **Чеклисты non-functional requirements**.
- **Карта типичных subsystems** (load balancer, cache, queue, DB, search, CDN) — что это, когда нужно, какие альтернативы.
- **Список case-страниц** (`wiki/system-design/`) с группировкой и порядком изучения.

## Опорные книги

См. [backlog](../backlog.md) → раздел System Design. Главные кандидаты: *System Design Interview* (Alex Xu, vol. 1+2), *Building Microservices* (Newman), *Designing Data-Intensive Applications* (уже есть в Foundations).

## Темы (TBD)

### Framework: 7 шагов system design интервью

(планируется)

1. Прояснить требования (functional / non-functional).
2. Capacity estimation.
3. API.
4. Data model.
5. High-level design.
6. Deep dives.
7. Trade-offs / wrap-up.

### Capacity estimation cookbook

(планируется)

- Кеши округлений (1 GB ≈ 10⁹ B, 1 day ≈ 10⁵ s).
- Бюджет latency по компонентам.
- Хранилище: записи × размер × срок хранения.

### Карта subsystems

(планируется)

- Load balancing (L4 vs L7, sticky sessions).
- Caching (cache-aside / write-through / write-behind).
- Message queues (Kafka / RabbitMQ / SQS — когда что).
- Databases (RDB / KV / wide-column / document / graph / search / time-series / OLAP).
- CDN.

### Case-страницы

См. список планируемых кейсов в [system-design/README.md](../system-design/README.md) и [backlog.md](../backlog.md).

## Связанные foundations

System Design опирается на Foundations:

- [Reliability, Scalability, Maintainability](../concepts/reliability-scalability-maintainability.md)
- [Replication](../concepts/replication.md)
- [Partitioning](../concepts/partitioning.md)
- [Distributed Systems Pitfalls](../concepts/distributed-systems-pitfalls.md)
- [Consistency and Consensus](../concepts/consistency-and-consensus.md)
- [Encoding and Schema Evolution](../concepts/encoding-and-schema-evolution.md)

## Связи

- [System Design (зона)](../system-design/README.md)
- [Patterns Catalog](../patterns/README.md) — паттерны, на которые опираются кейсы.
- [Java Backend Fundamentals](java-backend-fundamentals.md), [Data Engineering Fundamentals](data-engineering-fundamentals.md) — параллельные домены.
- [SCHEMA.md](../../schema/SCHEMA.md) — правила вики.
