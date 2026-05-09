# Patterns Catalog

Каталог **архитектурных и интеграционных паттернов** — отдельный тип страниц рядом с `concepts/`, `entities/`, `comparisons/`.

## Чем pattern отличается от concept

| | Concept | Pattern |
|---|---------|---------|
| Что описывает | **Как работает** механизм / технология (MVCC, watermarks, ISR) | **Решение типовой проблемы** (outbox, saga, idempotency) |
| Источник | Книга / документация / спецификация | Опыт + книги + community |
| Структура | Суть → внутренности → trade-offs → Q&A | Проблема → решение → реализация → когда не применять |
| Применение | На собеседовании отвечаешь «как X устроен» | На собеседовании отвечаешь «как ты бы решил Y» |

Паттерн — **cross-zone**: один и тот же outbox используется и в Java Backend (Spring + JPA + Kafka), и в Data Engineering (CDC pipelines). Поэтому каталог паттернов — горизонтальный, не привязан к одной зоне. Зона указывается на самой странице через поле `**Зона:**`.

## Категории

| Категория | Примеры |
|-----------|---------|
| **Reliability & messaging** | Outbox, Inbox, Dead-letter queue, Retry with backoff/jitter, Idempotency keys |
| **Distributed transactions** | Saga (orchestration / choreography), 2PC, TCC, Compensating actions |
| **Consistency & state** | CQRS, Event sourcing, Read replica routing, Optimistic locking, Pessimistic locking |
| **Resilience** | Circuit breaker, Bulkhead, Timeout & deadline propagation, Rate limiting algorithms |
| **Scalability & deployment** | Sharding strategies, Leader election, Sidecar, Service mesh basics |
| **Architecture / migration** | Strangler fig, Anti-corruption layer, BFF (backend-for-frontend), API gateway |
| **Data engineering** | Medallion architecture, SCD strategies, Late-arriving facts, CDC patterns |
| **ID & uniqueness** | Snowflake IDs, UUIDv7, KSUID, ULID, Distributed counter |

## Текущие страницы

(пусто — наполняется)

Планируемые паттерны (см. `wiki/backlog.md`):

- Outbox / Transactional messaging
- Saga (orchestration vs choreography)
- Idempotency keys
- Circuit breaker & bulkhead
- Retry with exponential backoff + jitter
- Optimistic vs pessimistic locking
- CQRS / Event sourcing
- Distributed ID generation (Snowflake, UUIDv7)
- Strangler fig
- Rate limiting algorithms
- Leader election

## Шаблон страницы

Filename: `kebab-case-english.md` без суффикса `-pattern` (контекст и так задан папкой).

```markdown
# Pattern: <Name>

**Зона:** Foundations / Data Engineering / Java Backend / System Design  
**Категория:** reliability / consistency / resilience / messaging / ...

## Проблема

Что и почему болит. Конкретный сценарий — не абстракция.
Например: «При публикации события в Kafka после COMMIT транзакции PG возможна ситуация, когда транзакция закоммичена, а событие потеряно».

## Решение

Как паттерн её решает. Один абзац — суть.

## Структура

- Компоненты.
- Поток данных (mermaid-диаграмма или ASCII).
- Инварианты, которые поддерживает паттерн.

## Реализация

Кодовый скелет / config / SQL — в стеке, релевантном зоне (Java + Spring, ClickHouse, Kafka и т. д.). Минимальный, читаемый, без копипасты доков.

## Trade-offs

| Плюсы | Минусы |
|-------|--------|
| ... | ... |

## Когда применять / не применять

- **Применять:** ...
- **Не применять:** ... (антипаттерны).

## Gotchas / production hazards

Что обычно ломается, на чём горят команды.

## Типичные вопросы на интервью

**Q: ...**
A: ...

(минимум 3-5 Q&A — как в concept-страницах)

## Связи

- Concepts: [...](../concepts/...)
- Другие patterns: [...](other.md)
- Использовано в кейсах: [...](../system-design/...)
```

## Связи

- [Patterns Catalog Overview](../overviews/patterns-catalog.md) — обзор и порядок изучения.
- [System Design](../system-design/README.md) — кейсы используют паттерны из этого каталога.
- [Backlog](../backlog.md) — планируемые паттерны.
