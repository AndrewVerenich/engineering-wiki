# Data Models and Query Languages

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (гл. 2)

## Суть

Выбор **модели данных** (реляционная, документная, графовая и т.д.) определяет, как выражать сущности, связи и запросы. Нет одной «лучшей» модели: важны паттерны доступа, нормализация vs денормализация, удобство приложения и операций. Языки запросов (SQL, declarative vs imperative) влияют на то, как оптимизатор и движок могут ускорять работу.

## Связи

- [Storage and Retrieval](storage-and-retrieval.md) — как модель ложится на физическое хранение.
- [Encoding and Schema Evolution](encoding-and-schema-evolution.md) — эволюция схем при смене модели.
- [PostgreSQL](../entities/postgresql.md) — пример реляционной модели.
