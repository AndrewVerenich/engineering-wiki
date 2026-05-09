# System Design

Зона **System Design** — отдельная от Foundations / Data Engineering / Java Backend. Здесь живёт **методология** проектирования систем под собеседование senior-уровня и **разборы кейсов**.

## Зачем отдельная зона

- На senior-собеседовании System Design — **отдельный round**, отличный от доменных вопросов.
- Кейс не является «концептом» — это **применение многих концептов сразу** (replication + partitioning + caching + queueing + transactions + observability в одной задаче).
- Кейсы пересекают и Java Backend, и Data Engineering, и Foundations — нет смысла прибивать их к одной зоне.

## Структура

Зона содержит два типа страниц:

### 1. Framework & methodology (`overviews/system-design-fundamentals.md`)

Алгоритм собеседования: functional → non-functional → API → data model → high-level → deep dive → bottlenecks → trade-offs. Capacity estimation, чеклисты non-functional requirements, типичные subsystems (load balancer, cache, queue, DB, search).

### 2. Cases (страницы в этой папке)

Конкретные системы, которые часто спрашивают на собеседовании. Каждый кейс — самостоятельная страница по шаблону ниже.

## Текущие страницы

(пусто — наполняется)

Планируемые кейсы (см. `wiki/backlog.md`):

- URL Shortener
- Rate Limiter (token bucket / leaky bucket / sliding window)
- Distributed Cache
- Notification System
- News Feed / Timeline
- Payment System (idempotency, double-spend, eventual consistency)
- Real-time Analytics Dashboard (Kafka → Flink → ClickHouse)
- CDC Replication Pipeline (Postgres → Kafka → DWH)
- Chat / Messaging
- Search Autocomplete (typeahead)
- Distributed Job Scheduler

## Шаблон страницы

Каждый кейс — отдельный `.md` файл в этой папке. Filename: `kebab-case-english.md` (например, `url-shortener.md`).

```markdown
# System Design: <Case Name>

**Контекст:** одно предложение — какой бизнес-кейс / задача.

## Functional requirements

- Что система должна делать (use cases).
- Что **не** делает (out of scope).

## Non-functional requirements

| Параметр | Значение | Обоснование |
|----------|----------|-------------|
| Scale (RPS) | ... | ... |
| Data volume | ... | ... |
| Latency (p99) | ... | ... |
| Availability | ... | ... |
| Consistency | strong / eventual / causal | ... |
| Cost / budget | ... | ... |

## Capacity estimation

Грубые числа на салфетке: storage, bandwidth, QPS read/write, ratio. Указываем единицы и формулы.

## API / Contract

Endpoints / events / queries. REST/gRPC/async. Идемпотентность.

## Data model

- Таблицы / topics / streams / keys.
- Indexes / partitioning keys.
- Хранилище: какое и почему (PG / ClickHouse / Cassandra / Redis / S3).

## High-level design

Mermaid-диаграмма + список компонентов: load balancer, app, cache, DB, queue, worker, CDN, ...

## Deep dives

Точки, где интервьюер копает глубже:

- Hot keys / hot partitions.
- Cache strategy (cache-aside / write-through / write-behind).
- Replication & failover.
- Consistency model и trade-offs.
- Backpressure и rate limiting.
- Schema evolution.

## Failure modes

- Что ломается, что чинит, что мониторим.
- Fallback / degradation strategy.

## Trade-offs

| Решение | Альтернатива | Почему выбрано |
|---------|--------------|-----------------|
| ... | ... | ... |

## Типичные follow-up вопросы

- «Как масштабируется до 10×?»
- «Что если сеть partition'ится между регионами?»
- «Как мигрируем схему без downtime?»

## Связи

- Использованные patterns: [...](../patterns/...)
- Использованные concepts: [...](../concepts/...)
- Cross-zone: ...
```

## Связи

- [System Design Fundamentals](../overviews/system-design-fundamentals.md) — методология и алгоритм собеседования.
- [Patterns](../patterns/README.md) — каталог паттернов, которые используются в кейсах.
- [Backlog](../backlog.md) — что планируется добавить.
