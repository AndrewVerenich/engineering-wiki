# Encoding and Schema Evolution

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 4)

## Суть

Данные передаются и хранятся в **бинарном или текстовом виде** (JSON, Thrift, Protobuf, Avro и т.д.). Важны **совместимость вперёд и назад** при смене схемы: кто читает старые данные, кто пишет новые поля. **Avro** в книге часто приводится как формат с хорошей эволюцией схем при контракте между writer/reader.

## Форматы сериализации

| Формат | Schema | Forward compat | Backward compat | Типичное применение |
|--------|--------|---------------|-----------------|---------------------|
| **JSON** | Нет (или JSON Schema) | Игнорируем незнакомые поля | Опциональные поля | REST API, логи, конфиги |
| **Protobuf** | `.proto` файл, поля по номерам | Старый reader игнорирует новые поля | Новые поля optional | gRPC, внутренние сервисы |
| **Avro** | JSON/IDL schema, writer/reader schema resolution | Writer schema → reader schema маппинг | Добавление/удаление полей с default | Kafka, Hadoop, data pipelines |
| **Thrift** | `.thrift` файл, поля по номерам | Аналогично Protobuf | Аналогично Protobuf | Legacy Facebook-стек |
| **Parquet** | Встроенная schema в файл | Column-level evolution | Добавление колонок | Data lake, Spark, analytics |

## Ключевые термины

| Термин | Определение |
|--------|-------------|
| **Forward compatibility** | Старый код читает данные, записанные новым кодом (игнорирует незнакомые поля). |
| **Backward compatibility** | Новый код читает данные, записанные старым кодом (новые поля имеют default). |
| **Schema registry** | Централизованное хранилище схем (Confluent Schema Registry для Kafka); валидирует совместимость при эволюции. |
| **Schema-on-read** | Интерпретация структуры при чтении (data lake); гибко, но ошибки поздно. |
| **Schema-on-write** | Валидация при записи (RDBMS, Avro + registry); строго, но миграции дороже. |

## Типичные вопросы на интервью

**Q: Что такое forward и backward compatibility и зачем обе нужны?**
A: **Backward** — новый код читает старые данные (нужно при деплое нового сервиса, а старые данные ещё в Kafka/storage). **Forward** — старый код читает новые данные (нужно при rolling deploy, когда часть инстансов ещё на старой версии). Для безопасного деплоя нужны **обе**.

**Q: Почему Avro популярен в data engineering?**
A: Writer/reader schema resolution: writer пишет со своей schema, reader читает со своей, Avro делает маппинг по именам полей. Это позволяет эволюцию без перекодировки данных. Плюс компактный бинарный формат и интеграция с Hadoop/Kafka.

**Q: Зачем нужен Schema Registry?**
A: Централизованный контроль совместимости схем: при публикации новой версии schema проверяется, что она backward/forward compatible с предыдущими. Без этого → broken consumers, corrupted data, тихие ошибки в pipeline.

**Q: Как в CDC pipeline обеспечить schema evolution?**
A: Landing/staging слой допускает эволюцию (Avro + schema registry или JSON с гибким парсингом). Downstream модели (dims/facts) меняются контролируемо: через dbt migrations, тесты на контракт полей, и мониторинг DDL в источнике.

## Связи

- [Stream Processing](stream-processing.md) — схемы в потоках событий.
- [Replication](replication.md) — репликация логов и форматов.
- [CDC: Debezium → Kafka → ClickHouse](cdc-debezium-analytics-pipeline.md) — schema evolution в CDC.