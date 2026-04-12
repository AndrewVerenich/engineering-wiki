# LLM Wiki Schema (Java Backend + Spring + Kotlin + Distributed Systems + System Design + Data Engineering)

## Purpose
Эта вики — персональная база знаний инженера.  
LLM полностью отвечает за создание, обновление и поддержание всех страниц.  
Моя задача — добавлять источники (книги, статьи, курсы) и задавать вопросы.  
Твоя задача — поддерживать структуру, связи и целостность вики.

---

# 1. Types of pages

## 1.1 Source pages
Страницы отдельных источников (книги, статьи, курсы, видео).
Хранятся в /wiki/sources/.

## 1.2 Concept pages
Страницы концептов, технологий, паттернов.
Примеры:
- JVM Memory Model
- Spring IoC
- Kotlin Coroutines
- CAP Theorem
- Event Sourcing
- OLTP vs OLAP
- Kafka Partitions
Хранятся в /wiki/concepts/.

## 1.3 Entity pages
Конкретные сущности:
- фреймворки (Spring Boot, Micronaut)
- инструменты (Kafka, Spark, Flink, Airflow)
- базы данных (Postgres, ClickHouse)
Хранятся в /wiki/entities/.

## 1.4 Overview pages
Обзоры тем:
- Java Backend Overview
- Spring Ecosystem Overview
- Distributed Systems Overview
- System Design Overview
- Data Engineering Roadmap
Хранятся в /wiki/overviews/.

## 1.5 Comparison pages
Сравнения:
- Kafka vs Pulsar
- Spark vs Flink
- Spring Boot vs Micronaut
Хранятся в /wiki/comparisons/.

---

# 2. Rules for ingest

При добавлении нового источника LLM должен:

1. Создать страницу источника в /wiki/sources/.
2. Выделить ключевые концепты и создать/обновить соответствующие concept/entity страницы.
3. Добавить cross-links между страницами.
4. Обновить /wiki/index.md.
5. Добавить запись в /wiki/log.md.
6. Строго следовать структуре SCHEMA.md.

---

# 3. Rules for query

При ответе на вопросы:

1. Читать index.md.
2. Находить релевантные страницы.
3. Давать структурированный ответ с цитатами.
4. Предлагать сохранить ответ как новую страницу (если он полезен).

---

# 4. Rules for lint

Периодически:

- искать противоречия,
- находить устаревшие страницы,
- предлагать новые концепты,
- улучшать cross-links,
- обновлять index.md.

---
