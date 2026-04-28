# Apache Flink Architecture

**Источник:** [Stream Processing with Apache Flink](../sources/stream-processing-with-apache-flink.md) (гл. 3, 9)

## Суть

Apache Flink — **распределённый dataflow engine**: программа компилируется в **DAG операторов**, который физически разворачивается в параллельные subtasks по нескольким машинам. Координацию делает **JobManager** (один на job), вычисления — **TaskManager**'ы (несколько). Единица параллелизма — **task slot** на TaskManager. Между операторами данные передаются через сетевые буферы; Flink делает back-pressure и periodic **checkpoints** для fault tolerance.

## Логическая модель: dataflow-граф

| Понятие | Смысл |
|---------|-------|
| **StreamGraph** | Изначальный граф из пользовательского кода (после `env.execute()`). Узлы — операторы, рёбра — потоки данных. |
| **JobGraph** | Оптимизированный StreamGraph: цепочки операторов сливаются в **operator chain** (chaining) для уменьшения сериализации между ними. |
| **ExecutionGraph** | Параллельная развёртка JobGraph: каждый оператор → N параллельных **subtask**'ов. Это то, что реально выполняется в кластере. |
| **Operator chain** | Последовательность operator'ов с одинаковым parallelism и forward-распределением, выполняющаяся в одном потоке без сериализации между шагами. По умолчанию `map → filter → map` склеиваются. Разрывается через `disableChaining()`. |

## Физические компоненты

### JobManager (master)

Единый координатор job'а. Состоит из трёх логических ролей:

| Компонент | Что делает |
|-----------|-----------|
| **Dispatcher** | Принимает submit job, поднимает JobMaster для каждого job'а, предоставляет REST API и Web UI. |
| **JobMaster** | На каждый job — свой. Превращает JobGraph в ExecutionGraph, запрашивает slot'ы у ResourceManager, рассылает таски TaskManager'ам, координирует checkpoint'ы. |
| **ResourceManager** | Учёт slot'ов в кластере. На разных deployments — разные реализации: Standalone, YARN, Kubernetes (Native), Mesos (deprecated). |

**HA для JobManager:** через ZooKeeper или Kubernetes leader election. Метаданные job'а (graph, jar, completed checkpoints) хранятся в надёжном хранилище (HDFS, S3) — standby JobManager при failover поднимается из них.

### TaskManager (worker)

JVM-процесс, который реально исполняет subtask'и. Ключевые ресурсы:

| Ресурс | Что хранит |
|--------|-----------|
| **Task slots** | Слоты для запуска subtask'ов. Количество — `taskmanager.numberOfTaskSlots`. Slot изолирует **managed memory** между задачами. |
| **Network buffers** | Пул прямых байтовых буферов (off-heap) для обмена данными между subtask'ами по сети. Размер пула ограничивает throughput и устойчивость к back-pressure. |
| **Managed memory** | Off-heap память под state backend (RocksDB block cache, in-memory state), сортировки, hash join'ы. Flink сам ей управляет через slab allocator. |
| **JVM heap** | Пользовательский код, объекты, HashMap state backend, метаданные. |

### Task slot vs CPU vs parallelism

Это самая частая путаница:

| | |
|--|--|
| **Slot** | Логическая единица **резервирования памяти**. Не привязан к CPU core. |
| **CPU core** | Физический ресурс. Slot'ы делят его. Если slot'ов больше core'ов — context switching. |
| **Parallelism** | Количество параллельных subtask'ов одного оператора. Совпадает с числом slot'ов, занятых job'ом (при slot sharing — см. ниже). |

**Slot sharing (включён по умолчанию):** в одном slot могут жить **по одной копии каждого оператора** из всего pipeline. Если pipeline `source → map → window → sink` с parallelism=4, нужно 4 slot'а, не 16. Это позволяет: (1) экономить память, (2) сократить сетевой трафик (forward chain в одном slot — без сериализации), (3) уменьшить blast radius при backpressure.

## Обмен данными между операторами

| Тип | Когда |
|-----|------|
| **Forward** | Между chained операторами в одном slot. Передача через method call, без сериализации. Самый дешёвый. |
| **Hash partition** | После `keyBy` — события с одинаковым ключом идут в один subtask следующего оператора. `hash(key) % parallelism`. |
| **Rebalance** | Round-robin между downstream subtask'ами. Используется после source с неравномерной нагрузкой (skew). |
| **Rescale** | Локальный rebalance — каждый upstream subtask раздаёт только своим downstream'ам. Дешевле сетево, но требует совместимых parallelism. |
| **Broadcast** | Каждое событие копируется во все downstream subtask'и. Дорого; используется для broadcast state (рассылка config/правил). |
| **Global** | Всё идёт в один subtask (parallelism=1). Применение редкое — финальные агрегаты. |

### Сетевые буферы и back-pressure

Каждое отношение `producer subtask → consumer subtask` имеет **выходные** и **входные буферы** фиксированного размера. Если consumer медленнее — его входные буферы заполняются → producer не может писать → его исходящие буферы заполняются → upstream subtask замедляется. Так back-pressure естественно распространяется по pipeline без потери данных. Метрики `inPoolUsage` / `outPoolUsage` в Flink UI показывают, где затор.

## Deployment modes

| Режим | Как устроен | Применение |
|-------|------------|------------|
| **Session** | Долгоживущий кластер (JobManager + TaskManager'ы) принимает несколько job'ов. Job'ы делят ресурсы и могут падать друг от друга | Dev / shared cluster. Не для production critical jobs. |
| **Per-job** *(deprecated в 1.15+)* | Отдельный кластер на каждый job. Job submit → создание JobManager + TaskManager'ов из jar. После завершения — кластер удаляется | Старый production-режим. Заменён Application mode. |
| **Application mode** | `main()` пользовательского кода исполняется **на JobManager**, а не на клиенте. Один кластер = одно приложение (может содержать несколько job'ов) | Современный production-стандарт. Дешевле сетево (jar не гоняется), быстрее старт. |

### Resource managers

| | Что даёт |
|--|---------|
| **Standalone** | Ручное поднятие TaskManager'ов. Простой, для dev/тестов. |
| **YARN** | Заявки на YARN containers. Flink сам поднимает/убивает TaskManager'ы. |
| **Kubernetes (Native)** | Flink сам общается с K8s API, создаёт pods для TaskManager'ов. Нет нужды в отдельном operator (но есть Flink Kubernetes Operator для high-level управления). |

## Жизненный цикл job'а

1. Клиент компилирует код → `StreamGraph` → `JobGraph`. В Application mode это происходит на JobManager.
2. JobMaster получает `JobGraph`, строит `ExecutionGraph`, запрашивает slot'ы у ResourceManager.
3. TaskManager'ы регистрируются → ResourceManager выдаёт offers → JobMaster деплоит subtask'и в slot'ы.
4. Source subtask'и читают первые события, pipeline стартует.
5. Каждые `checkpoint.interval` ms JobMaster инициирует checkpoint.
6. При сбое subtask'а → JobMaster определяет failover region → перезапускает таски и восстанавливает state из последнего checkpoint'а.
7. Завершение: пользовательский cancel/stop → savepoint (опционально) → завершение TaskManager'ов.

## Failover strategies

| Стратегия | Что происходит при падении одной subtask |
|-----------|------------------------------------------|
| **full** *(по умолчанию для batch)* | Перезапускается весь job. Простая и надёжная, но дорогая. |
| **region** *(default streaming)* | Перезапускается только **failover region** — связанная компонента графа, в которой произошёл сбой. Если pipeline разделён через `keyBy` shuffle на регионы — соседние не перезапускаются. |

## Типичные вопросы на интервью

**Q: Как устроен Flink на верхнем уровне — какие компоненты есть и за что отвечают?**
A: Один **JobManager** (Dispatcher + JobMaster + ResourceManager) — координатор: принимает job, строит ExecutionGraph, выдаёт таски, инициирует checkpoint'ы. Несколько **TaskManager**'ов — workers. Каждый TaskManager имеет фиксированное число **task slots** — это единица резервирования памяти. JobManager поддерживает HA через ZK/K8s leader election; метаданные job'а хранятся в HDFS/S3.

**Q: Чем slot отличается от CPU core?**
A: Slot — это логическое резервирование managed memory под subtask'и; он не привязан к ядру. На одном TaskManager'е можно настроить, например, 4 slot'а на 8-ядерной машине, чтобы остался запас CPU под GC и фоновую работу. Конкретное соотношение подбирается по нагрузке: CPU-bound job'у — slot'ов меньше core'ов; I/O-bound — можно больше.

**Q: Что такое slot sharing и зачем он включён по умолчанию?**
A: В одном slot могут крутиться **по одному параллельному инстансу каждого оператора** pipeline'а. Это снижает количество требуемых slot'ов с `operators × parallelism` до `parallelism` и позволяет forward-связям между chained операторами работать без сериализации. Также помогает при skew — медленные операторы делят slot с быстрыми, а не блокируют отдельный.

**Q: Что такое operator chaining и когда его отключают?**
A: Chaining — это слияние нескольких операторов с одинаковым parallelism и forward-распределением в один **runtime task** (одна нить, нет сериализации между шагами). Отключают, когда: (1) хочешь точные метрики по каждому оператору отдельно, (2) у одного из операторов тяжёлая работа (внешние вызовы) и нужно изолировать его в свой поток, (3) для отладки. API: `disableChaining()`, `startNewChain()`, `env.disableOperatorChaining()`.

**Q: Как работает back-pressure в Flink?**
A: Flink не имеет отдельного rate-limit'а — back-pressure возникает естественно через **сетевые буферы**: если consumer медленнее, его input pool заполняется, producer не может писать в output pool, его пул заполняется, и upstream operator замедляется в `collect()`. Это распространяется до source'а, который перестаёт читать из Kafka. Видно в UI как `inPoolUsage`/`outPoolUsage` ≈ 1.0 — это и есть «затор».

**Q: Что произойдёт при падении одного TaskManager'а?**
A: JobMaster обнаружит потерю heartbeat'ов, помечает все subtask'и на этом TaskManager'е как failed. Срабатывает failover strategy `region`: перезапускаются только subtask'и затронутых failover-регионов. Они стартуют на новых slot'ах (если есть свободные или ResourceManager поднимет новый TaskManager) и восстанавливают state из последнего успешного checkpoint'а. Источники (Kafka) откатываются на сохранённые offset'ы.

**Q: В чём разница между Session и Application mode?**
A: **Session** — один долгоживущий кластер на много job'ов; быстрее submit, но job'ы конкурируют за ресурсы и падение одного может задеть других через JobManager. **Application mode** — один кластер = одно приложение; `main()` запускается на JobManager (а не у клиента), что снижает сетевой трафик и упрощает deployment в Kubernetes. Для production critical pipelines стандарт — Application mode.

## Связи

- [Apache Flink](../entities/apache-flink.md) — entity-страница.
- [Flink DataStream API](flink-datastream-api.md) — то, что компилируется в этот граф.
- [Flink State Management](flink-state-management.md) — где живёт state в TaskManager'ах.
- [Flink Checkpoints and Savepoints](flink-checkpoints-and-savepoints.md) — координация snapshot'ов JobManager'ом.
- [Stream Processing](stream-processing.md) — DDIA-теория.
