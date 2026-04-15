# ClickHouse MergeTree Engines

**Источник:** [ClickHouse Official Documentation](../sources/clickhouse-official-documentation.md)

## Базовая идея семейства MergeTree

Все движки семейства MergeTree:

- хранят данные в parts;
- сортируют по `ORDER BY`;
- читают через sparse primary index;
- используют background merges.

Отличия — в том, **как интерпретировать строки при merge и чтении**.

## 1) MergeTree (базовый)

**Что делает:** просто хранит все строки как есть.  
**Когда использовать:** append-only raw/fact, без автоматического дедупа/суммирования.  
**Плюсы:** предсказуемое поведение, универсальность.

## 2) ReplacingMergeTree

**Что делает:** при merge оставляет одну “последнюю” версию строки по ключу сортировки (с optional version column).  
**Когда использовать:** upsert-паттерны, CDC latest-state.

Важно:

- дедуп не мгновенный, а по мере merge;
- для строгого результата “прямо сейчас” часто используют `SELECT ... FINAL`.

Per `insert-mutation-avoid-update`: хороший кандидат вместо частых `ALTER UPDATE`.

## 3) SummingMergeTree

**Что делает:** при merge суммирует числовые колонки для одинакового ключа.  
**Когда использовать:** аддитивные метрики (счетчики, суммы).

Ограничение:

- подходит только для корректно аддитивных величин;
- не подходит для сложных неаддитивных агрегатов (uniq, percentiles).

## 4) AggregatingMergeTree

**Что делает:** хранит состояния агрегатных функций (`AggregateFunction`), merge объединяет состояния.  
**Когда использовать:** `uniqState`, `quantileState`, сложные агрегаты из incremental MV.

Паттерн:

- в MV — `...State` функции;
- в запросе — `...Merge`.

Per `query-mv-incremental`: это стандартный подход для real-time pre-aggregation.

## 5) CollapsingMergeTree / VersionedCollapsingMergeTree

**Что делает:** “схлопывает” пары строк (например +1/-1) по ключу.  
**Когда использовать:** event-sourcing/soft-delete-сценарии, где изменения моделируются как поток событий.

Требует аккуратного моделирования знака/версии и корректных запросов.

## 6) CoalescingMergeTree

**Что делает:** сливает непустые значения из нескольких строк с одинаковым ключом (часто для sparse updates).  
**Когда использовать:** редкие кейсы частичных обновлений, где логика coalesce уместна.

## 7) Replicated*MergeTree

**Что делает:** добавляет репликацию и координацию между узлами для любого MergeTree-варианта (`ReplicatedMergeTree`, `ReplicatedReplacingMergeTree`, и т.д.).  
**Когда использовать:** отказоустойчивые прод-кластеры.

## 8) Как выбирать движок

| Нагрузка | Рекомендуемый движок |
|---------|-----------------------|
| Append-only raw/fact | `MergeTree` |
| Latest-state / upsert | `ReplacingMergeTree` |
| Суммы и счетчики | `SummingMergeTree` |
| Сложные агрегаты (uniq/quantile) | `AggregatingMergeTree` |
| Soft delete через события | `Collapsing*` |
| HA/репликация | `Replicated*` вариант выбранного движка |

## 9) Частые ошибки

- Ожидание “мгновенного” дедупа в ReplacingMergeTree без `FINAL`.
- Использование SummingMergeTree для неаддитивных KPI.
- Попытка лечить модель частыми мутациями вместо выбора правильного движка.

## Типичные вопросы на интервью

**Q: Когда ReplacingMergeTree, а когда обычный MergeTree?**
A: **MergeTree** — для append-only данных (логи, events, immutable facts). **ReplacingMergeTree** — когда нужен **latest state** по ключу (CDC upsert, dimension tables). Важно: дедуп не мгновенный (при merge), для точного результата — `SELECT ... FINAL`.

**Q: Чем SummingMergeTree отличается от AggregatingMergeTree?**
A: **Summing** — при merge суммирует числовые колонки по ключу. Прост, но только для **аддитивных** метрик (count, sum). **Aggregating** — хранит **состояния** произвольных агрегатных функций (`uniqState`, `quantileState`). Мощнее, но сложнее: запись через `...State()`, чтение через `...Merge()`.

**Q: Как моделировать delete в ClickHouse?**
A: Варианты: 1) **Soft delete** — флаг `is_deleted`, фильтрация в запросах. 2) **CollapsingMergeTree** — пара строк +1/-1, схлопывание при merge. 3) **DROP PARTITION** — массовое удаление по времени (быстро). 4) Lightweight `DELETE` — для умеренных случаев. Избегать: регулярные `ALTER TABLE DELETE` на больших таблицах.

**Q: Что такое FINAL в SELECT и зачем он нужен?**
A: `SELECT ... FINAL` заставляет ClickHouse применить логику движка (дедуп для Replacing, суммирование для Summing) **прямо в запросе**, даже если merge ещё не прошёл. Без FINAL — могут быть дубли/неконсолидированные строки. Цена: overhead на runtime-merge. Альтернатива: argMax/argMin паттерн.

**Q: Когда нужна Replicated* версия движка?**
A: Для production-кластера с **отказоустойчивостью**. `ReplicatedMergeTree` (и другие Replicated*) координируют данные между репликами через ClickHouse Keeper (или ZooKeeper). Каждая реплика хранит полную копию. Без Replicated — данные на одном узле, потеря узла = потеря данных.

## Связи

- [ClickHouse: индексы и background merges](clickhouse-indexes-and-background-merges.md)
- [ClickHouse Insert and Mutation Strategy](clickhouse-insert-and-mutation-strategy.md)
- [ClickHouse Query Optimization Patterns](clickhouse-query-optimization-patterns.md)
