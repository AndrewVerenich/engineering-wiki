# Storage and Retrieval

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 3)

## Суть

Данные на диске организуются через **структуры хранения** и **индексы**. Типичный контраст: **LSM-tree** (порционные слияния, хорош для write-heavy) vs **B-tree** (часто в БД, сбалансированное чтение/запись). Индексы ускоряют чтение ценой места и стоимости записи. Понимание движка помогает объяснять latency и план запросов.

## Ключевые структуры хранения

| Структура | Принцип | Write | Read | Примеры |
|-----------|---------|-------|------|---------|
| **B-tree** | Сбалансированное дерево; обновления in-place | Средняя (random I/O) | Быстрая точечная и range | PostgreSQL, MySQL InnoDB |
| **LSM-tree** | Append в memtable → flush в sorted SSTable → compaction | Быстрая (sequential I/O) | Медленнее точечная (проверка нескольких уровней) | RocksDB, Cassandra, LevelDB |
| **Columnar** | Данные по колонкам; сжатие, vectorized scans | Batch append | Очень быстрая для агрегаций/сканов | ClickHouse, Parquet, BigQuery |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **WAL (Write-Ahead Log)** | Журнал записи перед применением — для crash recovery. Используется и в B-tree, и в репликации. |
| **SSTable (Sorted String Table)** | Immutable файл с отсортированными записями; основа LSM-tree. |
| **Compaction** | Фоновый процесс слияния SSTables: удаление дублей, уменьшение числа файлов. Аналогия с merge в ClickHouse. |
| **Memtable** | In-memory буфер (обычно red-black tree), куда пишутся данные перед flush на диск. |
| **Write amplification** | Одна логическая запись → несколько физических (compaction, WAL). Проблема для SSD. |
| **Read amplification** | Для одного чтения нужно проверить несколько уровней/файлов (LSM). |

## Типичные вопросы на интервью

**Q: В чём ключевая разница между B-tree и LSM-tree?**
A: B-tree обновляет страницы in-place (random I/O, быстрее для точечных чтений). LSM-tree всегда append (sequential I/O, быстрее для записи), но чтение может проходить через несколько уровней. Trade-off: write throughput vs read latency.

**Q: Почему ClickHouse так быстр для аналитики?**
A: Колоночное хранение (читаем только нужные колонки), сжатие однородных данных в колонке, vectorized execution по блокам, sparse index по granules вместо строковых. Плюс MergeTree append-модель с background merge.

**Q: Что такое write amplification и почему это важно?**
A: При compaction одна логическая запись может быть переписана многократно на диск. Это влияет на износ SSD и I/O bandwidth. LSM-tree больше подвержен write amplification; B-tree — меньше, но имеет random I/O.

**Q: Зачем нужен WAL, если данные и так пишутся?**
A: Для crash recovery: если процесс упал между записью в memory и flush на диск, WAL позволяет восстановить незаконченные операции. Без WAL — потеря данных при сбое.

**Q: Когда бы ты выбрал LSM-tree vs B-tree?**
A: LSM-tree — для write-heavy нагрузок (логи, time-series, ingestion). B-tree — для mixed read/write с требованием к предсказуемой latency чтения (OLTP). В data engineering: источники (PostgreSQL, B-tree) → аналитика (ClickHouse, columnar/LSM-подобная модель).

## Связи

- [SQL Query Execution and Indexes](sql-query-execution-and-indexes.md) — план запроса, EXPLAIN, простые и составные индексы на практике.
- [Partitioning](partitioning.md) — индексы и партиции вместе задают производительность.
- [Encoding and Schema Evolution](encoding-and-schema-evolution.md) — сериализация значений.
- [PostgreSQL](../entities/postgresql.md) — B-tree и другие индексы в практике.
