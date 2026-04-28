# Redis Data Structures and Modeling

**Источник:** [Redis in Action](../sources/redis-in-action.md) (гл. 1-2)

## Суть

Главная идея Redis-моделирования: сначала выбирается **операция**, которую нужно делать быстро, и под неё подбирается структура данных (`String`, `Hash`, `Set`, `ZSET`, `List`). Это противоположно реляционному подходу «нормализуй таблицы и query planner всё соберёт». В Redis нет joins и вторичных индексов «из коробки» как в SQL — их строят руками через дополнительные ключи и соглашения именования.

## Ключевые структуры данных

| Структура | Что хранит | Типичный use-case |
|-----------|------------|-------------------|
| **String** | bytes/число/JSON blob | cache item, counters (`INCR`), token/session |
| **Hash** | map field -> value | профиль пользователя, состояние заказа |
| **List** | упорядоченная очередь | task queue, recent events |
| **Set** | уникальные элементы | теги, membership, dedup |
| **Sorted Set (ZSET)** | элементы с score | leaderboard, ranking, time-indexed feed |
| **Bitmap / HyperLogLog** | битовые и приближённые структуры | флаги активности, unique visitors estimate |

## Паттерны key design

| Паттерн | Пример | Зачем |
|---------|--------|-------|
| **Namespace prefix** | `user:123:profile` | избегаем коллизий, удобно удалять по группе |
| **Versioned key** | `product:v2:42` | безопасные rolling changes без мгновенной миграции |
| **Compound index key** | `post:tag:java` (`SET` of IDs) | ручной secondary index |
| **Time-bucket key** | `stats:login:2026-04-28T10` | агрегации и retention по времени |

## Manual secondary indexes

Поскольку Redis не делает SQL-подобные индексы, их создают отдельными ключами:

- `HASH user:42` — canonical data.
- `SET email:andrey@example.com -> {42}` — lookup by email.
- `ZSET user:last_seen` — сортировка пользователей по активности.

Trade-off: быстрые reads, но writes дорожают (нужно обновлять и primary key, и все индексы).

## Типичные вопросы на интервью

**Q: Почему в Redis нельзя «думать как в SQL»?**  
A: В SQL ты хранишь нормализованные таблицы, а DB engine сам строит plan, делает joins и использует индексы. В Redis модель обратная: ты заранее проектируешь ключи и структуры под конкретные read/write паттерны. Если не спроектировал lookup/index заранее, позже запрос будет дорогим или невозможным без full scan.

**Q: Когда выбирать Hash, а когда String с JSON?**  
A: `Hash` — когда нужно часто обновлять отдельные поля (`HSET user:42 email ...`) и читать частично (`HGET`). `String` + JSON — когда объект читается/пишется целиком, а схема часто меняется. Trade-off: Hash эффективнее для field-level updates, JSON проще сериализовать/десериализовать в приложении.

**Q: Почему ZSET так часто используют в interview-задачах?**  
A: Потому что он решает сразу две вещи: 1) хранит уникальный member, 2) поддерживает ordering по score и range queries (`ZRANGE`, `ZREVRANGE`, `ZRANGEBYSCORE`). Это идеальный инструмент для leaderboard, top-N, sliding-window rate limits и time-ordered feeds.

**Q: Как моделировать «поиск по полю», если нет secondary index?**  
A: Делаешь отдельный индексный ключ: например, `SET email:<value> -> user_id` или `ZSET users:by_created_at` для range. При update поля нужно транзакционно обновить и основной ключ, и индексные ключи (MULTI/EXEC или Lua), иначе появятся dangling indexes.

**Q: Какие анти-паттерны в key design?**  
A: 1) Случайные несогласованные имена ключей (сложно управлять lifecycle). 2) Гигантские значения без структуры. 3) Отсутствие TTL для кэша. 4) Отсутствие лимитов на cardinality в Set/ZSet (memory blow-up). 5) Использование `KEYS *` в production вместо `SCAN`.

## Связи

- [Redis](../entities/redis.md)
- [Redis Caching Patterns and Consistency](redis-caching-patterns-and-consistency.md)
- [Data Models and Query Languages](data-models-and-query-languages.md)
- [Partitioning](partitioning.md)
