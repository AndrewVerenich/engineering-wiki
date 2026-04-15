# Conformed Dimensions

**Источник:** [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md)

## Определение

**Conformed dimension** — измерение, которое **согласовано** между витринами: **одни и те же ключи и атрибуты** (или **подмножества** с общим ядром), чтобы метрики были **сопоставимы** и их можно было **объединять** в одном отчёте.

Пример: общая **Date**, общий **Product** на уровне SKU или бренда — тогда «продажи» и «закупки» можно сравнивать в одних разрезах.

## Bus matrix (матрица шины)

**Bus matrix** — таблица: **строки = бизнес-процессы (факты)**, **столбцы = conformed dimensions**. Показывает, какие витрины должны **делить** измерения; основа для **планирования** enterprise DW и **параллельной** разработки команд.

## Drill-across

**Drill-across** — запрос, который **объединяет** метрики из **разных** fact-таблиц по **общим conformed dimensions** (часто через `UNION`/агрегации с общими ключами). Без conformed dimensions такие отчёты **некорректны** или невозможны.

## Практика

- **Enterprise agreement** на определения (что такое «клиент», «регион»).
- **Один источник правды** для справочников измерений (master data) или чёткие правила синхронизации.

## Типичные вопросы на интервью

**Q: Что такое conformed dimension и зачем она нужна?**
A: Dimension, **согласованная** между витринами: одни и те же ключи, атрибуты, определения. Без неё метрики «продажи» и «закупки» нельзя корректно сравнить в одном отчёте — разные справочники дадут разные результаты. Conformed dimensions — основа **enterprise DWH** в Kimball.

**Q: Что такое bus matrix?**
A: Таблица: строки = бизнес-процессы (facts), столбцы = conformed dimensions. Показывает, какие витрины **разделяют** измерения. Инструмент **планирования** enterprise DWH: какие marts строить первыми, какие dimensions стандартизировать.

**Q: Что такое drill-across?**
A: Запрос, объединяющий метрики из **разных** fact-таблиц по **общим conformed dimensions**. Пример: «выручка (из fct_sales) vs закупки (из fct_purchases) по продуктам и регионам». Без conformed dimensions drill-across даёт **некорректные** результаты или невозможен.

**Q: Как обеспечить conformity на практике?**
A: 1) **Enterprise agreement** на определения (что такое «клиент», «регион»). 2) **Single source of truth** для справочников (master data management). 3) В dbt — общие dimension-модели, от которых зависят все marts. 4) Тесты на relationships между facts и shared dimensions.

## Связи

- [Dimensional Modeling (Kimball)](dimensional-modeling-kimball.md)
- [Fact Table Types (Kimball)](fact-table-types-kimball.md)
- [Kimball vs Inmon](../comparisons/kimball-vs-inmon.md)
