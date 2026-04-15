# PostgreSQL 17 изнутри

| Поле | Значение |
|------|----------|
| **Автор** | Егор Рогов |
| **Издательство** | Postgres Professional |
| **Издание** | 2025 (PostgreSQL 17) |
| **Тип** | книга (бесплатная, PDF) |

## О книге

Глубокое техническое руководство по внутреннему устройству PostgreSQL. Не «как пользоваться», а **как работает**: от страниц и версий строк до планировщика запросов и типов индексов. Основной фокус — **MVCC и изоляция транзакций** (почти треть книги), буферный кеш, WAL, блокировки, выполнение запросов.

## Структура

### Часть I. Изоляция и многоверсионность

1. Изоляция — см. [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md)
2. Страницы и версии строк
3. Снимки данных
4. Внутристраничная очистка и HOT-обновления
5. Очистка и автоочистка (VACUUM)
6. Заморозка (freeze)
7. Перестроение таблиц и индексов

### Часть II. Буферный кеш и журнал

8. Буферный кеш (shared buffers, clock-sweep eviction)
9. Журнал предзаписи (WAL internals)
10. Режимы журнала (synchronous, asynchronous, minimal)

### Часть III. Блокировки

11. Блокировки отношений (table-level locks)
12. Блокировки строк (row-level locks, multixact)
13. Блокировки разных объектов (advisory, predicate)
14. Блокировки в памяти (lightweight locks, spinlocks)

### Часть IV. Выполнение запросов

15. Этапы выполнения запросов (parse → rewrite → plan → execute)
16. Статистика (pg_statistic, гистограммы, MCV)
17. Табличные методы доступа (seq scan, heap)
18. Индексные методы доступа
19. Индексное сканирование (index scan, index-only scan, bitmap scan)
20. Вложенный цикл (nested loop join)
21. Хеширование (hash join, hash aggregate)
22. Сортировка и слияние (sort, merge join)

### Часть V. Типы индексов

23. Хеш-индекс
24. B-дерево (B-tree)
25. Индекс GiST
26. Индекс SP-GiST
27. Индекс GIN
28. Индекс BRIN

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| MVCC, snapshot isolation, VACUUM, xmin/xmax | [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md) |
| Сравнение MVCC: PostgreSQL vs MySQL | [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md) |
| Транзакции и изоляция (DDIA) | [Transactions and Isolation](../concepts/transactions-and-isolation.md) |
| PostgreSQL (entity) | [PostgreSQL](../entities/postgresql.md) |
| Storage internals (DDIA) | [Storage and Retrieval](../concepts/storage-and-retrieval.md) |

## Связанные обзоры

- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md)
