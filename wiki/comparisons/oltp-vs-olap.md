# OLTP vs OLAP

**Источник:** [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md) (в т.ч. контраст рабочих и аналитических нагрузок)

## Суть

| | **OLTP** | **OLAP** |
|---|----------|----------|
| Назначение | Операционные транзакции (заказы, платежи) | Аналитика, отчёты, агрегации |
| Запросы | Много мелких, точечных | Редкие тяжёлые, сканирующие |
| Модель данных | Нормализованная (часто) | Звезда/снежинка, колоночные хранилища |
| Задержка | Низкая для одной операции | Выше, допустим batch |

Обе перспективы связаны с [Batch Processing](../concepts/batch-processing.md) (часто OLAP-данные наполняются пакетно) и [Derived Data and Systems](../concepts/derived-data-and-systems.md).

## Связи

- [Designing Data-Intensive Applications](../sources/designing-data-intensive-applications.md)
