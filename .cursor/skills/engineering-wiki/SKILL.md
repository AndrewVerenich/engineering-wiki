---
name: engineering-wiki
description: Manage a personal engineering knowledge base for interview prep and learning. Use when the user wants to add sources (books, articles, courses), create or update concept/entity/comparison pages, review for interviews, check wiki integrity, or asks about wiki structure. Activates on any mention of wiki, knowledge base, interview prep, or adding engineering knowledge.
---

# Engineering Wiki — Personal Knowledge Base

Персональная вики для подготовки к собеседованиям и систематизации инженерных знаний.
Язык вики — **русский** (термины на английском, объяснения на русском).

## Core Principles

1. **Interview-first**: каждая concept-страница содержит секцию "Типичные вопросы на интервью" с Q&A.
2. **Deep understanding**: не пересказ документации, а объяснение **как работает под капотом** и **почему** выбран такой дизайн.
3. **Cross-linked**: страницы связаны. Нет изолированных страниц.
4. **Structured consistently**: все страницы одного типа следуют одному шаблону.

## File Structure

```
wiki/
├── index.md                    # Master index — все страницы, сгруппированные по темам
├── log.md                      # Changelog — каждое изменение фиксируется
├── sources/                    # Книги, статьи, курсы, документация
├── concepts/                   # Концепты, паттерны, технологии
├── entities/                   # Конкретные инструменты (PostgreSQL, Kafka...)
├── comparisons/                # Сравнения (X vs Y)
└── overviews/                  # Обзоры тем, learning tracks
schema/
└── SCHEMA.md                   # Правила структуры (types of pages, rules)
```

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

1. **Create source page** in `wiki/sources/` with metadata and structure.
2. **Create concept pages** for key topics. Each concept:
   - Has "Суть" section (concise summary).
   - Has "Типичные вопросы на интервью" (3-5 realistic Q&A).
   - Has "Связи" linking to existing wiki pages.
   - References the source via `**Источник:**`.
3. **Create comparison pages** if the source reveals X vs Y contrasts.
4. **Update existing pages** — add cross-links from related concepts.
5. **Update `wiki/index.md`** — add source and concepts to correct sections.
6. **Update `wiki/log.md`** — add `## [INGEST] Source Name` entry with list of created/updated files.

**Important:** if the user mentions specific topics they liked (e.g. "мне понравились транзакции и индексы"), focus concept pages on those topics with extra depth.

## Index Structure (`wiki/index.md`)

Index groups pages **by topic**, not by type:

```markdown
## Sources
- list of all sources

## Concepts
### Topic Group Name
- concepts for this topic
- comparisons for this topic (inline, not separate section)

## Entities
## Overviews
```

Comparisons live inside their topic group in Concepts, not in a separate section.

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

For detailed page type rules, see [SCHEMA.md](../../../schema/SCHEMA.md).
