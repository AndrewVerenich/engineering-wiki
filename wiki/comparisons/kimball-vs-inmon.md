# Kimball vs Inmon

**Источники:** [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md); общая практика DW (Inmon — *Corporate Information Factory*)

## Кратко

| | **Kimball (dimensional)** | **Inmon (normalized / CIF)** |
|---|-----------------------------|------------------------------|
| **Фокус** | Витрины: **star/snowflake**, готовые к BI | **3NF enterprise data warehouse** сначала, витрины — производные |
| **Интеграция** | **Bus**, **conformed dimensions**, итеративные витрины | Единое нормализованное хранилище как **источник истины** |
| **Скорость ценности** | Быстрее отдать **конкретные** subject areas | Часто дольше «большой» слой перед витринами |
| **Аудитория модели** | Бизнес и аналитики (проще читать звезду) | Моделирование ближе к **корпоративной** нормализации |

На собеседованиях часто ждут: **оба подхода могут сосуществовать** — например нормализованный core (Inmon-стиль) + Kimball-витрины сверху; либо только Kimball для SMB/конкретных доменов.

## Типичные вопросы на интервью

**Q: Когда выбрать Kimball, а когда Inmon?**
A: **Kimball** — когда нужна быстрая ценность для конкретных доменов, команда небольшая, бизнесу нужны понятные витрины. **Inmon** — когда есть ресурсы на построение полного нормализованного EDW, данных много, много источников, нужна строгая single source of truth. На практике часто **гибрид**: нормализованный core + Kimball-витрины сверху.

**Q: Можно ли сочетать оба подхода?**
A: Да, и на собеседовании это **правильный** ответ. Пример: нормализованный staging/core layer (Inmon-стиль для полноты и гибкости) + dimensional presentation layer (Kimball-витрины для BI). В dbt это natural: staging → intermediate (normalized) → facts/dims/marts (dimensional).

**Q: В чём главный минус Kimball?**
A: Нет единого нормализованного ядра — при добавлении нового бизнес-процесса может потребоваться создание новых conformed dimensions или рефакторинг существующих. Интеграция через bus architecture, но нет «одной таблицы на сущность» для ad-hoc запросов.

## Связи

- [Dimensional Modeling (Kimball)](../concepts/dimensional-modeling-kimball.md)
- [Conformed Dimensions](../concepts/conformed-dimensions.md)
- [OLTP vs OLAP](oltp-vs-olap.md)
