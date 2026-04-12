# Dimensional Modeling — подготовка к собеседованию

Сжатый чеклист по **Kimball / dimension modeling**. Детали — на страницах концептов.

## Определения (наизусть)

- **Fact** — строки **событий/измерений бизнеса** + **числовые меры** (и FK на измерения).
- **Dimension** — **контекст** (кто, что, где, когда); атрибуты для фильтров и группировок.
- **Grain** — что **одна строка** факта означает; должен быть **один** для таблицы.
- **Star vs snowflake** — компромисс **простоты запросов** vs **нормализации**; Kimball чаще **star**.

## Частые вопросы

### В чём разница OLTP и модели витрины?

OLTP — **нормализация**, мелкие транзакции, целостность; витрина — **денормализация** под **аналитику** и сканы. См. [OLTP vs OLAP](../comparisons/oltp-vs-olap.md).

### Что такое additive / semi-additive / non-additive?

См. [Star Schema and Grain](../concepts/star-schema-and-grain.md): примеры — сумма продаж vs баланс счёта vs ratio.

### Объясни SCD Type 1 и 2

См. [Slowly Changing Dimensions](../concepts/slowly-changing-dimensions.md): перезапись vs новая строка + surrogate.

### Три типа fact-таблиц Kimball

**Transaction**, **periodic snapshot**, **accumulating snapshot** — [Fact Table Types](../concepts/fact-table-types-kimball.md).

### Что такое conformed dimension и drill-across?

См. [Conformed Dimensions](../concepts/conformed-dimensions.md).

### Degenerate vs junk dimension

См. [Dimension Tables](../concepts/dimension-tables.md).

### Kimball vs Inmon

См. [Kimball vs Inmon](../comparisons/kimball-vs-inmon.md).

### Зачем два слоя: staging и presentation?

**Staging** — приём и фиксация данных из источников (повторные прогоны, аудит). **Presentation** — витрина со звездой, surrogate keys, SCD, conformed dimensions для BI. Подробно: [Staging and Presentation Layers](../concepts/staging-and-presentation-layers.md).

## Порядок изучения страниц

1. [Dimensional Modeling (Kimball)](../concepts/dimensional-modeling-kimball.md)  
2. [Staging and Presentation Layers](../concepts/staging-and-presentation-layers.md) — контекст пайплайна (staging ↔ витрина)  
3. [Star Schema and Grain](../concepts/star-schema-and-grain.md)  
4. [Dimension Tables](../concepts/dimension-tables.md)  
5. [Fact Table Types (Kimball)](../concepts/fact-table-types-kimball.md)  
6. [Slowly Changing Dimensions](../concepts/slowly-changing-dimensions.md)  
7. [Conformed Dimensions](../concepts/conformed-dimensions.md)  
8. [Advanced Dimensional Patterns](../concepts/advanced-dimensional-patterns.md)  

**Источник книги:** [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md)
