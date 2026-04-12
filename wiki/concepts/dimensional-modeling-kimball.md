# Dimensional Modeling (Kimball)

**Источник:** [The Data Warehouse Toolkit](../sources/the-data-warehouse-toolkit.md)

## Зачем dimensional modeling

**Цель** — сделать данные **понятными бизнесу** и удобными для **запросов и BI**: факты (меры событий) окружены **измерениями** (контекст: кто, что, где, когда). Модель близка к тому, как люди задают вопросы («продажи по регионам за квартал»).

## Основные идеи

| Идея | Смысл |
|------|--------|
| **Star schema** | Одна (или несколько) **fact** + набор **dimension**; предпочтение денормализации измерений ради простоты и скорости запросов. |
| **Bus architecture** | Интеграция витрин через **conformed dimensions** (общие измерения и факты), а не через один гигантский «кусок» схемы. |
| **Grain** | Самый низкий уровень детализации факта; от него зависят все меры и возможные разрезы. |
| **Surrogate keys** | Искусственные ключи измерений (часто целочисленные) для стабильности и поддержки **SCD**. |

## Жизненный цикл (упрощённо)

1. Согласовать **бизнес-процессы** (какие события считаем — заказы, доставки, возвраты).
2. Задать **grain** факта.
3. Выбрать **dimensions** и **measures**; проверить **additivity**.
4. Спроектировать **SCD** для критичных измерений.
5. Встроить в **enterprise** через **conformed dimensions**.

**Где живёт модель:** звезда и факты — во **[втором слое (presentation)](staging-and-presentation-layers.md)**; приём из источников — в **staging**.

## Связи

- [Staging and Presentation Layers](staging-and-presentation-layers.md)
- [Star Schema and Grain](star-schema-and-grain.md)
- [Dimension Tables](dimension-tables.md)
- [Fact Table Types (Kimball)](fact-table-types-kimball.md)
- [Slowly Changing Dimensions](slowly-changing-dimensions.md)
- [Conformed Dimensions](conformed-dimensions.md)
- [Kimball vs Inmon](../comparisons/kimball-vs-inmon.md)
- [Dimensional Modeling Interview Prep](../overviews/dimensional-modeling-interview-prep.md)
