# Redis Distributed Locks

**Источник:** [Redis Official Documentation](../sources/redis-official-documentation.md) (distributed locks, reliability patterns)

## Суть

Redis-lock работает, если ты чётко понимаешь границы гарантий. На single-instance Redis можно сделать эффективный lock для координации коротких операций. В распределённом сценарии (несколько узлов, partition, clock drift, GC pauses) возникают риски split-brain, и lock без fencing tokens может нарушить инварианты.

Senior-подход: выбирать механизм по уровню критичности. Для cache rebuild и idempotent jobs Redis-lock обычно достаточен. Для строгих инвариантов (денежные операции, external side effects) нужен coordination-сервис с более сильной моделью (ZooKeeper/etcd/consensus-based).

## Single-instance lock pattern

| Шаг | Команда/идея | Почему важно |
|-----|---------------|--------------|
| Acquire | `SET lock_key token NX PX ttl` | атомарное «взять если свободно» с TTL |
| Ownership | случайный `token` на клиента | чтобы не удалить чужой lock |
| Release | Lua compare-and-delete | `DEL` без проверки опасен |

Lua release pattern:
- если `GET lock_key == token` -> `DEL`
- иначе ничего не делать.

## Fencing tokens (критически важно)

Даже с TTL lock может «протухнуть», а старый владелец продолжит работу (например, из-за stop-the-world pause). Поэтому downstream система должна уметь отбрасывать старые операции по монотонному fencing token.

Без fencing token lock превращается в best-effort mutex, а не строгую гарантию корректности.

## Redlock: идея и ограничения

| Аспект | Redlock |
|--------|---------|
| Топология | N независимых Redis masters |
| Условие lock | успешно взят majority и уложились во временное окно |
| Плюс | снижает риск single-node failure |
| Риск | зависимость от времени, сети и пауз процесса |

Критика (Kleppmann): при неблагоприятных паузах/partition можно получить конкурентные владельцы. Позиция Antirez: для ряда практических сценариев с корректными таймингами и risk profile подход приемлем.

## Когда Redis-lock уместен

- Координация cache warmup/single-flight.
- Cron/job deduplication, где операция идемпотентна.
- Short critical sections и допустимый риск редких конфликтов.

## Когда лучше ZooKeeper/etcd

- Критичные инварианты и внешние side effects без rollback.
- Нужна строгая последовательность владения и lease semantics.
- Требуется долговременная координация между сервисами с высокой стоимостью ошибки.

## Типичные вопросы на интервью

**Q: Почему `SET NX EX` недостаточно без token?**  
A: Потому что lock может истечь, новый клиент получит его, а старый клиент всё ещё выполнит `DEL` и снесёт чужой lock. Token + compare-and-delete через Lua предотвращает эту ошибку ownership.

**Q: Что такое fencing token и зачем он нужен, если уже есть TTL?**  
A: TTL ограничивает время жизни lock, но не предотвращает late execution после паузы процесса. Fencing token (монотонный номер) позволяет downstream системе принять только самый новый владелец и отвергнуть stale операции.

**Q: Когда Redlock можно использовать безопасно?**  
A: Когда риск-модель допускает best-effort distributed lock, операции идемпотентны, есть conservative TTL/timeout, мониторинг и fallback. Для «деньги/дважды списать нельзя» Redlock обычно недостаточен без дополнительных защит (fencing + durable coordination).

**Q: Какой частый anti-pattern с Redis-lock в backend?**  
A: Использовать lock как единственный guard для non-idempotent side effects и считать, что это «строгий mutex как в локальном процессе». В распределённой системе это неверно без fencing и проверок в месте применения.

## Связи

- [Redis Atomicity and Performance Patterns](redis-atomicity-and-performance-patterns.md)
- [Redis Rate Limiting Patterns](redis-rate-limiting-patterns.md)
- [Consistency and Consensus](consistency-and-consensus.md)
- [Redis](../entities/redis.md)
