# Distributed Systems Pitfalls

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 8)

## Суть

В распределённой среде нельзя полагаться на «идеальную» сеть или часы: **задержки непредсказуемы**, **часы неточны**, процессы **паузятся** (GC, virtualization). **Частичные сбои** — норма. Отсюда таймауты, fencing, версионирование, и осторожность с «разумными» предположениями о времени и порядке событий.

## Ключевые проблемы

| Проблема | Суть | Последствия |
|----------|------|-------------|
| **Unreliable network** | Пакеты теряются, дублируются, задерживаются непредсказуемо | Нельзя отличить «узел умер» от «сеть задержалась» — только таймаут |
| **Unreliable clocks** | Часы узлов расходятся (NTP даёт точность ~мс, не нс) | Нельзя полагаться на timestamp для ordering событий между узлами |
| **Process pauses** | GC, page fault, preemption → процесс «замирает» на секунды | Лидер может не знать, что его «отстранили»; lease/fencing token |
| **Partial failures** | Часть системы работает, часть — нет | Нельзя полагаться на «всё или ничего»; нужны: таймауты, retry, идемпотентность |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **Byzantine fault** | Узел ведёт себя произвольно (ложь, corruption). Обычные системы не защищают от Byzantine; предполагают «честные» узлы. |
| **Fencing token** | Монотонно растущий номер, выдаваемый при lease/lock. Ресурс отклоняет запросы со старым token → защита от «зомби»-лидера. |
| **Lease** | Временная блокировка с TTL; нужно регулярно продлевать. Если процесс завис (GC) и lease истёк — другой узел становится лидером. |
| **Idempotency** | Повторное применение операции даёт тот же результат. Критично при retry: сеть не подтвердила, клиент повторяет, но сервер уже выполнил. |
| **Happens-before** | Частичный порядок событий: A → B, если A причинно предшествует B. Для concurrent событий порядок **не определён** без дополнительных механизмов. |

## Типичные вопросы на интервью

**Q: Почему нельзя полагаться на часы для упорядочивания событий?**
A: Часы узлов расходятся (clock skew). NTP корректирует, но с задержкой и конечной точностью. Если два события на разных узлах имеют close timestamps — нельзя определить, кто первый. Решения: logical clocks (Lamport), vector clocks, или явный consensus (Raft/Paxos).

**Q: Как отличить сбой узла от сетевой задержки?**
A: Никак — с точки зрения вызывающего, оба случая выглядят одинаково (нет ответа). Единственный инструмент — **таймаут**. Слишком короткий → false positive (узел жив, но медленный). Слишком длинный → долгий failover. Практика: adaptive timeouts + heartbeats.

**Q: Что такое split brain и как его предотвратить?**
A: Два узла считают себя лидером (например после network partition). Оба принимают writes → конфликтующие данные. Предотвращение: **quorum** (большинство должно согласиться), **fencing tokens** (ресурс отвергает запросы со старым token), **epoch numbers**.

**Q: Почему idempotency важна в распределённых системах?**
A: При retry (таймаут, но сервер уже обработал) без idempotency операция выполнится дважды. В data engineering: дубли в DWH, двойное списание. Решения: уникальные request ID, dedup в staging, idempotent writes (UPSERT, DROP PARTITION + reload).

**Q: Какие fallacies of distributed computing ты знаешь?**
A: Network is reliable, latency is zero, bandwidth is infinite, network is secure, topology doesn't change, there is one administrator, transport cost is zero, network is homogeneous. Все ложны — нужно проектировать с учётом каждого.

## Связи

- [Consistency and Consensus](consistency-and-consensus.md) — как строить порядок и согласие при этих ограничениях.
- [Replication](replication.md) — задержки репликации и разрыв с реальностью.
- [Redis Replication, Sentinel and Cluster](redis-replication-sentinel-cluster.md) — практический пример failover и trade-off между availability и consistency.
