# PostgreSQL MVCC Internals

**Источник:** [PostgreSQL 17 изнутри](../sources/postgresql-internals.md) (часть I)

## Суть

PostgreSQL реализует **MVCC** (Multi-Version Concurrency Control) через хранение **нескольких физических версий каждой строки** прямо в heap-страницах. Каждая версия (tuple) помечена `xmin`/`xmax` — ID транзакций создания и удаления. Видимость определяется **снимком** (snapshot): набором активных транзакций на момент начала запроса/транзакции. Мёртвые версии копятся и убираются **VACUUM**. Это архитектурный выбор с серьёзными последствиями: readers никогда не блокируют writers, но таблица раздувается без регулярной очистки.

## Версии строк: xmin, xmax, ctid

| Поле | Назначение |
|------|------------|
| **xmin** | ID транзакции, создавшей эту версию строки (INSERT или UPDATE → новая версия) |
| **xmax** | ID транзакции, «убившей» эту версию (DELETE или UPDATE → пометка старой). 0 = версия живая |
| **ctid** | Физический адрес (page, offset). При UPDATE старая версия хранит ctid новой → цепочка версий |
| **infomask** | Битовые флаги: committed, aborted, frozen, has nulls и др. Оптимизация — не нужно лезть в pg_xact каждый раз |

**UPDATE** в PostgreSQL — это фактически **DELETE старой версии + INSERT новой**. Старый tuple получает xmax = текущая транзакция, новый tuple получает xmin = текущая транзакция. Старый ctid указывает на новый.

## Снимки данных (snapshots)

Snapshot определяет, какие транзакции видимы для текущего запроса:

```
snapshot = (xmin, xmax, xip_list)
```

| Компонент | Смысл |
|-----------|-------|
| **xmin** | Самая старая активная транзакция на момент создания снимка. Всё, что < xmin и committed — видимо |
| **xmax** | Следующий ID после последнего назначенного. Всё ≥ xmax — точно невидимо |
| **xip_list** | Список активных (in-progress) транзакций между xmin и xmax — их результаты невидимы |

**Правило видимости tuple:** версия видима, если `xmin` committed и попадает в snapshot, а `xmax` либо отсутствует, либо не committed, либо не попадает в snapshot.

### Снимки и уровни изоляции

| Уровень | Когда создаётся snapshot |
|---------|--------------------------|
| **Read Committed** | В начале **каждого statement** — поэтому два SELECT в одной транзакции могут видеть разные данные |
| **Repeatable Read** | В начале **транзакции** — все statements видят один и тот же snapshot |
| **Serializable** | Как Repeatable Read + SSI (Serializable Snapshot Isolation) — отслеживание rw-зависимостей, abort при аномалиях |

## HOT Updates (Heap-Only Tuples)

Если обновление **не затрагивает индексированные столбцы** и новая версия помещается **на ту же страницу**:

- Новый tuple — HOT (heap-only), не попадает ни в один индекс.
- Index scan находит старый tuple → по ctid-цепочке добирается до актуальной версии.
- При очистке страницы (pruning) цепочка сворачивается, старые версии удаляются, ctid перенаправляется.

HOT экономит index maintenance. Без HOT каждый UPDATE требует обновления **каждого** индекса на таблице.

## VACUUM

VACUUM — основной механизм «уборки мусора». PostgreSQL не перезаписывает строки in-place → мёртвые версии копятся → **table bloat**.

| Операция | Что делает |
|----------|------------|
| **VACUUM (обычный)** | Сканирует таблицу, помечает мёртвые tuple в free space map. Не возвращает место ОС, но позволяет переиспользовать пространство |
| **VACUUM FULL** | Полная перезапись таблицы. Освобождает место на диске. **ACCESS EXCLUSIVE LOCK** — блокирует всё |
| **Autovacuum** | Фоновый демон. Запускается когда `n_dead_tup > threshold + scale_factor × n_live_tup`. Настройка per-table возможна |

### Freeze

Каждая транзакция имеет 32-битный `xid` → wraparound через ~4 млрд транзакций. VACUUM **замораживает** старые tuple (infomask = frozen), делая их видимыми для всех будущих транзакций без проверки xid. Если замораживание не успевает — PostgreSQL **отказывается принимать новые транзакции** (аварийный режим).

## Блокировки строк

| Режим | Что блокирует | Пример |
|-------|---------------|--------|
| **FOR KEY SHARE** | Минимальный: блокирует только удаление/обновление PK | FK check |
| **FOR SHARE** | Блокирует UPDATE и DELETE | Чтение с гарантией неизменности |
| **FOR NO KEY UPDATE** | Блокирует другие UPDATE и DELETE, но не FOR KEY SHARE | Обычный UPDATE |
| **FOR UPDATE** | Эксклюзив: блокирует всё | SELECT ... FOR UPDATE |

Блокировки строк реализованы через `xmax` + infomask-флаги, а не через отдельную таблицу локов. Когда нескольким транзакциям нужна shared-блокировка — используется **multixact** (массив транзакций вместо одного xmax).

## Буферный кеш и WAL

| Механизм | Суть |
|----------|------|
| **Shared buffers** | Кеш страниц в shared memory. По умолчанию 128 MB (рекомендация: 25% RAM). Алгоритм вытеснения — **clock-sweep** (аналог LRU с меньшим overhead) |
| **WAL** | Все изменения сначала пишутся в журнал предзаписи, потом страницы модифицируются в буферном кеше. При crash — recovery из WAL. Также основа для streaming replication |
| **Checkpoint** | Фоновый процесс сбрасывает грязные страницы из кеша на диск. После checkpoint WAL-сегменты до него можно удалить |
| **Full page writes** | После checkpoint первая запись в страницу сохраняет **полную копию** страницы в WAL (защита от torn page при crash) |

## Выполнение запросов

Pipeline: **parse → rewrite → optimize → execute**.

| Этап | Что происходит |
|------|----------------|
| **Parse** | SQL → parse tree. Синтаксический разбор |
| **Rewrite** | Применение rules (VIEW = rule на SELECT) |
| **Optimize** | Cost-based optimizer. Строит дерево планов, оценивает стоимость через статистику (`pg_statistic`). Выбирает cheapest path |
| **Execute** | Volcano model (iterator): каждый узел дерева отдаёт по одной строке вверх |

### Основные методы доступа

| Метод | Когда |
|-------|-------|
| **Seq Scan** | Нет подходящего индекса или selectivity низкая (читать таблицу целиком дешевле) |
| **Index Scan** | Один индексный lookup → heap fetch по ctid. Хорошо для высокой selectivity |
| **Index-Only Scan** | Все нужные столбцы в индексе + visibility map подтверждает, что страница all-visible → heap fetch не нужен |
| **Bitmap Scan** | Несколько условий или средняя selectivity: bitmap из ctid → сортировка по страницам → seq I/O по heap |

## Типы индексов

| Индекс | Для чего |
|--------|----------|
| **B-tree** | Equality, range, ORDER BY. По умолчанию. Balanced tree, O(log N) |
| **Hash** | Только equality. Компактнее B-tree для этого случая. С PG 10+ WAL-logged |
| **GiST** | Геопространственные данные, range types, full-text. Generalized Search Tree |
| **SP-GiST** | Space-partitioned: quad-tree, k-d tree, radix tree. Для неравномерных данных |
| **GIN** | Inverted index: массивы, JSONB, full-text search. Быстрый поиск, дорогой UPDATE |
| **BRIN** | Block Range Index: min/max на группу страниц. Для time-series, append-only. Компактный, но неточный |

## Типичные вопросы на интервью

**Q: Как именно PostgreSQL хранит версии строк?**
A: Прямо в heap-страницах таблицы. Каждая версия — отдельный tuple с `xmin` (кто создал) и `xmax` (кто удалил/обновил). UPDATE = INSERT новой версии + пометка xmax на старой. Старая версия хранит ctid новой, образуя цепочку. Мёртвые версии убирает VACUUM.

**Q: Чем опасен table bloat и как с ним бороться?**
A: Без VACUUM мёртвые tuple копятся → таблица растёт → seq scan медленнее, кеш менее эффективен. Бороться: 1) настроить autovacuum агрессивнее на горячих таблицах (`autovacuum_vacuum_scale_factor`), 2) мониторить `pg_stat_user_tables.n_dead_tup`, 3) в крайнем случае `VACUUM FULL` или `pg_repack` (без exclusive lock).

**Q: Что такое transaction ID wraparound и почему это опасно?**
A: `xid` — 32-битный, ~4 млрд значений. Без заморозки через ~2 млрд транзакций старые xid «обгонят» текущий → строки станут «из будущего» и невидимыми. VACUUM freeze помечает старые tuple как frozen (видимы всегда). Если freeze не успевает — PostgreSQL переходит в **emergency mode** и отказывается от новых транзакций пока не будет выполнен VACUUM.

**Q: Почему UPDATE в PostgreSQL дороже, чем в MySQL?**
A: PostgreSQL создаёт **полную новую версию строки** + обновляет все индексы (кроме HOT case). MySQL InnoDB обновляет строку **in-place** в clustered index, а старую версию пишет в undo log. Поэтому при write-heavy нагрузке PostgreSQL быстрее раздувается (bloat) и требует VACUUM, а InnoDB — нет.

**Q: Как работает SSI (Serializable Snapshot Isolation)?**
A: Поверх snapshot isolation отслеживаются **rw-зависимости** между транзакциями. Если три транзакции образуют «dangerous structure» (цикл rw-зависимостей), одна из них абортится. Это optimistic: не блокирует, но может откатить. Дешевле 2PL, но требует retry-логики в приложении.

**Q: Когда Index-Only Scan действительно работает?**
A: Когда: 1) все нужные столбцы есть в индексе, 2) страницы heap помечены как all-visible в **visibility map** (т.е. все tuple на странице видимы для всех). Если страница «грязная» (есть невидимые версии) — нужен heap fetch для проверки видимости. VACUUM обновляет visibility map → чем чаще VACUUM, тем чаще Index-Only Scan эффективен.

## Связи

- [PostgreSQL](../entities/postgresql.md) — entity-страница.
- [SQL Transactions, Locking and Isolation](sql-transactions-locking-standard.md) — SQL-стандарт: блокировки, 2PL, deadlocks.
- [Transactions and Isolation](transactions-and-isolation.md) — уровни изоляции в контексте DDIA.
- [Storage and Retrieval](storage-and-retrieval.md) — индексы, LSM vs B-tree.
- [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md) — сравнение реализаций.
