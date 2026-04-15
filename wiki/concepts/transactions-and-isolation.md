# Transactions and Isolation

**Источники:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 7), [SQL: The Complete Reference](../sources/sql-complete-reference.md) (Part IV)

## Суть

**Транзакции** группируют операции с гарантиями **ACID** (в разных СУБД интерпретации различаются). **Уровни изоляции** (read committed, snapshot isolation, serializable) защищают от гонок, lost update, phantom reads по разной цене. Распределённые транзакции (2PC и альтернативы) усложняют отказоустойчивость и latency.

## Уровни изоляции

| Уровень | Защищает от | Не защищает от | Реализация |
|---------|-------------|----------------|------------|
| **Read Uncommitted** | — | Dirty reads, всё остальное | Почти не используется |
| **Read Committed** | Dirty reads, dirty writes | Non-repeatable reads, phantom | Lock на запись; read видит committed версию |
| **Repeatable Read / Snapshot Isolation** | + non-repeatable reads | Write skew, phantoms (зависит от реализации) | MVCC — каждая транзакция видит snapshot на начало |
| **Serializable** | Всё: write skew, phantoms | — (полная изоляция) | 2PL (lock), SSI (optimistic), или actual serial |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **ACID** | Atomicity (всё или ничего), Consistency (инварианты), Isolation (транзакции не мешают), Durability (данные на диске). В разных СУБД интерпретации отличаются (особенно isolation). |
| **Dirty read** | Чтение данных, которые ещё не committed другой транзакцией. |
| **Lost update** | Два concurrent read-modify-write → один перезаписывает другого. Решение: atomic ops, explicit lock, CAS. |
| **Write skew** | Две транзакции читают одно и то же, принимают решение, пишут разное — результат нарушает инвариант. Snapshot isolation **не** защищает. |
| **Phantom** | Одна транзакция ищет строки по условию, другая вставляет строку, подходящую под условие — первая «не видит» новую строку. |
| **MVCC** | Multi-Version Concurrency Control — каждая транзакция видит свой snapshot; writers не блокируют readers. PostgreSQL, MySQL InnoDB. |
| **2PC (Two-Phase Commit)** | Координатор → prepare → все участники OK → commit. Блокирующий: если координатор падает после prepare — участники заблокированы. |

## Типичные вопросы на интервью

**Q: Объясни уровни изоляции простыми словами.**
A: Read committed: ты видишь только committed данные, но при повторном чтении можешь увидеть другое значение. Snapshot isolation: ты видишь consistent snapshot на момент начала транзакции (как «замороженный» вид). Serializable: транзакции выполняются «как будто последовательно» — максимальная корректность, минимальная производительность.

**Q: Что такое write skew и почему snapshot isolation не помогает?**
A: Пример: два врача одновременно проверяют «хотя бы один дежурный» → оба снимаются с дежурства → ноль дежурных. Snapshot isolation видит одинаковый snapshot, оба решения «корректны» по отдельности, но вместе нарушают инвариант. Защита: serializable isolation или explicit application-level lock.

**Q: Когда использовать 2PC и в чём его проблемы?**
A: 2PC — для координации транзакций между разными БД/сервисами. Проблемы: блокировка при сбое координатора (in-doubt participants держат locks), latency (два round-trip), single point of failure координатора. Альтернативы: Saga pattern, eventual consistency + компенсации.

**Q: Как PostgreSQL реализует snapshot isolation?**
A: Через **MVCC**: каждая строка имеет xmin/xmax (ID транзакции создания/удаления). Транзакция видит только строки с xmin < своего snapshot и xmax > своего snapshot (или нет xmax). Concurrent writes не блокируют reads.

**Q: Почему в data engineering транзакции менее критичны, чем в OLTP?**
A: В аналитических системах паттерн обычно **append-only** (insert, не update); изоляция обеспечивается на уровне batch/partition: «атомарная» загрузка батча, idempotent replay, DROP PARTITION + reload. ClickHouse вообще не поддерживает традиционные транзакции.

## Связи

- [SQL Transactions, Locking and Isolation](sql-transactions-locking-standard.md) — SQL-стандарт: блокировки (S/X/intent), 2PL, deadlocks, гранулярность.
- [PostgreSQL MVCC Internals](postgresql-mvcc-internals.md) — детальная реализация MVCC в PostgreSQL (xmin/xmax, snapshots, VACUUM).
- [PostgreSQL vs MySQL: MVCC](../comparisons/postgresql-vs-mysql-mvcc.md) — сравнение MVCC-реализаций.
- [Partitioning](partitioning.md) — координация между партициями.
- [Consistency and Consensus](consistency-and-consensus.md) — строгая сериализуемость и консенсус.
- [PostgreSQL](../entities/postgresql.md) — MVCC и изоляция на практике.
