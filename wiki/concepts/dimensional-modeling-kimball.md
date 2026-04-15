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

## Типичные вопросы на интервью

**Q: Что такое dimensional modeling и зачем оно нужно?**
A: Методология проектирования DWH, где данные организованы вокруг бизнес-процессов: **fact tables** (события + меры) окружены **dimension tables** (контекст: кто, что, где, когда). Цель: сделать данные **понятными бизнесу** и **быстрыми для запросов** (минимум joins, удобно для BI). Альтернатива — нормализованный EDW (Inmon).

**Q: Каков порядок проектирования dimensional модели?**
A: 1) Выбрать **бизнес-процесс** (заказы, доставки). 2) Определить **grain** (одна строка = что). 3) Выбрать **dimensions** (контекст). 4) Определить **facts/measures** и проверить additivity. 5) Решить SCD-стратегию. 6) Согласовать conformed dimensions с другими витринами.

**Q: Зачем нужны surrogate keys?**
A: 1) **Независимость от источника** — natural key может меняться или конфликтовать между системами. 2) **Поддержка SCD Type 2** — у каждой версии измерения свой surrogate. 3) **Производительность** — integer joins быстрее, чем composite string keys. 4) **Стабильность** — fact всегда ссылается на конкретную версию dimension.

**Q: Что такое bus architecture?**
A: Подход Kimball к enterprise DWH: вместо одного монолитного хранилища — набор **витрин** (data marts), интегрированных через **conformed dimensions**. Bus matrix показывает, какие факты используют какие общие измерения. Позволяет строить витрины итеративно, сохраняя сопоставимость метрик.

## Связи

- [Staging and Presentation Layers](staging-and-presentation-layers.md)
- [Star Schema and Grain](star-schema-and-grain.md)
- [Dimension Tables](dimension-tables.md)
- [Fact Table Types (Kimball)](fact-table-types-kimball.md)
- [Slowly Changing Dimensions](slowly-changing-dimensions.md)
- [Conformed Dimensions](conformed-dimensions.md)
- [Kimball vs Inmon](../comparisons/kimball-vs-inmon.md)
- [Dimensional Modeling Interview Prep](../overviews/dimensional-modeling-interview-prep.md)
