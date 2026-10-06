# AutoSalesApi

REST API для системы продаж автомобилей (AutoSales), реализованный на ASP.NET Core Web API (net10.0).

Архитектура построена по образцу [rakdav/RestAPI](https://github.com/rakdav/RestAPI.git): контроллеры + сервисы с абстракциями, EF Core DbContext, Swagger.

## Структура

```
AutoSalesApi/
├── Controllers/           # Producer, Model, Client, PriceList, Order, Report
├── Models/
│   ├── Abstractions/      # ICommonService<T,K>, AbstractionService
│   ├── Data/              # AutoSalesDbContext, SeedData
│   ├── Dto/               # PriceListItem, ModelWithPrice, OrderItem, ReportResult
│   ├── Services/          # ProducerService, ModelService, ClientService,
│   │                      # PriceListService, OrderService, ReportService
│   ├── Producer.cs        # сущности
│   ├── Model.cs
│   ├── Offer.cs
│   ├── PriceList.cs
│   ├── Client.cs
│   └── Order.cs
├── Program.cs             # DI, Swagger, CORS, кэш
└── appsettings.json       # строка подключения SQLite
AutoSales.slnx
```

## Сущности

| Сущность | Описание | Связи |
|---|---|---|
| `Producer` | Поставщик (фирма) | M:N с `Model` через `Offer` |
| `Model` | Модель автомобиля | 1:1 с `PriceList`, M:N с `Producer` |
| `Offer` | Связь поставщик–модель | — |
| `PriceList` | Прейскурант | 1:1 с `Model` |
| `Client` | Клиент | 1:N с `Order` |
| `Order` | Продажа (заказ) | → `Client`, → `Model` |

## Эндпоинты

| Метод | Путь | Описание |
|---|---|---|
| GET | `/api/producer?page=1` | Поставщики постранично (кэш 5 сек) |
| GET | `/api/producer/all` | Все поставщики |
| GET | `/api/producer/{id}` | Поставщик по id |
| POST | `/api/producer` | Создать поставщика |
| PUT | `/api/producer/{id}` | Обновить поставщика |
| DELETE | `/api/producer/{id}` | Удалить поставщика |
| GET | `/api/model/all` | Все модели с прейскурантом |
| GET | `/api/model/{id}` | Модель по id (с прейскурантом и поставщиками) |
| POST | `/api/model` | Создать модель |
| POST | `/api/model/with-details` | Создать модель + прейскурант + связи с поставщиками |
| PUT | `/api/model/{id}/offers` | Обновить связи модели с поставщиками |
| DELETE | `/api/model/{id}` | Удалить модель (каскадно прейскурант и связи) |
| GET | `/api/client/all` | Все клиенты |
| POST | `/api/client` | Создать клиента |
| DELETE | `/api/client/{id}` | Удалить клиента (запрещено при наличии заказов) |
| GET | `/api/pricelist` | Прейскурант (строки с названием модели и полной стоимостью) |
| POST | `/api/pricelist` | Добавить прейскурант (1:1 с моделью) |
| DELETE | `/api/pricelist/{id}` | Удалить запись прейскуранта |
| GET | `/api/order` | Продажи (строки с ФИО клиента и названием модели) |
| GET | `/api/order/models-with-price` | Модели с ценой (для формы продажи) |
| GET | `/api/order/next-number` | Следующий номер договора |
| POST | `/api/order` | Оформить продажу (стоимость считается по прейскуранту) |
| DELETE | `/api/order/{id}` | Удалить продажу |
| GET | `/api/report` | Список отчётов |
| GET | `/api/report/{index}` | Отчёт по индексу (0..4) |
| GET | `/api/report/by-name?name=...` | Отчёт по имени |

### Отчёты

0. Сводка по поставщикам
1. Сводка по моделям
2. Сводка по клиентам
3. Непроданные модели
4. Доходность по моделям

## Технологии

- ASP.NET Core Web API (net10.0)
- Entity Framework Core + SQLite (`Data Source=autosales.db`)
- Swashbuckle (Swagger UI)
- `IMemoryCache`, `ResponseCaching`, CORS

## Запуск

```bash
dotnet run --project AutoSalesApi
```

При первом запуске база SQLite создаётся и наполняется демонстрационными данными автоматически.

Swagger UI: http://localhost:5103/swagger

Примеры всех запросов — в файле `AutoSalesApi/AutoSalesApi.http`.
