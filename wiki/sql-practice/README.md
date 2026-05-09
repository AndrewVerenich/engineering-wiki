# SQL Practice

Раздел **практических SQL-задач** — отдельный от `concepts/sql-*` (теория) и от dimensional/clickhouse концептов (применение).

## Зачем отдельный раздел

Концепт-страницы в `concepts/` отвечают на вопрос **«как работает SQL/индексы/EXPLAIN»**. Этот раздел отвечает на вопрос **«как написать запрос для задачи X»**, который часто и есть формат интервью на DE/senior-backend позиции.

## Категории и сложность

| Категория | Сложность | Что тренируем |
|-----------|-----------|----------------|
| **Joins & filters** | easy / medium | INNER/LEFT/SEMI/ANTI joins, EXISTS vs IN, NULL-семантика |
| **Aggregations & GROUP BY** | easy / medium | HAVING, FILTER, GROUPING SETS, ROLLUP / CUBE |
| **Window functions** | medium / hard | ROW_NUMBER, RANK, LAG/LEAD, FIRST_VALUE, frame clause |
| **Top-N per group** | medium | разные варианты: window vs LATERAL vs DISTINCT ON |
| **Recursive CTE** | medium / hard | иерархии, графы, генерация серий, BFS-обходы |
| **Gaps & islands** | medium / hard | непрерывные интервалы, sessionization |
| **Pivot / unpivot** | medium | в чистом SQL без расширений |
| **Date / time arithmetic** | medium | bucketing, business days, calendar dimensions |
| **Analytics tasks** | hard | retention, funnel, cohort, RFM |
| **Performance / EXPLAIN** | hard | переписать запрос ради плана; индексы |
| **Schema design tasks** | medium / hard | DDL, normalization, SCD, slowly-changing-fact |

## Текущие задачи

(пусто — наполняется)

Планируемые задачи (см. `wiki/backlog.md`):

- Top-N per group (window vs LATERAL vs DISTINCT ON)
- Sessionization (gaps & islands) — пользовательские сессии из event-лога
- Retention cohort matrix
- Funnel из event-stream
- Recursive org chart traversal
- Pivot months → columns без расширений
- Median / percentiles в чистом SQL
- Running totals + window frames
- Detect duplicate / overlapping intervals
- Slowly Changing Dimension Type 2: SQL-реализация upsert'а

## Шаблон задачи

Filename: `kebab-case-english.md` (например, `top-n-per-group.md`, `sessionization.md`).

```markdown
# SQL Practice: <Название задачи>

**Категория:** window functions / recursive CTE / gaps & islands / ...  
**Сложность:** easy / medium / hard  
**Диалект:** PostgreSQL (если решение специфично) / ANSI

## Задача

Описание словами + бизнес-контекст.

## Входные данные

```sql
-- DDL таблиц + INSERT'ы тестовых данных
CREATE TABLE ...;
INSERT INTO ...;
```

## Ожидаемый результат

| col1 | col2 | ... |
|------|------|-----|
| ...  | ...  | ... |

## Решение

```sql
-- основной запрос
```

## Объяснение

Пошагово: что делает каждая часть, почему именно так. Где лежит сложность.

## План выполнения (EXPLAIN)

Что важного в плане: какие joins, sequential vs index scans, сортировки.
Где может сломаться при росте данных.

## Альтернативные решения

- Вариант через LATERAL.
- Вариант через DISTINCT ON (PG-специфичный).
- Вариант через подзапрос с агрегатом.

Сравнение по читаемости / производительности / переносимости.

## Типичные ошибки

- ... (NULL-семантика, дубли, пропуски при LEFT JOIN).

## Когда спрашивают на интервью

- На каких ролях (DE / senior backend / analytics engineer).
- В какой формулировке.

## Связи

- [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md)
- [SQL Transactions, Locking and Isolation](../concepts/sql-transactions-locking-standard.md)
- (другие связанные задачи / концепты)
```

## Подход к решению

1. Прочитать задачу → выписать **что является группой** и **что является событием**.
2. Сформулировать **ожидаемую гранулярность результата** (одна строка на что?).
3. Подобрать инструмент: GROUP BY / window / recursive CTE / lateral.
4. Написать решение → проверить на edge cases (пустые группы, NULL, дубли, ties).
5. Прогнать `EXPLAIN ANALYZE` → понять план → подумать про индексы.
6. Записать альтернативу (если есть) — на интервью часто просят 2 решения.

## Связи

- [SQL Practice Roadmap](../overviews/sql-practice-roadmap.md) — порядок изучения и approach.
- [SQL Query Execution and Indexes](../concepts/sql-query-execution-and-indexes.md) — теоретическая база.
- [Normalization and Normal Forms](../concepts/normalization-and-normal-forms.md) — для schema design задач.
- [Backlog](../backlog.md) — что планируется добавить.
