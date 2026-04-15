# SQL Query Execution and Indexes

**Источники:** [Реляционные базы данных в примерах](../sources/relational-databases-in-examples.md) (раздел 2), [PostgreSQL 17 изнутри](../sources/postgresql-internals.md) (части IV–V)

## Суть

СУБД принимает SQL-запрос (declarative) и строит **план выполнения** (imperative дерево операций). Оптимизатор выбирает методы доступа, порядок JOIN, использование индексов — всё на основе **статистики** и **стоимостной модели**. Индексы — ключевой инструмент ускорения чтения, но у каждого типа свои ограничения. Составные индексы работают по правилу **leftmost prefix**.

## Как СУБД выполняет запрос

### Pipeline

```
SQL-текст → Parse → Rewrite → Optimize → Execute → Result
```

| Этап | Что происходит |
|------|----------------|
| **Parse** | Лексический и синтаксический разбор SQL → parse tree. Проверка синтаксиса, разрешение имён таблиц/столбцов |
| **Rewrite** | Применение правил: раскрытие VIEW (view = stored query), применение rules и security policies |
| **Optimize** | Cost-based optimizer: перебирает альтернативные планы, оценивает стоимость каждого через статистику. Выбирает cheapest |
| **Execute** | Volcano/iterator model: каждый узел плана запрашивает следующую строку у дочернего узла (pull-based). Результат — поток строк |

### Cost-based optimizer

Оптимизатор оценивает стоимость плана через:

| Параметр | Откуда | Влияние |
|----------|--------|---------|
| **Selectivity** | Статистика: гистограммы, MCV (most common values), n_distinct | Определяет, сколько строк пройдёт фильтр → выбор index scan vs seq scan |
| **Cardinality** | Оценка количества строк на каждом этапе плана | Влияет на выбор join-стратегии и порядок join |
| **Cost model** | seq_page_cost, random_page_cost, cpu_tuple_cost и др. | Переводит I/O и CPU в единую «стоимость» для сравнения планов |
| **Статистика** | `pg_statistic` / `ANALYZE`. Гистограммы, корреляция, null_frac | Без актуальной статистики оптимизатор выбирает плохие планы |

**ANALYZE** (PostgreSQL) / **UPDATE STATISTICS** (SQL Server) — команда для сбора статистики. Autovacuum в PostgreSQL запускает ANALYZE автоматически.

## EXPLAIN — чтение плана запроса

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...
```

| Поле | Значение |
|------|----------|
| **cost=0.00..123.45** | Оценка стоимости: startup cost..total cost (в условных единицах) |
| **rows=1000** | Оценка количества строк (без ANALYZE — прогноз, с ANALYZE — факт) |
| **actual time=0.05..12.3** | Реальное время выполнения (мс), только с ANALYZE |
| **loops=1** | Сколько раз узел выполнялся (nested loop → inner child выполняется много раз) |
| **Buffers: shared hit=50 read=10** | Сколько страниц из кеша (hit) и с диска (read) |

### Основные узлы плана

| Узел | Когда используется |
|------|--------------------|
| **Seq Scan** | Полное сканирование таблицы. Нет подходящего индекса или selectivity < ~5-10% |
| **Index Scan** | Индексный lookup → получение строки из heap по ctid. Высокая selectivity |
| **Index Only Scan** | Все нужные столбцы в индексе → heap fetch не нужен (если страница all-visible) |
| **Bitmap Index Scan → Bitmap Heap Scan** | Средняя selectivity или OR-условия. Bitmap из ctid → сортировка по страницам → sequential heap I/O |
| **Nested Loop** | Для каждой строки outer — lookup в inner. Хорошо когда inner маленький или есть индекс |
| **Hash Join** | Build hash table по меньшей таблице → probe по большей. Хорошо для equality join без индекса |
| **Merge Join** | Обе стороны отсортированы (или сортируем) → merge. Хорошо для больших таблиц с индексом |
| **Sort** | Сортировка (в памяти или на диске если > work_mem) |
| **HashAggregate / GroupAggregate** | Агрегация: hash (быстро, нужна память) или sort-based (экономнее по памяти) |

## Индексы

### Зачем нужен индекс

Без индекса: **Seq Scan** — чтение всей таблицы O(N).
С индексом: **Index Scan** — B-tree lookup O(log N) + fetch нужных строк.

Trade-off: индекс ускоряет SELECT, но **замедляет INSERT/UPDATE/DELETE** (нужно поддерживать структуру индекса).

### B-tree — основной тип

Сбалансированное дерево. Поддерживает: `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, `IN`, `IS NULL`, `ORDER BY`, `MIN/MAX`.

Не поддерживает: `LIKE '%abc'` (wildcard в начале), функции над столбцом (`WHERE UPPER(name) = 'FOO'` — если нет functional index).

### Простой vs составной индекс

**Простой индекс** — на один столбец:

```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

Ускоряет: `WHERE customer_id = 42`, `ORDER BY customer_id`.

**Составной (composite) индекс** — на несколько столбцов:

```sql
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);
```

#### Правило leftmost prefix

Составной индекс `(A, B, C)` может быть использован для:

| Условие | Используется? | Почему |
|---------|--------------|--------|
| `WHERE A = 1` | Да | Leftmost prefix |
| `WHERE A = 1 AND B = 2` | Да | Prefix (A, B) |
| `WHERE A = 1 AND B = 2 AND C = 3` | Да | Полный индекс |
| `WHERE B = 2` | Нет | Нет leading column A |
| `WHERE A = 1 AND C = 3` | Частично | Использует A, но C — skip (зависит от СУБД; PostgreSQL может использовать index для A и фильтровать C) |
| `WHERE A > 1 AND B = 2` | Частично | Range на A → B не может быть использован для точного lookup (range breaks prefix) |
| `ORDER BY A, B` | Да | Совпадает с порядком индекса |
| `ORDER BY B, A` | Нет | Порядок не совпадает |

#### Порядок столбцов в составном индексе

Общее правило для выбора порядка:

1. **Equality-столбцы первыми** — столбцы из `WHERE col = value`.
2. **Range/sort-столбец последним** — столбец из `WHERE col > value` или `ORDER BY col`.
3. **Selectivity** — более селективный столбец раньше (сильнее сужает диапазон).

Пример: запрос `WHERE status = 'active' AND created_at > '2024-01-01' ORDER BY created_at` → индекс `(status, created_at)`, а не `(created_at, status)`.

### Covering index (покрывающий)

Индекс содержит **все** столбцы, нужные запросу → СУБД может ответить из индекса, не обращаясь к таблице (Index Only Scan).

```sql
CREATE INDEX idx_covering ON orders(customer_id, order_date) INCLUDE (total);
```

Запрос `SELECT order_date, total FROM orders WHERE customer_id = 42` → Index Only Scan (в PostgreSQL — при условии, что visibility map позволяет).

### Partial index (частичный)

Индексирует только строки, удовлетворяющие условию:

```sql
CREATE INDEX idx_active_orders ON orders(customer_id) WHERE status = 'active';
```

Меньше размер, быстрее обновление. Полезен для hot data или когда большинство строк не нужны в запросах.

### Functional (expression) index

Индекс на выражение:

```sql
CREATE INDEX idx_lower_email ON users(LOWER(email));
```

Позволяет использовать индекс для `WHERE LOWER(email) = 'foo@bar.com'`. Без такого индекса СУБД не может использовать обычный индекс на `email` для выражений.

### Другие типы индексов

| Тип | Для чего | СУБД |
|-----|----------|------|
| **Hash** | Только equality (`=`). Компактнее B-tree для этого случая | PostgreSQL (с PG 10+ WAL-logged), MySQL |
| **GIN** | Inverted index: массивы, JSONB, full-text search | PostgreSQL |
| **GiST** | Геопространственные, range types | PostgreSQL (PostGIS) |
| **BRIN** | Block range: min/max на группу страниц. Time-series, append-only | PostgreSQL |
| **Bitmap** | Колонки с низкой cardinality. Комбинируется через AND/OR | Oracle, PostgreSQL (как runtime structure, не persistent) |

## Когда индекс НЕ используется

| Причина | Пример |
|---------|--------|
| **Низкая selectivity** | `WHERE gender = 'M'` на таблице 50/50 → seq scan дешевле |
| **Функция на столбце** | `WHERE YEAR(created_at) = 2024` → нужен functional index |
| **Неявное приведение типов** | `WHERE varchar_col = 123` → СУБД может cast'ить столбец, ломая индекс |
| **Leading column отсутствует** | Composite index `(A, B)`, запрос `WHERE B = 5` |
| **OR между разными столбцами** | `WHERE A = 1 OR B = 2` → для каждого столбца нужен свой индекс + bitmap merge |
| **Маленькая таблица** | Seq scan одной страницы быстрее index lookup |
| **Устаревшая статистика** | Оптимизатор не знает реальное распределение → выбирает seq scan. Решение: `ANALYZE` |

## Стратегии JOIN

| Стратегия | Алгоритм | Когда выбирается |
|-----------|----------|------------------|
| **Nested Loop** | Для каждой строки outer → lookup в inner | Inner маленький, или есть индекс на join key inner |
| **Hash Join** | Build hash по меньшей таблице → probe по большей | Equality join, нет индекса, обе таблицы средние/большие |
| **Merge Join** | Обе стороны отсортированы → merge | Обе таблицы большие и уже отсортированы (индекс) или sort дёшев |

Оптимизатор также выбирает **порядок JOIN** (для N таблиц — N! вариантов; при > 8-12 таблицах используются эвристики вместо полного перебора).

## Типичные вопросы на интервью

**Q: Как СУБД решает, использовать индекс или seq scan?**
A: Cost-based optimizer оценивает стоимость обоих вариантов через статистику. Если selectivity высокая (мало строк) — index scan дешевле (меньше random I/O). Если selectivity низкая (много строк) — seq scan дешевле (sequential I/O быстрее, чем много random reads). Порог обычно ~5-15% строк таблицы.

**Q: Как работает составной индекс и что такое leftmost prefix?**
A: Составной индекс `(A, B, C)` — B-tree, отсортированный по A, затем по B внутри каждого A, затем по C. Поэтому поиск работает слева направо: `WHERE A = 1` — ОК, `WHERE A = 1 AND B = 2` — ОК, `WHERE B = 2` — индекс бесполезен (нет leading column). Range-условие «ломает» использование следующих столбцов: `WHERE A > 1 AND B = 2` → A через range scan, B не может точно отфильтроваться через индекс.

**Q: Как выбрать порядок столбцов в составном индексе?**
A: Equality-столбцы первыми (сужают диапазон точно), range/sort-столбец последним. Более селективный — раньше. Пример: `WHERE tenant_id = X AND created_at > Y ORDER BY created_at` → `(tenant_id, created_at)`.

**Q: Что такое покрывающий индекс?**
A: Индекс, содержащий все столбцы запроса (в ключевой части или через INCLUDE). СУБД возвращает данные прямо из индекса без обращения к heap-таблице (Index Only Scan). Ускоряет чтение, но увеличивает размер индекса и стоимость записи.

**Q: Почему запрос не использует индекс, хотя он есть?**
A: 1) Низкая selectivity — seq scan дешевле. 2) Функция на столбце (`WHERE UPPER(name) = ...`) — нужен functional index. 3) Implicit type cast. 4) Нет leading column составного индекса. 5) Устаревшая статистика — выполнить ANALYZE. 6) Маленькая таблица — одна страница, seq scan быстрее.

**Q: В чём разница между Hash Join, Nested Loop и Merge Join?**
A: **Nested Loop**: O(N × M), но с индексом на inner — O(N × log M). Лучший для маленького inner. **Hash Join**: O(N + M), строим hash по меньшей таблице. Лучший для equality join без индекса. **Merge Join**: O(N log N + M log M) с сортировкой, O(N + M) если уже отсортированы. Лучший для больших отсортированных datasets.

**Q: Как понять, что индекс не нужен или вреден?**
A: Проверить `pg_stat_user_indexes.idx_scan` — если 0 за длительный период, индекс не используется. Каждый лишний индекс: +время на INSERT/UPDATE/DELETE, +место на диске, +работа для VACUUM. На write-heavy таблицах минимизировать количество индексов.

## Связи

- [Storage and Retrieval](storage-and-retrieval.md) — B-tree vs LSM, физическое хранение.
- [PostgreSQL MVCC Internals](postgresql-mvcc-internals.md) — query pipeline и индексы в PostgreSQL.
- [Normalization and Normal Forms](normalization-and-normal-forms.md) — нормализация → больше таблиц → больше JOIN → важнее индексы.
- [PostgreSQL](../entities/postgresql.md) — типы индексов (B-tree, GIN, GiST, BRIN).
- [SQL Transactions, Locking and Isolation](sql-transactions-locking-standard.md) — блокировки при чтении/записи индексов.
