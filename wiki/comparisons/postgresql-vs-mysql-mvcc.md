# PostgreSQL vs MySQL: MVCC

**Контекст:** обе СУБД реализуют MVCC для snapshot isolation, но **архитектурно противоположным способом**. PostgreSQL хранит версии в heap, MySQL InnoDB — в undo log. Это определяет trade-offs по bloat, UPDATE cost, VACUUM, индексам и read performance.

## Главная разница: где живут старые версии

| | PostgreSQL | MySQL InnoDB |
|--|-----------|--------------|
| **Хранение версий** | В heap-таблице рядом с актуальными | Актуальная версия in-place в clustered index; старые — в **undo log** (rollback segment) |
| **UPDATE** | DELETE старой + INSERT новой версии. Все индексы обновляются (кроме HOT) | In-place update в clustered index. Старое значение → undo log. Secondary indexes не трогаются (кроме изменённых столбцов) |
| **DELETE** | xmax помечается → физически tuple остаётся до VACUUM | Delete mark → purge thread удаляет асинхронно |
| **Очистка** | **VACUUM** (отдельный процесс, сканирует таблицу) | **Purge thread** (фоновый, чистит undo log). Tablespace не раздувается |

## Видимость и snapshots

| | PostgreSQL | MySQL InnoDB |
|--|-----------|--------------|
| **Snapshot** | Список активных xid (xmin, xmax, xip_list) | Read view: список активных transaction ID |
| **Проверка видимости** | По xmin/xmax + infomask в самом tuple | По DB_TRX_ID в записи → если нужна старая версия, идёт по undo-цепочке |
| **Read Committed** | Новый snapshot на **каждый statement** | Новый snapshot на **каждый statement** (аналогично) |
| **Repeatable Read** | Snapshot на начало транзакции. Настоящий snapshot isolation | Snapshot на начало транзакции. **Но**: MySQL не детектирует write skew (в отличие от PG Serializable). MySQL RR + locking reads (FOR UPDATE) — основной способ избежать аномалий |
| **Serializable** | SSI (Serializable Snapshot Isolation) — optimistic, detect & abort | **Не snapshot-based**: все SELECT становятся `SELECT ... LOCK IN SHARE MODE`. Фактически 2PL, не SSI |

## Индексы и clustered storage

| | PostgreSQL | MySQL InnoDB |
|--|-----------|--------------|
| **Clustered index** | Нет (heap — неупорядоченный). PK — обычный B-tree index → ctid в heap | **Обязательно**. Данные физически упорядочены по PK. Если PK не задан — InnoDB создаёт скрытый |
| **Secondary index** | Содержит ctid (физический адрес в heap) | Содержит значение PK → для получения строки нужен **double lookup** (secondary → PK → row) |
| **Index-Only Scan** | Возможен, но требует проверки visibility map (VACUUM должен пометить страницу all-visible) | **Covering index** работает без дополнительных проверок (undo log отделён от данных) |
| **UPDATE + индексы** | Все индексы обновляются (новый tuple = новый ctid) — кроме HOT | Только secondary indexes с изменёнными столбцами. Clustered index обновляется in-place |

## Bloat и обслуживание

| | PostgreSQL | MySQL InnoDB |
|--|-----------|--------------|
| **Table bloat** | Главная проблема: мёртвые tuple копятся → таблица растёт. VACUUM возвращает пространство для reuse, но **не ОС** (для этого VACUUM FULL / pg_repack) | Tablespace стабилен. Undo log — отдельное пространство, purge чистит автоматически |
| **VACUUM** | Обязателен. Autovacuum по умолчанию. Нагружает I/O, конкурирует с нагрузкой | Не нужен. Purge thread легковесный |
| **Transaction ID wraparound** | Реальная угроза: 32-битный xid → freeze обязателен. Без freeze — emergency shutdown | Нет проблемы: InnoDB trx_id — 48 бит, undo log очищается по мере необходимости |
| **Мониторинг** | `pg_stat_user_tables.n_dead_tup`, `pg_stat_activity`, autovacuum_count | `SHOW ENGINE INNODB STATUS` — history list length, purge lag |

## Производительность: trade-offs

| Сценарий | PostgreSQL | MySQL InnoDB |
|----------|-----------|--------------|
| **Read-heavy, мало UPDATE** | Отлично. Snapshot isolation «бесплатно», index-only scan при хорошем VACUUM | Отлично. Covering indexes без visibility map overhead |
| **Write-heavy, частые UPDATE** | Дороже: полная копия строки + обновление всех индексов + bloat → нужен агрессивный VACUUM | Дешевле: in-place update, undo log компактный, purge автоматический |
| **Long-running транзакции** | Держат xmin snapshot → VACUUM не может убрать мёртвые tuple → bloat нарастает | Undo log не может быть purged → history list растёт, но tablespace стабилен |
| **HOT UPDATE** | Спасение для write-heavy: если не затронуты индексированные столбцы и хватает места на странице — нет обновления индексов | Не актуально (in-place update и так) |

## Типичные вопросы на интервью

**Q: В чём главное архитектурное различие MVCC между PostgreSQL и MySQL?**
A: PostgreSQL хранит **все версии строк в heap** (xmin/xmax), а мёртвые убирает VACUUM. MySQL InnoDB хранит **актуальную версию in-place** в clustered index, а предыдущие — в undo log. Следствие: PostgreSQL проще в реализации видимости (всё на месте), но страдает от bloat. InnoDB не bloat'ится, но undo log может расти при длинных транзакциях.

**Q: Почему UPDATE в PostgreSQL дороже?**
A: Создаётся **новый tuple** с новым ctid → каждый индекс на таблице должен получить новую запись (кроме HOT). В InnoDB — in-place update, secondary indexes обновляются только если затронуты их столбцы. На таблице с 10 индексами разница колоссальная.

**Q: Почему PostgreSQL нужен VACUUM, а MySQL — нет?**
A: Потому что PostgreSQL оставляет мёртвые версии в самой таблице → без очистки она растёт бесконечно. InnoDB складывает старые версии в undo log, который purge thread чистит автоматически, а основная таблица не раздувается.

**Q: Как отличается Serializable в PostgreSQL и MySQL?**
A: PostgreSQL — **SSI** (Serializable Snapshot Isolation): optimistic, позволяет параллельное выполнение, детектирует anomalies и абортит одну из транзакций. MySQL — фактически **2PL**: все SELECT'ы становятся locking reads, что сильнее ограничивает concurrency, но не требует retry.

**Q: Когда PostgreSQL лучше, когда MySQL?**
A: PostgreSQL: сложные запросы, расширяемость (PostGIS, JSONB, custom types), корректная Serializable isolation, продвинутый optimizer. MySQL InnoDB: write-heavy OLTP с частыми UPDATE, проще в обслуживании (нет VACUUM), предсказуемый размер таблиц, быстрее на простых point lookups по PK (clustered index).

## Связи

- [PostgreSQL MVCC Internals](../concepts/postgresql-mvcc-internals.md) — детальная механика PostgreSQL.
- [PostgreSQL](../entities/postgresql.md) — entity-страница.
- [Transactions and Isolation](../concepts/transactions-and-isolation.md) — теория уровней изоляции.
