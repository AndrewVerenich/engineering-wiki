# Encoding and Schema Evolution

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 4)

## Суть

Данные передаются и хранятся в **бинарном или текстовом виде** (JSON, Thrift, Protobuf, Avro и т.д.). Важны **совместимость вперёд и назад** при смене схемы: кто читает старые данные, кто пишет новые поля. **Avro** в книге часто приводится как формат с хорошей эволюцией схем при контракте между writer/reader.

## Связи

- [Stream Processing](stream-processing.md) — схемы в потоках событий.
- [Replication](replication.md) — репликация логов и форматов.
