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

## Связи

- [Dimensional Modeling (Kimball)](../concepts/dimensional-modeling-kimball.md)
- [Conformed Dimensions](../concepts/conformed-dimensions.md)
- [OLTP vs OLAP](oltp-vs-olap.md)
