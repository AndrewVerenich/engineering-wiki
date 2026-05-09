# Kafka Streams vs Flink

**Контекст:** оба инструмента решают задачи stateful stream processing, но Kafka Streams — библиотека внутри приложения, а Flink — отдельный distributed engine.

## Главная разница

| Критерий | Kafka Streams | Flink |
|----------|---------------|-------|
| Модель запуска | JVM library в вашем сервисе | Отдельный кластерный runtime |
| Управление ресурсами | Через scaling приложения и partitions | Через JobManager/TaskManager/slots |
| Состояние | Local stores + changelog topics | Managed state + checkpoints/savepoints |
| Time model | Stream-time, без отдельного watermark протокола | Event-time + watermark как first-class |
| Типичный fit | App-centric streaming и enrichment | Сложные DE streaming pipelines |
| Операционная цена | Ниже стартовый порог | Выше, но больше контроля и мощности |

## Когда что выбирать

| Сценарий | Лучший выбор | Почему |
|----------|--------------|--------|
| Уже есть JVM микросервис и нужен stream layer | **Kafka Streams** | Легко встроить без отдельного runtime. |
| Сложные multi-stage pipeline и богатая time-логика | **Flink** | Глубокие возможности event-time и state orchestration. |
| Минимизировать инфраструктурные компоненты | **Kafka Streams** | Достаточно Kafka-кластера и приложения. |
| Нужен единый движок для широкой DE-платформы | **Flink** | Сильная модель deployment/operations для data pipelines. |

## Типичные вопросы на интервью

**Q: Можно ли считать Kafka Streams «маленьким Flink»?**  
A: Нет, это другой operational trade-off. Kafka Streams проще встраивается в backend-сервис, Flink даёт более мощный и универсальный engine-level контроль.

**Q: Где проще обеспечить low-latency read path через materialized state?**  
A: В Kafka Streams это естественно через Interactive Queries в том же приложении. Во Flink обычно строят отдельный serving слой.

**Q: Чем отличается подход к exactly-once?**  
A: Kafka Streams использует Kafka transactions (`exactly_once_v2`), Flink опирается на distributed checkpoints и согласование с sink.

**Q: Что выбрать для команды без DE платформы, но с сильным Java backend?**  
A: Часто Kafka Streams как первый шаг: ниже порог, быстрее delivery, а затем при росте сложности можно эволюционировать в выделенный engine.

## Связи

- [Kafka Streams Architecture](../concepts/kafka-streams-architecture.md)
- [Kafka Streams Exactly-Once](../concepts/kafka-streams-exactly-once.md)
- [Apache Flink Architecture](../concepts/flink-architecture.md)
- [Flink Exactly-Once Semantics](../concepts/flink-exactly-once-semantics.md)
