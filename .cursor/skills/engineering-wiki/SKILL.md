---
name: engineering-wiki
description: Manage a personal engineering knowledge base for interview prep and learning. Use when the user wants to add sources (books, articles, courses), create or update concept/entity/comparison pages, review for interviews, check wiki integrity, or asks about wiki structure. Activates on any mention of wiki, knowledge base, interview prep, or adding engineering knowledge.
---

# Engineering Wiki — Personal Knowledge Base

Персональная вики для подготовки к собеседованиям и систематизации инженерных знаний.
Язык вики — **русский** (термины на английском, объяснения на русском).

Wiki ориентирована на **два рабочих домена**: Java Backend и Data Engineering. Общий материал (распределённые системы, транзакции, SQL, моделирование) живёт в зоне **Foundations**. Дополнительно есть **System Design** — отдельная зона для архитектурных кейсов и методологии собеседований. Поверх зон есть два **горизонтальных каталога**: `Patterns` (паттерны, cross-zone) и `SQL Practice` (упражнения). См. [SCHEMA.md](../../../schema/SCHEMA.md) — там зафиксированы зоны, подсекции и зональный маршрутизатор для классификации новых страниц.

## Core Principles

1. **Interview-first**: каждая concept-страница содержит секцию "Типичные вопросы на интервью" с Q&A.
2. **Deep understanding**: не пересказ документации, а объяснение **как работает под капотом** и **почему** выбран такой дизайн.
3. **Cross-linked**: страницы связаны. Нет изолированных страниц.
4. **Structured consistently**: все страницы одного типа следуют одному шаблону.
5. **Zone-routed**: каждая concept/comparison-страница принадлежит ровно одной зоне (Foundations / Data Engineering / Java Backend / System Design). Дублирование ссылок в `index.md` запрещено; пересечения — через `## Связи`.
6. **Тип страницы определяется ролью контента**: концепт (как устроено) ≠ паттерн (как решается проблема) ≠ system design кейс (как проектируется система) ≠ SQL practice задача.

## Wiki Zones (4 зоны + 2 горизонтальных каталога)

Полное описание — в `schema/SCHEMA.md` (раздел 1). Краткая сводка для оперативного решения:

| Зона | Подсекции | Что туда идёт |
|------|-----------|----------------|
| **Foundations** | Distributed Systems & System Design / Transactions, Isolation & Storage Internals / SQL & Relational Modeling | Replication, partitioning, consensus, encoding/schema evolution, transactions, MVCC, locking, PG/MySQL internals, SQL execution, indexes, normalization, data models |
| **Data Engineering** | DDIA-data chapters / Kimball / ClickHouse / dbt / Spark / Stream Processing (Flink, Kafka Streams) | DWH, dimensional modeling, columnar engines, batch/stream pipelines, lakehouse, CDC |
| **Java Backend** | JVM internals / Concurrency & Memory Model / Spring / Kotlin / Web & APIs / Observability / Caching & Redis / Messaging & Kafka | GC, JIT, JMM, virtual threads, Spring (любой модуль), Kotlin coroutines, REST/gRPC/OpenAPI, Micrometer/OpenTelemetry |
| **System Design** | Framework & methodology / Cases | Алгоритм собеседования, capacity estimation, разборы кейсов (URL shortener, rate limiter, payment system, real-time analytics, CDC pipeline и т. д.) |

**Горизонтальные каталоги (не зоны, а cross-zone разделы):**

| Каталог | Что туда идёт | Где живут |
|---------|---------------|-----------|
| **Patterns** | Архитектурные / интеграционные паттерны (outbox, saga, idempotency, circuit breaker, CQRS, distributed ID generation) | `wiki/patterns/`, в индексе — секция `## Patterns` |
| **SQL Practice** | Прикладные SQL-задачи (top-N per group, sessionization, recursive CTE и т. д.) | `wiki/sql-practice/`, в индексе — секция `## SQL Practice` |

**Принцип определения типа страницы для нового материала:**

1. **Источник** (книга/статья/курс) → `wiki/sources/`.
2. **Описывает механизм/устройство** (как работает MVCC, watermarks, ISR) → concept в `wiki/concepts/` + зона по маршрутизатору.
3. **Описывает решение типовой проблемы** (outbox, saga, circuit breaker) → pattern в `wiki/patterns/` + cross-zone каталог.
4. **Проектирует end-to-end систему** (URL shortener, payment system) → case в `wiki/system-design/`.
5. **Прикладная SQL-задача** (top-N, sessionization) → exercise в `wiki/sql-practice/`.
6. **Сравнение X vs Y** → `wiki/comparisons/`.
7. **Конкретный инструмент/СУБД** (PostgreSQL, Kafka) → entity в `wiki/entities/`.
8. **Обзор темы / learning track / interview prep** → `wiki/overviews/`.

**Принцип определения зоны для concept-страницы:**

1. Общее свойство данных или распределённых систем (replication, transactions, MVCC, normalization, data models) → Foundations.
2. DE-инструмент или DE-паттерн (ClickHouse engines, dbt layers, Flink windows, dimensional modeling) → Data Engineering.
3. JVM / Spring / Kotlin / Java-concurrency / web API → Java Backend.
4. Если страница попадает в две зоны — выбирается та, где она наиболее «корневая», во второй зоне ставится cross-link через `## Связи`.

**Multi-zone ingest:** одна книга может породить страницы в разных зонах И в разных каталогах (например, *Release It!* → Patterns (catalog: circuit breaker, bulkhead, timeout) + cross-links в Java Backend). При ingest материал разносится по нужным контейнерам.

## File Structure

```
wiki/
├── index.md                    # Master index — 4 зоны + Patterns + SQL Practice + Sources + Entities + Overviews + Backlog
├── log.md                      # Changelog — фактическая история ingest'ов
├── backlog.md                  # Reading & ingest backlog — план будущих источников/тем
├── sources/                    # Книги, статьи, курсы, документация
├── concepts/                   # Концепты механизмов (плоская папка; зона определяется в index.md)
├── patterns/                   # Каталог паттернов (cross-zone; зона указывается в шапке страницы)
├── system-design/              # Кейсы system design (зона System Design)
├── sql-practice/               # Прикладные SQL-задачи
├── entities/                   # Конкретные инструменты (PostgreSQL, Kafka...)
├── comparisons/                # Сравнения (X vs Y)
└── overviews/                  # Обзоры тем, learning tracks, interview prep
schema/
└── SCHEMA.md                   # Правила структуры: зоны, types of pages, rules
```

Папки `concepts/`, `comparisons/`, `entities/`, `patterns/`, `system-design/`, `sql-practice/` — плоские. Зона concept-страницы определяется не папкой, а её положением в `wiki/index.md`.

## Page Templates

### Source Page (`wiki/sources/`)

Внешний источник знаний: книга, статья, курс, документация.

```markdown
# Title

| Поле | Значение |
|------|----------|
| **Автор** | ... |
| **Издательство** | ... |
| **Год/Издание** | ... |
| **Тип** | книга / статья / курс / документация |

## О книге
Одни абзац: о чём, какая ценность для вики.

## Структура
Оглавление с ссылками на concept-страницы вики.

## Ключевые темы вики
Таблица: тема → ссылка на concept.

## Связи
Ссылки на related overviews, entities.
```

**Правила для sources:**
- Только **внешние материалы** (не личные планы, не roadmaps).
- Каждый source порождает concept-страницы с `**Источник:** [ссылка](...)`.
- Filename: `kebab-case-english.md`.

### Concept Page (`wiki/concepts/`)

Концепт, технология, паттерн — ядро вики.

```markdown
# Title

**Источник:** [Source Name](../sources/source.md) (глава/раздел)

## Суть
1-2 абзаца: что это, зачем, ключевой trade-off. **Bold** на главных терминах.

## [Тематические секции]
Таблицы, схемы, примеры. Формат зависит от темы:
- Таблицы сравнения (| Термин | Определение |)
- Пошаговые объяснения механизмов
- Примеры кода/SQL где уместно

## Типичные вопросы на интервью
**Q: Вопрос, как его задают на собеседовании.**
A: Ответ — конкретный, структурированный, с примерами. Не абстрактный.

## Связи
- [Related Concept](related.md) — одна строка: почему связано.
```

**Правила для concepts:**
- Секция "Суть" — обязательна. Это summary для быстрого повторения.
- Секция "Типичные вопросы на интервью" — **обязательна**. Минимум 3-5 Q&A.
- Q&A должны быть **реалистичными**: вопрос как на собеседовании, ответ как от сильного кандидата.
- Ответы содержат конкретику: числа, примеры, trade-offs, "когда X, когда Y".
- Таблицы предпочтительнее абзацев для сравнений и терминов.
- Секция "Связи" — обязательна. Минимум 2 ссылки с кратким пояснением.

### Comparison Page (`wiki/comparisons/`)

```markdown
# X vs Y

**Контекст:** одно предложение — почему сравниваем.

## Главная разница
Таблица: | | X | Y | — 4-6 ключевых критериев.

## [Детальные секции]

## Типичные вопросы на интервью

## Связи
```

### Entity Page (`wiki/entities/`)

```markdown
# Entity Name

**Тип:** категория (реляционная СУБД, message broker, etc.)

## В контексте вики
Какие источники покрывают, ссылки.

## Ключевые характеристики
Таблица: | Характеристика | Суть |

## Типичные вопросы на интервью

## Связи
```

### Overview Page (`wiki/overviews/`)

Обзор темы, learning track, interview prep guide. Собирает ссылки на concepts, comparisons, sources.

### Pattern Page (`wiki/patterns/`)

Архитектурный/интеграционный паттерн.

```markdown
# Pattern: Name

**Зона:** Foundations / Data Engineering / Java Backend / System Design  
**Категория:** reliability / consistency / resilience / messaging / scalability / ...

## Проблема
Что и почему болит. Конкретный сценарий.

## Решение
Как паттерн её решает. Один абзац — суть.

## Структура
Компоненты, поток данных, инварианты. Mermaid/ASCII-диаграмма.

## Реализация
Кодовый скелет / config / SQL — в стеке релевантной зоны.

## Trade-offs
| Плюсы | Минусы |

## Когда применять / не применять
- **Применять:** ...
- **Не применять:** (антипаттерны)

## Gotchas / production hazards

## Типичные вопросы на интервью
(минимум 3-5 Q&A)

## Связи
- Concepts, другие patterns, кейсы system design.
```

**Правила для patterns:**
- Поле `**Зона:**` обязательно — pattern горизонтальный, зона указывается в шапке.
- Секции `Проблема` и `Решение` — обязательны (это main differentiator от concept).
- Q&A — минимум 3-5, как в concept.
- Filename: `kebab-case-english.md` без суффикса `-pattern` (контекст задан папкой).

Полный шаблон и каталог — см. [wiki/patterns/README.md](../../../wiki/patterns/README.md).

### System Design Case Page (`wiki/system-design/`)

Кейс проектирования системы.

```markdown
# System Design: Case Name

**Контекст:** одно предложение — какой бизнес-кейс / задача.

## Functional requirements
- Что система делает (use cases).
- Что НЕ делает (out of scope).

## Non-functional requirements
| Параметр | Значение | Обоснование |
| Scale (RPS) | ... | ... |
| Latency (p99) | ... | ... |
| Availability | ... | ... |
| Consistency | strong / eventual / causal | ... |
| Cost / budget | ... | ... |

## Capacity estimation
Storage, bandwidth, QPS read/write.

## API / Contract
Endpoints / events / queries. Идемпотентность.

## Data model
Таблицы / topics / streams / keys / partitioning.

## High-level design
Mermaid-диаграмма + список компонентов.

## Deep dives
Точки, где интервьюер копает: hot keys, cache strategy, replication, consistency, backpressure, schema evolution.

## Failure modes
Что ломается, что мониторим, fallback strategy.

## Trade-offs
| Решение | Альтернатива | Почему выбрано |

## Типичные follow-up вопросы

## Связи
- Использованные patterns, concepts, cross-zone.
```

**Правила для system design pages:**
- Все секции до `Trade-offs` — обязательны.
- Capacity estimation **с числами**, не «много/мало».
- Trade-offs **с альтернативами**: почему не выбрали другое.
- Кейс **использует** концепты и паттерны через `## Связи`, не пересказывает их. Если возникает новый общий концепт — выделяется в `wiki/concepts/` или `wiki/patterns/`.
- Filename: `kebab-case-english.md` (например, `url-shortener.md`, `payment-system.md`).

Полный шаблон и список планируемых кейсов — см. [wiki/system-design/README.md](../../../wiki/system-design/README.md).

### SQL Practice Page (`wiki/sql-practice/`)

Прикладная SQL-задача.

```markdown
# SQL Practice: Task Name

**Категория:** window functions / recursive CTE / gaps & islands / ...
**Сложность:** easy / medium / hard
**Диалект:** PostgreSQL / ANSI

## Задача
Бизнес-описание.

## Входные данные
DDL + INSERT'ы тестовых данных.

## Ожидаемый результат
Таблица с примером.

## Решение
SQL.

## Объяснение
Пошагово, что делает каждая часть.

## План выполнения (EXPLAIN)
Что важного: joins, scans, sorts. Где сломается при росте.

## Альтернативные решения
Сравнение по читаемости / производительности / переносимости.

## Типичные ошибки
NULL-семантика, дубли, ties.

## Когда спрашивают на интервью

## Связи
- SQL концепты, индексы.
```

**Правила для SQL practice:**
- DDL входных данных и ожидаемый результат — обязательны (задача воспроизводима).
- Минимум 1 решение + пояснение.
- Альтернативные решения — желательны, особенно для medium/hard задач.
- Filename: описывает паттерн задачи, не входные данные (`top-n-per-group.md`, не `users-and-orders.md`).

Полный шаблон и план задач — см. [wiki/sql-practice/README.md](../../../wiki/sql-practice/README.md).

## Ingest Workflow (adding a new source)

When the user says they read a book/article/course:

1. **Create source page** in `wiki/sources/` with metadata and structure. Указать в `wiki/index.md` рядом со ссылкой зону (Foundations / DE / Backend / System Design / mixed).
2. **Classify material by type AND zone.** Перед созданием страниц определить:
   - **Тип** каждого выделенного элемента: concept (как устроено) / pattern (как решается проблема) / system design case (как проектируется система) / SQL practice (упражнение) / comparison / entity.
   - **Зону** для concept (Foundations / Data Engineering / Java Backend / System Design) — см. таблицу выше и SCHEMA.md раздел 1.
   - Pattern — горизонтальный каталог (зона указывается в шапке страницы).
   - SQL Practice — горизонтальный каталог (без зоны).
   - System Design case — всегда зона System Design.
   Multi-zone и multi-каталог ingest — норма (одна книга → концепты в Foundations + паттерны в каталоге + кейсы в SD).
3. **Create concept pages** for key topics. Each concept:
   - Has "Суть" section (concise summary).
   - Has "Типичные вопросы на интервью" (3-5 realistic Q&A).
   - Has "Связи" linking to existing wiki pages — особенно cross-zone links и links на использованные patterns.
   - References the source via `**Источник:**`.
4. **Create pattern pages** if the source describes problem→solution patterns (outbox, saga, circuit breaker). Pattern всегда содержит секции `Проблема` / `Решение` / `Реализация` / `Trade-offs` / `Когда не применять`.
5. **Create system design pages** if the source includes case studies (URL shortener, payment system). Case всегда содержит functional/non-functional/capacity/API/data-model/high-level/trade-offs.
6. **Create SQL practice pages** if the source provides SQL exercises. Always include DDL + expected result + solution + EXPLAIN.
7. **Create comparison pages** if the source reveals X vs Y contrasts.
8. **Update existing pages** — add cross-links from related concepts/patterns/cases.
9. **Update `wiki/index.md`** — add new pages to **correct section**:
   - Concept/Comparison → зональная секция (Foundations / DE / Java Backend).
   - Pattern → секция `## Patterns`.
   - System Design case → секция `## System Design`.
   - SQL Practice → секция `## SQL Practice`.
   - Source → таблица `## Sources` с пометкой зон/каталогов.
   Не дублировать ссылку в нескольких секциях.
10. **Update relevant overview**:
    - DE-материал → `wiki/overviews/data-engineering-fundamentals.md`.
    - Backend-материал → `wiki/overviews/java-backend-fundamentals.md`.
    - System design → `wiki/overviews/system-design-fundamentals.md`.
    - Patterns → `wiki/overviews/patterns-catalog.md`.
    - SQL practice → `wiki/overviews/sql-practice-roadmap.md`.
    - Foundations-материал → блок «Связанные foundations» в обоих доменных overview.
11. **Update `wiki/log.md`** — add `## [INGEST] Source Name` entry with list of created/updated files **и затронутых зон/каталогов**.
12. **Update `wiki/backlog.md`** — если источник был в плане, переместить пункт в статус `[x]` со ссылкой на запись в `log.md`. Если по ходу ingest появились идеи новых источников/тем — добавить в backlog.

**Important:** if the user mentions specific topics they liked (e.g. "мне понравились транзакции и индексы"), focus pages on those topics with extra depth.

## Index Structure (`wiki/index.md`)

Index организован по **зонам и горизонтальным каталогам**:

```markdown
## Domains (overviews)
- Java Backend Fundamentals
- Data Engineering Fundamentals
- System Design Fundamentals
- Patterns Catalog
- SQL Practice Roadmap

## Sources
| Источник | Зона / каталог |
- table: source name + zone (Foundations / DE / Backend / SD / Patterns / SQL Practice / mixed)

## Foundations (shared)
### Distributed Systems & System Design
### Transactions, Isolation & Storage Internals
### SQL & Relational Modeling

## Data Engineering
### DDIA — главы по обработке данных
### Dimensional modeling (Kimball)
### ClickHouse
### dbt
### Spark
### Stream Processing (Flink)
### Stream Processing (Kafka Streams)
### Apache Kafka

## Java Backend
### Caching & Redis
### Messaging & Kafka
### JVM internals (planned)
### Concurrency & memory model (planned)
### Spring (planned)
### Kotlin (planned)
### Web & APIs (planned)
### Observability (planned)

## System Design
### Framework & methodology
### Cases

## Patterns
- список pattern-страниц с указанием зоны

## SQL Practice
### Joins & filters / Aggregations / Window functions / ...
- список задач с указанием сложности

## Entities
- entity name — пометка зоны

## Overviews
- domain overviews + interview-prep + learning tracks

## Backlog
- ссылка на wiki/backlog.md
```

**Правила index'а:**
- Каждая concept/comparison-страница появляется в `index.md` **ровно один раз** — в своей зоне.
- Pattern-страница появляется **только** в секции `## Patterns`, не в зональной секции (даже если в шапке pattern'а указана зона Java Backend).
- System design case — только в `## System Design`.
- SQL practice — только в `## SQL Practice`.
- Comparison живёт внутри подсекции своей зоны, не выносится в отдельный раздел.
- Если страница тематически пересекает зоны — в `index.md` она лежит в одной зоне, а cross-link идёт через секцию `## Связи` внутри самой страницы.
- Подсекции Java Backend и пустые секции System Design / Patterns / SQL Practice — нормальный placeholder, наполняются по мере ingest.

## Log Format (`wiki/log.md`)

```markdown
## [ACTION] Title

- Источник: `path`
- Концепт: `path` — краткое описание
- Обновлены: список файлов

---
```

Actions: `[INGEST]`, `[UPDATE]`, `[CLEANUP]`, `[INIT]`.

## Quality Standards

### Interview Q&A Quality

Bad Q&A (too vague):
```
Q: Что такое MVCC?
A: Это механизм управления конкурентным доступом через версии.
```

Good Q&A (specific, structured, with trade-offs):
```
Q: Как работает MVCC в PostgreSQL?
A: Каждая строка имеет xmin (ID создавшей транзакции) и xmax (ID удалившей).
При UPDATE создаётся новая версия (новый xmin), старая помечается xmax.
Транзакция видит строки где xmin < snapshot и xmax > snapshot.
VACUUM очищает мёртвые версии. Trade-off: readers не блокируют writers,
но без VACUUM — table bloat.
```

### Content Quality

- **Explain WHY, not just WHAT.** "PostgreSQL хранит версии в heap" → объясни последствия (bloat, VACUUM, HOT).
- **Use tables** for comparisons and term definitions.
- **Trade-offs always.** Каждая технология имеет цену — указывай её.
- **Concrete numbers** where possible (порог selectivity ~5-15%, 32-bit xid → ~4 млрд).
- **No copy-paste from docs.** Переработанное объяснение своими словами.

### Naming Conventions

- Filenames: `kebab-case-english.md` (e.g. `postgresql-mvcc-internals.md`).
- Titles: English for concepts, Russian for descriptions.
- Headings inside pages: mix (English terms, Russian explanations).

## Lint / Maintenance

Periodically check:
- Broken links (deleted pages still referenced).
- Pages without "Типичные вопросы на интервью" section (concept, comparison, entity, pattern).
- Pages without "Связи" section.
- Orphan pages (not in index.md).
- Sources without concept/pattern/case pages.
- Outdated information.
- **Zone integrity**:
  - Concept-страница лежит в правильной зоне в `index.md` (см. зональный маршрутизатор в SCHEMA.md).
  - Нет дублирования: одна страница не появляется в двух зонах одновременно.
  - Pattern-страница имеет поле `**Зона:**` в шапке и появляется в секции `## Patterns` индекса, а не в зональной.
  - System design case лежит в `wiki/system-design/` и в секции `## System Design` индекса.
  - SQL practice задача лежит в `wiki/sql-practice/` и в секции `## SQL Practice` индекса.
  - Domain overviews актуальны: `java-backend-fundamentals`, `data-engineering-fundamentals`, `system-design-fundamentals`, `patterns-catalog`, `sql-practice-roadmap`.
  - Cross-zone links корректны: страница в Foundations, на которую ссылаются и backend, и DE концепты, имеет обратные ссылки на оба домена.
- **Page type integrity:**
  - Pattern-страница содержит секции `Проблема` и `Решение` (отличие от concept).
  - System design case содержит functional/non-functional/capacity/API/data-model/high-level/trade-offs.
  - SQL practice задача содержит DDL входных данных, ожидаемый результат, решение, EXPLAIN.
- **Backlog ↔ Log синхронизация:**
  - Каждый закрытый источник в `backlog.md` (статус `[x]`) имеет запись `[INGEST]` в `log.md`.
  - Новые ingest'ы не игнорируют backlog (если книга была в плане — статус обновляется).

For detailed page type rules and zone routing, see [SCHEMA.md](../../../schema/SCHEMA.md).
