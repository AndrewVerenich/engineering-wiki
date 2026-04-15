# SQL Transactions, Locking and Isolation

**Источник:** [SQL: The Complete Reference](../sources/sql-complete-reference.md) (Part IV)

## Суть

SQL-стандарт определяет транзакционную модель через **уровни изоляции** и **допустимые аномалии**. На практике СУБД реализуют эти уровни двумя принципиально разными способами: **блокировками** (2PL — классический подход) или **версионированием** (MVCC — PostgreSQL, InnoDB). Groff подробно описывает обе модели и показывает, почему стандарт и реальность расходятся.

## Транзакционная модель SQL

### Границы транзакции

| Операция | Смысл |
|----------|-------|
| **BEGIN / START TRANSACTION** | Явное начало транзакции |
| **COMMIT** | Фиксация: все изменения становятся видимыми и durable |
| **ROLLBACK** | Откат: все изменения отменяются |
| **SAVEPOINT name** | Точка сохранения внутри транзакции. Можно откатить до неё без отката всей транзакции |
| **ROLLBACK TO SAVEPOINT** | Частичный откат до savepoint. Транзакция продолжается |

**Autocommit:** по умолчанию в большинстве СУБД каждый statement — отдельная транзакция. Для multi-statement транзакций нужен явный BEGIN.

### ACID — что именно гарантирует каждая буква

| Свойство | Стандарт говорит | На практике |
|----------|------------------|-------------|
| **Atomicity** | Транзакция — атомарна: всё или ничего | Реализуется через undo log (InnoDB) или xmin/xmax + rollback (PostgreSQL). При crash — recovery из WAL |
| **Consistency** | После транзакции БД в корректном состоянии (constraints соблюдены) | Это свойство **приложения**, не СУБД. БД проверяет только declared constraints (FK, CHECK, UNIQUE). Бизнес-инварианты — ответственность кода |
| **Isolation** | Concurrent транзакции не видят промежуточных состояний друг друга | Зависит от **уровня изоляции**. Полная изоляция (Serializable) дорогая → большинство приложений работает на Read Committed |
| **Durability** | После COMMIT данные не теряются даже при crash | WAL + fsync. Trade-off: synchronous commit (безопасно, медленно) vs asynchronous commit (быстро, окно потерь) |

## Уровни изоляции по SQL-стандарту

Стандарт SQL определяет 4 уровня через **допустимые аномалии**:

| Уровень | Dirty Read | Non-Repeatable Read | Phantom |
|---------|-----------|-------------------|---------|
| **READ UNCOMMITTED** | Да | Да | Да |
| **READ COMMITTED** | Нет | Да | Да |
| **REPEATABLE READ** | Нет | Нет | Да |
| **SERIALIZABLE** | Нет | Нет | Нет |

### Аномалии подробно

| Аномалия | Что происходит | Пример |
|----------|----------------|--------|
| **Dirty read** | T1 читает данные, которые T2 ещё не закоммитила. Если T2 откатится — T1 прочитала «мусор» | T2: UPDATE balance = 0. T1: SELECT balance → видит 0. T2: ROLLBACK → balance снова 100, но T1 уже приняла решение на основе 0 |
| **Non-repeatable read** | T1 читает строку дважды — между чтениями T2 её изменила и закоммитила | T1: SELECT balance → 100. T2: UPDATE balance = 50; COMMIT. T1: SELECT balance → 50 (другое значение!) |
| **Phantom** | T1 выполняет SELECT по условию, T2 вставляет строку, подходящую под это условие | T1: SELECT COUNT(*) WHERE dept='sales' → 5. T2: INSERT INTO emp (dept='sales'); COMMIT. T1: SELECT COUNT(*) → 6 |

### Чего стандарт НЕ описывает

Стандарт не упоминает **write skew** и **lost update** — аномалии, которые выявлены позже (Berenson et al., 1995). DDIA (Kleppmann) подробно их разбирает. На практике:

- **Lost update** — два concurrent read-modify-write. Формально не phantom и не non-repeatable read, но данные теряются. Некоторые СУБД детектируют на Repeatable Read (PostgreSQL), другие — нет (MySQL).
- **Write skew** — только Serializable защищает.

## Блокировки (locking model)

### Типы блокировок

| Блокировка | Обозначение | Назначение |
|-----------|-------------|------------|
| **Shared (S)** | Read lock | Несколько транзакций могут держать S-lock одновременно. Блокирует X-lock |
| **Exclusive (X)** | Write lock | Только одна транзакция. Блокирует и S, и X |
| **Intent Shared (IS)** | Table-level hint | «Кто-то внутри будет брать S-lock на строки» — позволяет не проверять каждую строку |
| **Intent Exclusive (IX)** | Table-level hint | «Кто-то внутри будет брать X-lock на строки» |
| **Shared Intent Exclusive (SIX)** | Комбо | S на таблицу + IX (буду читать всё, но некоторые строки обновлю) |

### Матрица совместимости

|  | S | X | IS | IX | SIX |
|--|---|---|----|----|-----|
| **S** | Да | Нет | Да | Нет | Нет |
| **X** | Нет | Нет | Нет | Нет | Нет |
| **IS** | Да | Нет | Да | Да | Да |
| **IX** | Нет | Нет | Да | Да | Нет |
| **SIX** | Нет | Нет | Да | Нет | Нет |

### Гранулярность блокировок

| Уровень | Concurrency | Lock overhead |
|---------|------------|---------------|
| **Row-level** | Максимальная. Разные транзакции работают с разными строками параллельно | Много lock'ов → больше памяти и CPU |
| **Page-level** | Средняя. Блокируется страница (обычно 8 KB, несколько строк) | Компромисс |
| **Table-level** | Минимальная. Вся таблица заблокирована | Минимальный overhead, но сериализует всех |

**Lock escalation:** если транзакция набрала слишком много row-lock'ов (порог зависит от СУБД), СУБД автоматически повышает до page- или table-lock. Экономит память, но убивает concurrency. SQL Server делает это активно, PostgreSQL — нет (использует lightweight locks иначе).

## Two-Phase Locking (2PL)

Классический протокол обеспечения serializable isolation через блокировки:

| Фаза | Что происходит |
|------|----------------|
| **Growing (захват)** | Транзакция берёт новые lock'и по мере необходимости. Ни один lock не отпускается |
| **Shrinking (освобождение)** | После первого release транзакция может только отпускать lock'и, новые не берёт |

**Strict 2PL:** все lock'и держатся **до COMMIT/ROLLBACK**. Это то, что реально используют СУБД — предотвращает cascading aborts.

### 2PL vs MVCC

| | 2PL | MVCC |
|--|-----|------|
| **Readers блокируют writers** | Да (S-lock мешает X-lock) | Нет (читают старую версию) |
| **Writers блокируют readers** | Да (X-lock мешает S-lock) | Нет |
| **Deadlocks** | Частые при высокой concurrency | Редкие (только write-write конфликты) |
| **Overhead** | Lock table, detection | Хранение версий, VACUUM / purge |
| **Кто использует** | SQL Server (default), MySQL с `LOCK IN SHARE MODE` | PostgreSQL, MySQL InnoDB (default), Oracle |

## Deadlocks

### Условия возникновения (все четыре одновременно)

1. **Mutual exclusion** — ресурс в exclusive mode.
2. **Hold and wait** — транзакция держит lock и ждёт другой.
3. **No preemption** — lock нельзя отобрать силой.
4. **Circular wait** — T1 ждёт T2, T2 ждёт T1 (цикл).

### Как СУБД обрабатывают deadlock

| Стратегия | Как работает |
|-----------|-------------|
| **Detection (wait-for graph)** | СУБД строит граф ожиданий. Если находит цикл — одна транзакция (victim) откатывается. PostgreSQL, InnoDB |
| **Timeout** | Если транзакция ждёт lock дольше N секунд — abort. Грубо, но просто. Часто как дополнение к detection |
| **Prevention (lock ordering)** | Приложение берёт lock'и всегда в одном порядке → цикл невозможен. Ответственность разработчика |

### Выбор victim

СУБД выбирает для отката транзакцию с **наименьшей стоимостью**: меньше работы сделала, меньше lock'ов держит, меньше данных изменила. Конкретные критерии зависят от СУБД.

## Как СУБД реализуют уровни изоляции

| СУБД | Read Committed | Repeatable Read | Serializable |
|------|---------------|-----------------|--------------|
| **PostgreSQL** | MVCC: snapshot на statement | MVCC: snapshot на транзакцию | SSI (optimistic, detect rw-deps) |
| **MySQL InnoDB** | MVCC: snapshot на statement | MVCC: snapshot на транзакцию. Locking reads для write-защиты | Все SELECT → `LOCK IN SHARE MODE` (фактически 2PL) |
| **SQL Server** | Lock-based (S-locks на read, отпускаются сразу) или RCSI (MVCC через tempdb) | Lock-based: S-locks держатся до конца транзакции. Или Snapshot Isolation (MVCC) — отдельный уровень | 2PL + range locks (key-range locking) |
| **Oracle** | MVCC (undo tablespace) | Нет отдельного уровня; используй SELECT FOR UPDATE | MVCC + serialization checks |

## Типичные вопросы на интервью

**Q: Чем отличается lock-based изоляция от MVCC?**
A: Lock-based (2PL): readers берут shared lock → блокируют writers. Обеспечивает serializable, но снижает concurrency и провоцирует deadlocks. MVCC: readers читают старую версию → не блокируют writers и наоборот. Выше concurrency, но нужна очистка старых версий (VACUUM в PG, purge в InnoDB). Большинство современных СУБД по умолчанию используют MVCC.

**Q: Какие блокировки существуют в SQL и когда они применяются?**
A: **Shared (S)** — при чтении (если lock-based mode), несколько транзакций одновременно. **Exclusive (X)** — при записи, одна транзакция. **Intent locks (IS, IX)** — на уровне таблицы, сигнализируют о намерении блокировать строки внутри. Гранулярность: row → page → table. Lock escalation повышает гранулярность при слишком большом количестве мелких lock'ов.

**Q: Что такое deadlock и как его предотвратить?**
A: Циклическое ожидание: T1 держит lock A, ждёт B; T2 держит B, ждёт A. СУБД детектирует через wait-for graph и откатывает одну транзакцию (victim). Предотвращение в коде: 1) брать lock'и всегда в одном порядке, 2) минимизировать время транзакции, 3) использовать SELECT FOR UPDATE с NOWAIT или таймаутом.

**Q: Почему стандарт SQL плохо описывает аномалии?**
A: Стандарт определяет только 3 аномалии (dirty read, non-repeatable read, phantom). Он не описывает **lost update** и **write skew**, которые возможны на Repeatable Read в MVCC-реализациях. Berenson et al. (1995) показали, что стандартная классификация неполна. PostgreSQL на Serializable (SSI) ловит write skew, MySQL на Repeatable Read — нет.

**Q: В чём разница между `SELECT FOR UPDATE` и `SELECT FOR SHARE`?**
A: `FOR UPDATE` — exclusive lock на выбранные строки. Другие транзакции не могут ни читать с lock, ни писать. Используется для read-modify-write (чтобы предотвратить lost update). `FOR SHARE` — shared lock: другие могут тоже FOR SHARE, но не FOR UPDATE и не UPDATE/DELETE. Используется для проверки существования (FK check, «строка не удалится пока я работаю»).

**Q: Когда нужен explicit locking (FOR UPDATE), а когда хватает уровня изоляции?**
A: На Read Committed (default в PG и MySQL) **всегда** нужен FOR UPDATE для read-modify-write, иначе lost update. На Repeatable Read PostgreSQL сам обнаружит конфликт и откатит транзакцию. На Serializable — полная защита, explicit lock не нужен, но нужна retry-логика. Trade-off: explicit lock проще понять, Serializable + retry надёжнее, но дороже.

## Связи

- [Transactions and Isolation](transactions-and-isolation.md) — теория из DDIA (distributed perspective, MVCC, 2PC).
- [PostgreSQL MVCC Internals](postgresql-mvcc-internals.md) — как PostgreSQL реализует MVCC под капотом.
- [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md) — сравнение реализаций.
- [PostgreSQL](../entities/postgresql.md) — entity-страница.
