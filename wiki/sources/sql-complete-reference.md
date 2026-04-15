# SQL: The Complete Reference

| Поле | Значение |
|------|----------|
| **Авторы** | James R. Groff, Paul N. Weinberg, Andrew J. Oppel |
| **Издательство** | McGraw-Hill |
| **Издание** | 3-е (2009) |
| **Тип** | книга |

## О книге

Полное руководство по SQL: от основ языка до транзакций, безопасности, оптимизации и распределённых баз данных. Ценность для вики — **детальное описание транзакционной модели SQL**: как стандарт определяет уровни изоляции, какие блокировки за ними стоят, как разные СУБД реализуют эти гарантии на практике.

## Ключевые части (в контексте вики)

### Part IV. SQL and Transaction Processing

Главы по транзакциям — основной фокус:

- **Транзакционная модель SQL:** COMMIT, ROLLBACK, SAVEPOINT, autocommit, границы транзакций.
- **ACID** с акцентом на практику: что именно гарантирует каждая буква в реальных СУБД.
- **Уровни изоляции по стандарту:** READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE. Какие аномалии каждый уровень допускает (dirty read, non-repeatable read, phantom).
- **Блокировки:** shared (S) и exclusive (X) locks, lock granularity (row, page, table), lock escalation, intent locks. Матрица совместимости.
- **Deadlocks:** условия возникновения, detection (wait-for graph), prevention (timeout, ordering), разрешение (victim selection).
- **Two-phase locking (2PL):** growing phase + shrinking phase. Strict 2PL — locks до commit.

### Другие части

- Part I–III: SQL DML/DDL, JOIN'ы, подзапросы, агрегация — хорошая база, но в вики уже покрыта.
- Part V: Security, privileges, views.
- Part VI: SQL и распределённые данные, программный доступ (embedded SQL, ODBC, JDBC).

## Ключевые темы вики

| Тема | Страница |
|------|----------|
| Транзакции, блокировки, изоляция (SQL-стандарт + практика) | [SQL Transactions, Locking and Isolation](../concepts/sql-transactions-locking-standard.md) |
| Транзакции и изоляция (DDIA — distributed perspective) | [Transactions and Isolation](../concepts/transactions-and-isolation.md) |
| MVCC в PostgreSQL (реализация) | [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md) |
| MVCC: PostgreSQL vs MySQL | [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md) |

## Связи

- [PostgreSQL](../entities/postgresql.md)
- [Data Engineering Fundamentals](../overviews/data-engineering-fundamentals.md)
