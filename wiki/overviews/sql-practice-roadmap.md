# SQL Practice Roadmap

**Каркас раздела** [SQL Practice](../sql-practice/README.md). Раздел в стадии наполнения — упражнения добавляются постепенно (см. [backlog](../backlog.md) → SQL Practice).

## Зачем эта страница существует

[`sql-practice/README.md`](../sql-practice/README.md) — landing с категориями и шаблоном задачи.

Эта overview-страница — **порядок изучения и approach**: с чего начать, какие категории осваивать когда, какие концепты-страницы повторить перед каждой темой.

## Опорные источники

См. [backlog](../backlog.md) → раздел SQL Practice. Главные:

- LeetCode Database section — топ-50.
- StrataScratch / DataLemur — задачи реальных DE-интервью.
- PostgreSQL Documentation — Window Functions, Recursive CTE.
- *SQL Antipatterns* — Bill Karwin (для понимания типичных ошибок).

## Порядок изучения

### Этап 1. Refresher по join'ам и агрегациям (easy)

**Сначала:** перечитать [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md).

Задачи (TBD):

- (TBD) INNER vs LEFT vs SEMI vs ANTI join — задачка с неоднозначностями.
- (TBD) GROUP BY + HAVING — фильтрация на агрегатах.
- (TBD) NULL-семантика в join'ах и WHERE.

### Этап 2. Window functions (medium → must-know для DE)

**Сначала:** прочитать раздел Window Functions в PG docs.

Задачи (TBD):

- (TBD) Top-N per group — три способа.
- (TBD) Running totals.
- (TBD) Moving averages с frame clause.
- (TBD) LAG/LEAD: разница между соседними строками.

### Этап 3. Аналитические задачи (medium → hard)

Задачи (TBD):

- (TBD) Sessionization (gaps & islands).
- (TBD) Retention cohort matrix.
- (TBD) Funnel.
- (TBD) Detect overlapping intervals.

### Этап 4. Recursive CTE и иерархии (hard)

Задачи (TBD):

- (TBD) Recursive org chart traversal.
- (TBD) Category tree (with depth).
- (TBD) BFS поиск по графу.

### Этап 5. Schema design + DDL

Задачи (TBD):

- (TBD) Спроектировать схему под use case.
- (TBD) SCD Type 2 — реализация upsert'а в SQL.

### Этап 6. Performance & EXPLAIN

**Сначала:** освежить [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md), [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md).

Задачи (TBD):

- (TBD) Анализ медленного запроса по `EXPLAIN ANALYZE` → переписать.
- (TBD) Какой индекс закроет этот запрос.
- (TBD) Когда не помогает индекс (low selectivity, function on column).

## Approach к решению задачи

1. **Прочитать → переформулировать.** Что является «группой», что — «событием», какая гранулярность результата.
2. **Выбрать инструмент.** GROUP BY / window / recursive CTE / lateral.
3. **Написать решение.** Прогнать на seed-данных.
4. **Проверить edge cases.** NULL, пустые группы, дубли, ties.
5. **EXPLAIN ANALYZE.** Понять план. Подумать про индексы.
6. **Альтернативное решение.** Часто на интервью просят 2 способа — сравнить по читаемости и производительности.

## Связи

- [SQL Practice (раздел)](../sql-practice/README.md) — landing + шаблон.
- [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md) — теоретическая база.
- [Normalization and Normal Forms](../concepts/normalization-and-normal-forms.md) — для schema-design задач.
- [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md) — для performance задач.
- [Backlog](../backlog.md) — план добавления.
