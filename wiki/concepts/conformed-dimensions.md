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

## Связи

- [Dimensional Modeling (Kimball)](dimensional-modeling-kimball.md)
- [Fact Table Types (Kimball)](fact-table-types-kimball.md)
- [Kimball vs Inmon](../comparisons/kimball-vs-inmon.md)
