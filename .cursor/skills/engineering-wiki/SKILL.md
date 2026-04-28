---
name: engineering-wiki
description: Manage a personal engineering knowledge base for interview prep and learning. Use when the user wants to add sources (books, articles, courses), create or update concept/entity/comparison pages, review for interviews, check wiki integrity, or asks about wiki structure. Activates on any mention of wiki, knowledge base, interview prep, or adding engineering knowledge.
---

# Engineering Wiki — Personal Knowledge Base

Персональная вики для подготовки к собеседованиям и систематизации инженерных знаний.
Язык вики — **русский** (термины на английском, объяснения на русском).

Wiki ориентирована на **два рабочих домена**: Java Backend и Data Engineering. Общий материал (распределённые системы, транзакции, SQL, моделирование) живёт в зоне **Foundations**. См. [SCHEMA.md](../../../schema/SCHEMA.md) — там зафиксированы зоны, подсекции и зональный маршрутизатор для классификации новых страниц.

## Core Principles

1. **Interview-first**: каждая concept-страница содержит секцию "Типичные вопросы на интервью" с Q&A.
2. **Deep understanding**: не пересказ документации, а объяснение **как работает под капотом** и **почему** выбран такой дизайн.
3. **Cross-linked**: страницы связаны. Нет изолированных страниц.
4. **Structured consistently**: все страницы одного типа следуют одному шаблону.
5. **Zone-routed**: каждая concept/comparison-страница принадлежит ровно одной зоне (Foundations / Data Engineering / Java Backend). Дублирование ссылок в `index.md` запрещено; пересечения — через `## Связи`.

## Wiki Zones (трёхзонная модель)

Полное описание — в `schema/SCHEMA.md` (раздел 1). Краткая сводка для оперативного решения:

| Зона | Подсекции | Что туда идёт |
|------|-----------|----------------|
| **Foundations** | Distributed Systems & System Design / Transactions, Isolation & Storage Internals / SQL & Relational Modeling | Replication, partitioning, consensus, encoding/schema evolution, transactions, MVCC, locking, PG/MySQL internals, SQL execution, indexes, normalization, data models |
| **Data Engineering** | DDIA-data chapters / Kimball / ClickHouse / dbt / Spark / Stream Processing (Flink) | DWH, dimensional modeling, columnar engines, batch/stream pipelines, lakehouse, CDC |
| **Java Backend** | JVM internals / Concurrency & Memory Model / Spring / Kotlin / Web & APIs / Observability | GC, JIT, JMM, virtual threads, Spring (любой модуль), Kotlin coroutines, REST/gRPC/OpenAPI, Micrometer/OpenTelemetry |

**Принцип определения зоны для новой concept-страницы:**

1. Если страница объясняет **общее свойство данных или распределённых систем** (replication, transactions, MVCC, normalization, data models) → Foundations.
2. Если объясняет **конкретный DE-инструмент или DE-паттерн** (ClickHouse engines, dbt layers, Flink windows, dimensional modeling) → Data Engineering.
3. Если объясняет **JVM / Spring / Kotlin / Java-concurrency / web API** → Java Backend.
4. Если страница попадает в две зоны — выбирается та, где она наиболее «корневая» (где её используют для понимания базы), а во второй зоне ставится cross-link через `## Связи`.

**Multi-zone ingest:** одна книга может породить страницы в разных зонах (например, DDIA → Foundations + Data Engineering). При ingest концепты разносятся по соответствующим зонам, а не складываются в одну.

## File Structure

```
wiki/
├── index.md                    # Master index — три зоны (Foundations / DE / Java Backend) + Sources + Entities + Overviews
├── log.md                      # Changelog — каждое изменение фиксируется
├── sources/                    # Книги, статьи, курсы, документация
├── concepts/                   # Концепты, паттерны, технологии (плоская папка; зона определяется в index.md)
├── entities/                   # Конкретные инструменты (PostgreSQL, Kafka...)
├── comparisons/                # Сравнения (X vs Y)
└── overviews/                  # Обзоры тем, learning tracks (включая java-backend-fundamentals.md и data-engineering-fundamentals.md)
schema/
└── SCHEMA.md                   # Правила структуры: зоны, types of pages, rules
```

Папки `concepts/`, `comparisons/`, `entities/` — плоские. Зона страницы определяется не папкой, а её положением в `wiki/index.md` (зональный раздел и подсекция).

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

## Ingest Workflow (adding a new source)

When the user says they read a book/article/course:

1. **Create source page** in `wiki/sources/` with metadata and structure. Указать в `wiki/index.md` рядом со ссылкой зону (Foundations / DE / Backend / mixed).
2. **Classify topics by zone.** Перед созданием concept-страниц определить, в какую зону попадает каждая выделенная тема (Foundations / Data Engineering / Java Backend) — см. таблицу выше и SCHEMA.md раздел 1. Multi-zone ingest — норма (одна книга → концепты в разных зонах).
3. **Create concept pages** for key topics. Each concept:
   - Has "Суть" section (concise summary).
   - Has "Типичные вопросы на интервью" (3-5 realistic Q&A).
   - Has "Связи" linking to existing wiki pages — особенно cross-zone links, если концепт пересекается с другим доменом.
   - References the source via `**Источник:**`.
4. **Create comparison pages** if the source reveals X vs Y contrasts. Comparison идёт в зону сравниваемых концептов (или более узкоспециализированного из них).
5. **Update existing pages** — add cross-links from related concepts (включая cross-zone, если уместно).
6. **Update `wiki/index.md`** — add source and concepts to **correct zone and subsection**. Не дублировать ссылку в нескольких зонах.
7. **Update domain overview**:
   - Если ingest добавил DE-материал → обновить `wiki/overviews/data-engineering-fundamentals.md`.
   - Если backend-материал → обновить `wiki/overviews/java-backend-fundamentals.md`.
   - Если foundations-материал → обновить блок «Связанные foundations» в обоих overview, если новая тема того стоит.
8. **Update `wiki/log.md`** — add `## [INGEST] Source Name` entry with list of created/updated files **и затронутых зон**.

**Important:** if the user mentions specific topics they liked (e.g. "мне понравились транзакции и индексы"), focus concept pages on those topics with extra depth.

## Index Structure (`wiki/index.md`)

Index организован по **зонам**, не по типам страниц:

```markdown
## Domains (overviews)
- Java Backend Fundamentals
- Data Engineering Fundamentals

## Sources
| Источник | Зона |
- table: source name + zone (Foundations / DE / Backend / mixed)

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

## Java Backend
### JVM internals / Concurrency / Spring / Kotlin / Web & APIs / Observability
(пустые подсекции — placeholder, наполняются по мере ingest)

## Entities
- entity name — пометка зоны (Foundations / DE / Backend / mixed)

## Overviews
- domain overviews + interview-prep + learning tracks
```

**Правила index'а:**
- Каждая concept/comparison-страница появляется в `index.md` **ровно один раз** — в своей зоне.
- Comparison живёт внутри своей подсекции, не выносится в отдельный раздел.
- Если страница тематически пересекает зоны — в `index.md` она лежит в одной зоне, а cross-link идёт через секцию `## Связи` внутри самой страницы.
- Подсекции Java Backend могут быть пустыми, пока в зоне нет страниц — это нормально, они служат placeholder'ом для будущего ingest.

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
- Pages without "Типичные вопросы на интервью" section.
- Pages without "Связи" section.
- Orphan pages (not in index.md).
- Sources without concept pages.
- Outdated information.
- **Zone integrity**:
  - Concept-страница лежит в правильной зоне в `index.md` (см. зональный маршрутизатор в SCHEMA.md).
  - Нет дублирования: одна страница не появляется в двух зонах одновременно.
  - Domain overviews (`java-backend-fundamentals.md`, `data-engineering-fundamentals.md`) актуальны и содержат все страницы своей зоны.
  - Cross-zone links корректны: страница в Foundations, на которую ссылаются и backend, и DE концепты, имеет обратные ссылки на оба домена.

For detailed page type rules and zone routing, see [SCHEMA.md](../../../schema/SCHEMA.md).
