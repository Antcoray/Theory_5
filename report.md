# Отчёт по лабораторным работам — WEB_453501_Kolesnikow

Сквозной проект — меню пиццерии. Предметная область: `Pizza` (пицца) и `Category` (категория).
В методичке объект называется `Dish`, у нас вместо него `Pizza` (свойства `Price`, `ImageFilePath`, `MIME`).

Код развивается от лабы к лабе: в разделе каждой ЛР описано, что было сделано в ней, а более поздние ЛР иногда меняют тот же файл (например, PUT/POST в API — JSON в ЛР4, `multipart/form-data` в ЛР5; адрес каталога `/Product` в ЛР4–ЛР7, `/Catalog` с ЛР8). В самом коде такие места помечены комментариями `// ЛР5`, `// ЛР6`, `// ЛР8`.

| Проект | Назначение | Порты (https/http) |
|---|---|---|
| `WEB_453501_Kolesnikow.UI` | сайт MVC: каталог, корзина, админка | 7001 / 5001 |
| `WEB_453501_Kolesnikow.API` | REST API + Minimal API, EF Core, PostgreSQL, MediatR | 7002 / 5002 |
| `WEB_453501_Kolesnikow.Domain` | общие классы: `Pizza`, `Category`, `ResponseData<T>`, `ListModel<T>`, `Cart`, `CartItem` | — |

Контейнеры Docker (все описаны в `WEB_453501_Kolesnikow.API/DB/compose.yaml`):

| Контейнер | Назначение | Порт | С какой ЛР |
|---|---|---|---|
| `postgres` | СУБД PostgreSQL (БД `MenuDvb_V01` — данные сайта, БД `keycloak` — данные Keycloak) | 5432 | ЛР4 |
| `pgadmin4` | веб-интерфейс PostgreSQL | 5454 | ЛР4 |
| `keycloak` | сервер аутентификации | 8080 | ЛР6 |
| `redis` | распределённый кэш | 6379 | ЛР8 |

## Как запустить

```powershell
# 1. Контейнеры (Docker Desktop должен быть запущен) — из папки WEB_453501_Kolesnikow.API/DB:
docker compose --env-file postgres.env up -d
#    на новой машине дополнительно: создать БД для Keycloak и настроить его
#    docker exec postgres psql -U admin -d postgres -c "CREATE DATABASE keycloak;"
#    docker compose --env-file postgres.env restart keycloak
#    bash keycloak-setup.sh      (в Git Bash; выведет новый Client Secret → UI/appsettings.json)

# 2. API — в отдельном терминале, из папки решения
dotnet run --project WEB_453501_Kolesnikow.API --launch-profile https

# 3. UI — во втором терминале
dotnet run --project WEB_453501_Kolesnikow.UI --launch-profile https
```

- Сайт: https://localhost:7001, каталог: https://localhost:7001/Catalog, админка: https://localhost:7001/Admin.
- Пользователи сайта (пароль `123456`): `admin@example.com` (роль `POWER-USER`, админка), `user@example.com`, `new@example.com`.
- Консоль Keycloak: http://localhost:8080 (`admin` / `123456`), realm `Kolesnikow`.
- pgAdmin: http://localhost:5454 (`admin@example.com` / `123456`), сервер: Host `postgres`, Port 5432, `admin` / `123456`.
- Ручная проверка API: файл `WEB_453501_Kolesnikow.API/WEB_453501_Kolesnikow.API.http` (в VS Code — расширение REST Client, ссылка *Send Request*). С ЛР6 запросы по `id`, POST/PUT/DELETE требуют заголовок `Authorization: Bearer <токен>` (получение токена — ЛР6, п. 4.5).

---

## ЛР4. Работа с REST API

**Цель:** знакомство с принципом работы сервисов REST; научиться создавать REST API и обращаться к нему из приложения.

**Итог:** создан проект `WEB_453501_Kolesnikow.API` с БД PostgreSQL. Категории отдаёт REST-контроллер `/api/Categories`, пиццы — Minimal API `/api/Pizza` (список — через MediatR, с фильтром по категории и пагинацией). Сайт `UI` больше не хранит данные в памяти, а получает их из API через `HttpClient`.

### Что сделано по пунктам задания

**3.1. Создание проекта**
- Проект ASP.NET Core Web API `WEB_453501_Kolesnikow.API` (.NET 9) добавлен в решение, есть ссылка на `WEB_453501_Kolesnikow.Domain`.
- Порты 7002/5002 и `"launchBrowser": false` — `Properties/launchSettings.json` (API вызывает не человек, а UI, браузер не нужен).
- Пакеты NuGet: `Microsoft.EntityFrameworkCore` 9.0.20, `Microsoft.EntityFrameworkCore.Tools` 9.0.20, `Microsoft.VisualStudio.Web.CodeGeneration.Design` 9.0.12.
- Изображения скопированы в `wwwroot/Images` (1.jpg … 6.jpg). В `Program.cs` подключён Middleware статических файлов `app.UseStaticFiles()` — без него `https://localhost:7002/Images/1.jpg` давал бы 404. Он стоит до `UseAuthorization()`, чтобы картинки не проходили проверку доступа (пригодится в ЛР6).
- Одновременный запуск: в VS — *Multiple startup projects* (`WEB_453501_Kolesnikow.slnLaunch.user`), в VS Code — два терминала (см. «Как запустить»).
- Шаблонный `WeatherForecast.cs` удалён.

**3.2. Выбор БД** — PostgreSQL, пакет `Aspire.Npgsql.EntityFrameworkCore.PostgreSQL` (подтягивает провайдер `Npgsql.EntityFrameworkCore.PostgreSQL`, в котором находится `UseNpgsql`).

**3.3. Сервер PostgreSQL в Docker** — `DB/compose.yaml` описывает два контейнера: `postgres` (сервер СУБД, порт 5432, данные в томе `postgres_data`) и `pgadmin` (веб-интерфейс, порт 5454). Логины/пароли — в `DB/postgres.env`. Запуск: `docker compose --env-file postgres.env up -d`.

**3.4. Контекст БД** — `Data/AppDbContext.cs`: наследник `DbContext`, конструктор принимает `DbContextOptions<AppDbContext>`, свойства `DbSet<Pizza> Pizzas` и `DbSet<Category> Categories` — это таблицы, к которым пишутся LINQ-запросы.

**3.5. Начальная миграция**
- Строка подключения `ConnectionStrings:Postgres` в `appsettings.json` (БД `MenuDvb_V01`, пользователь `admin`).
- Регистрация контекста в `Program.cs`: `builder.Services.AddDbContext<AppDbContext>(options => options.UseNpgsql(connString));` — контекст регистрируется как **Scoped** (один на HTTP-запрос).
- Миграция `Migrations/20260928145354_InitialCreate.cs` создаёт таблицы `Categories` и `Pizzas` (внешний ключ `Pizzas.CategoryId → Categories.Id`).

**3.6. Заполнение начальными данными** — `Data/DBInitializer.cs`, метод `public static async Task SeedData(WebApplication app)`, вызывается в `Program.cs` сразу после `builder.Build()`:
- создаёт область `app.Services.CreateScope()` и берёт из неё `AppDbContext` (контекст — Scoped-сервис, а при старте приложения HTTP-запроса ещё нет);
- `MigrateAsync()` применяет миграции (на новой машине БД создастся сама);
- если пицц ещё нет — добавляет 3 категории и 6 пицц, `Id` не указывается (назначает БД), `SaveChangesAsync()`;
- адрес API для картинок берётся из `appsettings.json` (`"ApiUrl": "https://localhost:7002"`), в БД пишется полный URL `https://localhost:7002/Images/1.jpg` — картинку рисует UI (порт 7001), а файл лежит в API (порт 7002).

**3.7. Контроллер API для категорий** — `Controllers/CategoriesController.cs`, сгенерирован командой
`dotnet aspnet-codegenerator controller -name CategoriesController -async -api -m Category -dc AppDbContext -outDir Controllers`.
Маршрут `/api/Categories`, методы GET (все / по id), PUT, POST, DELETE. В `Program.cs` — `builder.Services.AddControllers()` и `app.MapControllers()`.

**3.8. Minimal API для пицц** — `Endpoints/PizzaEndpoints.cs`, метод-расширение `MapPizzaEndpoints()`, в `Program.cs` — `app.MapPizzaEndpoints();`. Группа `routes.MapGroup("/api/Pizza")`:

| Запрос | Что делает |
|---|---|
| `GET /api/Pizza/{category?}?pageNo=&pageSize=` | список (п. 3.10) |
| `GET /api/Pizza/{id:int}` | одна пицца, вместе с категорией (`.Include(d => d.Category)`) |
| `PUT /api/Pizza/{id:int}` | изменить (`ExecuteUpdateAsync` — один SQL `UPDATE`) |
| `POST /api/Pizza` | добавить, ответ **201 Created** |
| `DELETE /api/Pizza/{id:int}` | удалить (`ExecuteDeleteAsync`) |

Правки после генерации: ограничение `{id:int}` (иначе `/api/Pizza/cheese` подходил бы под оба маршрута); из PUT убрана строка `SetProperty(m => m.Id, ...)` (нельзя перезаписывать первичный ключ); в PUT добавлено `SetProperty(m => m.CategoryId, ...)`.
В сущность `Pizza` добавлено свойство `int CategoryId` (внешний ключ) рядом с навигационным свойством `Category?` — без него POST/PUT не могли передать категорию числом (`"categoryId": 1`). Столбец `CategoryId` в БД уже был, новая миграция не понадобилась.

**3.9. Проверка API** — запросы в `WEB_453501_Kolesnikow.API.http` (GET можно и в адресной строке браузера). Результаты:

| Запрос | Ответ |
|---|---|
| `GET /api/Categories`, `GET /api/Categories/1` | 200, JSON с категориями |
| `GET /api/Pizza/1` | 200, пицца с вложенным объектом `category` |
| `POST /api/Pizza` | 201 Created, новая пицца с `id` |
| `PUT /api/Pizza/{id}` | 200, повторный GET показывает новые данные |
| `DELETE /api/Pizza/{id}` | 200, повторный DELETE — 404 |
| `GET /Images/1.jpg` | 200, `image/jpeg` |

Примечание: в шаблоне .NET 9 нет страницы Swagger — `AddOpenApi()`/`MapOpenApi()` отдают только JSON-описание `/openapi/v1.json`, поэтому проверка — через `.http`-файл.

**3.10. Доработка `MapGet("/")` — MediatR**
- Пакет `MediatR`, регистрация: `builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));`.
- `Use-Cases/GetListOfProducts.cs`: запрос `record GetListOfProducts(string? CategoryNormalizedName, int PageNo = 1, int PageSize = 3) : IRequest<ResponseData<ListModel<Pizza>>>` и обработчик `GetListOfProductsHandler(AppDbContext db)`.
- Логика обработчика: `pageSize` ограничен `Math.Clamp(pageSize, 1, _maxPageSize)` (`_maxPageSize = 20`) → фильтр по `Category.NormalizedName` → `CountAsync` и число страниц → `OrderBy(Id).Skip((pageNo-1)*pageSize).Take(pageSize)`. Ответ — `ResponseData<ListModel<Pizza>>`. EF Core собирает всё в один SQL-запрос.
- Конечная точка только отправляет запрос: `mediator.Send(new GetListOfProducts(category, pageNo, pageSize))`. Категория — сегмент маршрута `{category?}`, номер и размер страницы — строка запроса. Ограничение `alpha` из методички не используется, чтобы проходили имена категорий с дефисом.

**3.11. Проверка фильтрации и пагинации** (запросы — в конце `.http`-файла):

| Запрос | Результат |
|---|---|
| `/api/Pizza` | страница 1 из 2, пиццы 1–3 |
| `/api/Pizza?pageNo=2` | страница 2 из 2, пиццы 4–6 |
| `/api/Pizza/cheese` | только сырные (2 шт.), 1 страница |
| `/api/Pizza/classic?pageNo=2&pageSize=1` | страница 2 из 2, «Охотничья» |
| `/api/Pizza?pageSize=100` | все 6 на одной странице (не больше 20) |
| `/api/Pizza?pageNo=50` | `successfull: false`, «Нет такой страницы» |
| `/api/Pizza?pageNo=0` | `successfull: false`, «Номер страницы должен быть больше 0» |
| `/api/Pizza/seafood` | `successfull: true`, пустой список |

Ошибки приходят с HTTP 200: сам запрос выполнен, а неуспех бизнес-операции передаётся внутри `ResponseData` (`Successfull`, `ErrorMessage`).

**3.12. Получение данных с API в проекте UI**
- 3.12.1. Класс `UriData` (`UriData.cs`, свойство `ApiUri`) и раздел `"UriData": { "ApiUri": "https://localhost:7002/api/" }` в `appsettings.json`. Объект читается из конфигурации: `builder.Configuration.GetSection("UriData").Get<UriData>()` — в `Extensions/HostingExtensions.cs`, там же, где регистрируются сервисы (с ЛР3 регистрация вынесена из `Program.cs` в этот метод-расширение).
- 3.12.2. `Services/CategoryService/ApiCategoryService.cs` — GET к `/api/categories/`, массив категорий оборачивается в `ResponseData`.
- 3.12.2. `Services/ProductService/ApiProductService.cs` — реализует все методы `IProductService`:
  - `GetProductListAsync` — URL собирается `StringBuilder`-ом: базовый адрес + категория + `?pageNo=…&pageSize=…` (размер страницы — `ItemsPerPage` из `appsettings.json`), ответ читается `ReadFromJsonAsync<ResponseData<ListModel<Pizza>>>`, при ошибке — запись в лог и `ResponseData.Error(...)`;
  - `GetProductByIdAsync` — GET `…/pizza/{id}`;
  - `CreateProductAsync` / `UpdateProductAsync` / `DeleteProductAsync` — `HttpRequestMessage` с методом POST/PUT/DELETE, тело — JSON (`Content-Type: application/json`), отправка `_httpClient.SendAsync`. Передача файла — в ЛР5.
- Регистрация типизированных клиентов в `HostingExtensions.cs` (вместо `MemoryCategoryService`/`MemoryProductService` из ЛР3):
  ```csharp
  builder.Services.AddHttpClient<IProductService, ApiProductService>(opt =>
      opt.BaseAddress = new Uri(uriData.ApiUri + "pizza/"));
  builder.Services.AddHttpClient<ICategoryService, ApiCategoryService>(opt =>
      opt.BaseAddress = new Uri(uriData.ApiUri + "categories/"));
  ```
  Завершающий `/` нужен, чтобы относительный адрес `"5"` дописывался к `…/pizza/`, а не заменял последний сегмент.
- `ProductController` при этом не изменился: он работает с интерфейсами, а какая реализация подставится — решает DI.
- Проверка: https://localhost:7001/Product показывает пиццы из БД, картинки грузятся с `https://localhost:7002/Images/...`, фильтр по категориям и переключение страниц работают.
- Попутно исправлено: `Views/Product/Index.cshtml` был сохранён в кодировке Windows-1251 (русский текст отображался «кракозябрами») — пересохранён в UTF-8; в `_Layout.cshtml` пути к CSS/JS заменены на `~/lib/...` (относительные пути ломались на адресах вида `/Product/Index`); профиль `https` в `launchSettings.json` UI исправлен на `https://localhost:7001;http://localhost:5001`.

**3.13. Вертикальные срезы** — необязательный пункт, не выполнялся.

### Ответы на контрольные вопросы

**1. Чем контроллер API отличается от обычного?**

Обычный (MVC) контроллер наследуется от `Controller` и возвращает HTML через представления (`return View(...)`). Контроллер API наследуется от `ControllerBase` (без поддержки представлений) и возвращает данные — объекты, которые сериализуются в JSON, и коды состояния HTTP. Его клиент — не обязательно браузер, а любая программа. Атрибут `[ApiController]` добавляет удобства: автоматический ответ 400 при невалидной модели, привязку сложных параметров из тела запроса без `[FromBody]`, обязательную атрибутную маршрутизацию.

**Где в проекте:**
- обычный контроллер — [UI/Controllers/Product.cs](WEB_453501_Kolesnikow.UI/Controllers/Product.cs):
  ```csharp
  public class ProductController : Controller
  {
      public async Task<IActionResult> Index(string? category, int pageNo = 1)
      {
          ...
          return View(productResponse.Data);   // HTML: представление Views/Product/Index.cshtml
      }
  }
  ```
- контроллер API — [API/Controllers/CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs):
  ```csharp
  [Route("api/[controller]")]
  [ApiController]
  public class CategoriesController : ControllerBase
  {
      [HttpGet]
      public async Task<ActionResult<IEnumerable<Category>>> GetCategories()
      {
          return await _context.Categories.ToListAsync();   // JSON-массив категорий
      }
  }
  ```

**2. Как выбирается Action контроллера API?**

По атрибутной маршрутизации: шаблон контроллера `[Route("api/[controller]")]` (`[controller]` заменяется именем класса без суффикса `Controller`) + атрибут HTTP-метода с шаблоном у действия (`[HttpGet]`, `[HttpGet("{id}")]`, `[HttpPut("{id}")]`, `[HttpPost]`, `[HttpDelete("{id}")]`). Действие выбирается по совпадению **URL и HTTP-метода**, имя действия в адрес не входит.

**Где в проекте:** [API/Controllers/CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs)

| Запрос | Атрибут | Метод |
|---|---|---|
| `GET /api/Categories` | `[HttpGet]` | `GetCategories()` |
| `GET /api/Categories/1` | `[HttpGet("{id}")]` | `GetCategory(int id)` |
| `PUT /api/Categories/1` | `[HttpPut("{id}")]` | `PutCategory(int id, Category category)` |
| `POST /api/Categories` | `[HttpPost]` | `PostCategory(Category category)` |
| `DELETE /api/Categories/1` | `[HttpDelete("{id}")]` | `DeleteCategory(int id)` |

**3. Где и в каком виде передаются данные в контроллер API?**

- в маршруте (сегмент URL) — `[FromRoute]`, например `id`;
- в строке запроса (`?pageNo=2`) — `[FromQuery]`;
- в теле запроса, обычно JSON — `[FromBody]` (для сложных типов в `[ApiController]` и Minimal API — по умолчанию); нужен заголовок `Content-Type: application/json`, иначе 415. Форма / файл — `multipart/form-data`, `[FromForm]` (ЛР5);
- в заголовках — `[FromHeader]` (в ЛР6 — токен в `Authorization`);
- из DI — `[FromServices]` (в Minimal API сервисы берутся из DI автоматически).

**Где в проекте:**
- маршрут + строка запроса + DI — [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs):
  ```csharp
  // GET /api/Pizza/cheese?pageNo=2&pageSize=3
  group.MapGet("/{category?}", async (IMediator mediator, string? category, int pageNo = 1, int pageSize = 3) => ...
  //                                  ^ из DI          ^ из маршрута     ^ из строки запроса
  ```
- маршрут + JSON-тело — [API/Controllers/CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs):
  ```csharp
  [HttpPut("{id}")]
  public async Task<IActionResult> PutCategory(int id, Category category)   // id — из маршрута, category — из JSON-тела
  ```
- как UI формирует такой запрос — [UI/Services/ProductService/ApiProductService.cs](WEB_453501_Kolesnikow.UI/Services/ProductService/ApiProductService.cs):
  ```csharp
  var urlString = new StringBuilder(_httpClient.BaseAddress!.AbsoluteUri);   // https://localhost:7002/api/pizza/
  if (categoryNormalizedName != null)
      urlString.Append(categoryNormalizedName);                              // категория — в маршрут
  urlString.Append($"?pageNo={pageNo}&pageSize={_pageSize}");                // страница — в строку запроса
  ```

**4. Как получить Scoped-сервис в коде?**

Обычно — внедрением зависимости в конструктор или параметр метода: внутри HTTP-запроса уже есть своя область (scope). Вне запроса (например, при старте приложения) область нужно создать вручную через `CreateScope()`. Из корневого `app.Services` Scoped-сервис брать нельзя: он прожил бы всё время работы приложения.

**Где в проекте:**
- регистрация Scoped-сервиса — [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs): `builder.Services.AddDbContext<AppDbContext>(options => options.UseNpgsql(connString));`
- получение вне запроса — [API/Data/DBInitializer.cs](WEB_453501_Kolesnikow.API/Data/DBInitializer.cs):
  ```csharp
  using var scope = app.Services.CreateScope();
  var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
  ```
- получение внутри запроса через конструктор — [CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs): `public CategoriesController(AppDbContext context)`
- через параметр лямбды — [PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs): `group.MapGet("/{id:int}", async ... (int id, AppDbContext db) => ...)`

**5. Что могут возвращать методы контроллера API?**

- конкретный тип (объект, коллекцию) — сериализуется в JSON с кодом 200;
- `IActionResult` / `ActionResult<T>` — результат с кодом состояния: `Ok()` (200), `CreatedAtAction()` (201 + заголовок `Location`), `NoContent()` (204), `BadRequest()` (400), `Unauthorized()` (401), `NotFound()` (404), `StatusCode(…)`;
- асинхронно — `Task<…>` от любого из них.

В Minimal API аналог — `TypedResults.Ok/Created/NotFound` и тип `Results<Ok<T>, NotFound>`, перечисляющий возможные ответы.

**Где в проекте:**
- [API/Controllers/CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs):
  ```csharp
  public async Task<ActionResult<Category>> GetCategory(int id)
  {
      var category = await _context.Categories.FindAsync(id);
      if (category == null)
      {
          return NotFound();                 // 404
      }
      return category;                       // 200 + JSON
  }
  ...
  return CreatedAtAction("GetCategory", new { id = category.Id }, category);   // POST → 201
  ...
  return NoContent();                        // PUT, DELETE → 204
  ```
- Minimal API — [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs):
  ```csharp
  group.MapGet("/{id:int}", async Task<Results<Ok<Pizza>, NotFound>> (int id, AppDbContext db) => ...
      ? TypedResults.Ok(model)
      : TypedResults.NotFound();
  ```

**6. Что такое Minimal API?**

Способ описать конечные точки HTTP без классов-контроллеров: вызовом `MapGet`/`MapPost`/`MapPut`/`MapDelete` с делегатом (лямбдой), чьи параметры заполняются автоматически (из маршрута, строки запроса, тела, DI). Меньше кода и накладных расходов на обработку запроса, чем у контроллеров.

**Где в проекте:** все точки для пицц — [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs), например:
```csharp
group.MapGet("/{id:int}", async Task<Results<Ok<Pizza>, NotFound>> (int id, AppDbContext db) =>
{
    return await db.Pizzas.AsNoTracking()
        .Include(d => d.Category)
        .FirstOrDefaultAsync(model => model.Id == id)
        is Pizza model
            ? TypedResults.Ok(model)
            : TypedResults.NotFound();
})
.WithName("GetPizzaById");
```

**7. Как зарегистрировать конечную точку Minimal API?**

Вызвать у `WebApplication` (или `IEndpointRouteBuilder`) метод `MapGet/MapPost/MapPut/MapDelete(шаблон, делегат)`, при необходимости задать имя `.WithName(...)`. Чтобы не загромождать `Program.cs`, точки собирают в метод-расширение.

**Где в проекте:**
- метод-расширение — [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs):
  ```csharp
  public static class PizzaEndpoints
  {
      public static void MapPizzaEndpoints(this IEndpointRouteBuilder routes)
      {
          ...
          group.MapDelete("/{id:int}", async ... => { ... }).WithName("DeletePizza");
      }
  }
  ```
- вызов — [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs): `app.MapPizzaEndpoints();`

**8. Как зарегистрировать группу конечных точек?**

Методом `MapGroup(префикс)`: он возвращает группу, у которой точки регистрируются теми же `MapGet/MapPost…`, и все они получают общий префикс маршрута и общие настройки (`.WithTags(...)`, `.DisableAntiforgery()`, а в ЛР6 — `.RequireAuthorization()`).

**Где в проекте:** [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs):
```csharp
var group = routes.MapGroup("/api/Pizza")
                  .DisableAntiforgery();       // настройка сразу для всех точек группы (добавлена в ЛР5)

group.MapGet("/{id:int}", ...);                // отвечает по адресу /api/Pizza/{id}
```

---

## ЛР5. Razor Pages. Передача файлов

**Цель:** знакомство со сценарием построения приложений на страницах (Razor Pages) и с передачей файлов клиент → сервер → API.

**Итог:** в `UI` появилась админка — CRUD для пицц на Razor Pages в области `Admin` (https://localhost:7001/Admin). При создании и редактировании пиццы картинка загружается файлом, API сохраняет её в `wwwroot/Images` под случайным именем, а старый файл удаляет.

### Часть 1. Страницы администратора

**3.1. Постановка задачи** — страницы лежат в области (Area) `Admin`: папка [UI/Areas/Admin/Pages](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages). Ссылка «Администрирование» в меню (`_MenuPartial.cshtml`, ЛР2) уже вела на `asp-area="Admin" asp-page="/Index"`.

**3.2. Подготовка проекта** — для генератора временно добавлены пакеты EF Core (`Microsoft.EntityFrameworkCore`, `.Sqlite`, `.SqlServer`, `.Design`, `.Tools`, версии 9.0.20) и класс `UI/Data/TempDbContext.cs` с `DbSet<Pizza>` и `UseSqlite("")` в `OnConfiguring`. Реальная БД генератору не нужна — ему хватает модели. После генерации контекст и пакеты удалены (в проекте остался только `Microsoft.VisualStudio.Web.CodeGeneration.Design`).

**3.3. Создание страниц**
- Генерация (из папки `UI`):
  `dotnet aspnet-codegenerator razorpage -m WEB_453501_Kolesnikow.Domain.Entities.Pizza -dc WEB_453501_Kolesnikow.UI.Data.TempDbContext -udl -outDir Areas/Admin/Pages --referenceScriptLibraries`
  Созданы 5 страниц — `Index`, `Details`, `Create`, `Edit`, `Delete` (каждая: `.cshtml` — разметка, `.cshtml.cs` — модель страницы).
- В моделях всех страниц `TempDbContext` заменён на `IProductService` (и `ICategoryService` — для выпадающего списка категорий в `Create`/`Edit`). Пространство имён исправлено на `WEB_453501_Kolesnikow.UI.Areas.Admin.Pages`.
- В `Areas/Admin/Pages` скопированы `_ViewImports.cshtml` (using-и и подключение tag-helpers) и `_ViewStart.cshtml` (макет `_Layout`) — без них страницы области не видят файлов из `Views`.
- [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs): `builder.Services.AddRazorPages();` и `app.MapRazorPages();`.
- `Index`: список берётся постранично (`GetProductListAsync(null, pageNo)`), вместо имени файла — тег `<img>`, кнопки действий — иконки bootstrap-icons (`bi-pencil`, `bi-info-circle`, `bi-trash`), пагинация — Bootstrap *Pagination* с иконками `bi-chevron-left/right`.
- `Details`, `Edit`, `Delete` получают `id` сегментом маршрута: `@page "{id:int?}"` → `/Admin/Details/5`; `/Admin/Details/abc` → 404.
- `Create`/`Edit`: `[BindProperty] public Pizza Pizza`, категория — `<select asp-for="Pizza.CategoryId" asp-items="Model.Categories">` (`SelectList(категории, "Id", "Name")`). После успешного POST — `RedirectToPage("./Index")` (шаблон Post/Redirect/Get: обновление страницы не отправит форму повторно). В `Edit` скрытые поля `Id`, `ImageFilePath`, `MIME` — чтобы они не потерялись при сохранении.
- `Delete`: GET показывает пиццу, удаление — только POST (`OnPostAsync(int? id)`).
- Удалён лишний файл `Pages/_ValidationScriptsPartial.cshtml`, созданный генератором (такой уже есть в `Views/Shared`).

### Часть 2. Передача файлов

**4.1. Постановка задачи** — картинки хранятся в `API/wwwroot/Images`, имена случайные (`Guid`), при замене или удалении старый файл удаляется. Имя поля файла в HTTP-запросе — `file`, объект передаётся JSON-строкой в поле `pizza`.

**4.2. Use-case сохранения файла** — [API/Use-Cases/SaveImage.cs](WEB_453501_Kolesnikow.API/Use-Cases/SaveImage.cs): запрос `record SaveImage(IFormFile file) : IRequest<string>`, обработчик `SaveImageHandler(IWebHostEnvironment env, IHttpContextAccessor httpContextAccessor)`: путь `env.WebRootPath/Images` → имя `Guid.NewGuid() + расширение` → `file.CopyToAsync(FileStream)` → возвращает URL `{Scheme}://{Host}/Images/{имя}` (адрес хоста API из текущего запроса). В `API/Program.cs` добавлено `builder.Services.AddHttpContextAccessor();`.
Дополнительно [API/Use-Cases/DeleteImage.cs](WEB_453501_Kolesnikow.API/Use-Cases/DeleteImage.cs): запрос `record DeleteImage(string? imageUrl) : IRequest` — по URL находит файл в `wwwroot/Images` (`Path.GetFileName`) и удаляет его.

**4.3. Конечные точки API с файлом** — [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs):
- группа помечена `.DisableAntiforgery()` — конечные точки Minimal API, читающие форму, иначе требуют antiforgery-токен, которого у `HttpClient` из UI нет;
- `POST /api/Pizza` и `PUT /api/Pizza/{id}` принимают `[FromForm] string pizza` + `[FromForm] IFormFile? file` (тело `multipart/form-data`), объект — `JsonSerializer.Deserialize<Pizza>(pizza)`, файл — `mediator.Send(new SaveImage(file))`, `MIME = file.ContentType`;
- PUT теперь загружает пиццу (`FindAsync`) и меняет поля — нужно знать **старый** URL картинки, чтобы удалить файл (`DeleteImage`) перед сохранением нового;
- `DELETE` удаляет и запись, и файл картинки;
- Middleware статических файлов (`app.UseStaticFiles()`) подключён ещё в ЛР4 — через него новые картинки доступны клиенту;
- в `.http`-файле POST/PUT переделаны на `multipart/form-data` (видны `boundary` и части `Content-Disposition: form-data; name="pizza"` / `name="file"; filename="..."`).

**4.4. Доработка `ApiProductService`** — [UI/Services/ProductService/ApiProductService.cs](WEB_453501_Kolesnikow.UI/Services/ProductService/ApiProductService.cs): `CreateProductAsync`/`UpdateProductAsync` собирают тело методом `CreateMultipartContent`: `MultipartFormDataContent`, файл — `StreamContent(formFile.OpenReadStream())` в поле `file` (с MIME-типом файла), объект — `StringContent(JsonSerializer.Serialize(product))` в поле `pizza`.

**4.5. Страницы Create/Edit** — в моделях `[BindProperty] public IFormFile? Image { get; set; }`, передаётся в сервис: `CreateProductAsync(Pizza, Image)`, `UpdateProductAsync(Pizza.Id, Pizza, Image)`. В разметке — форма `<form method="post" enctype="multipart/form-data">` и `<input asp-for="Image" class="form-control" accept="image/*" />` (`asp-for` у свойства `IFormFile` сам ставит `type="file"`).

**4.5.1. (необязательно)** — на странице `Edit` над полем выбора файла выводится текущая картинка.

### Проверка (выполнена)

| Действие | Результат |
|---|---|
| `/Admin` | таблица из 3 пицц с картинками, пагинация `‹ 1 2 ›` с подсветкой текущей страницы |
| `/Admin?pageNo=2` | пиццы 4–6 |
| `/Admin/Details/1` | описание, цена, категория «Классические» |
| Создать пиццу с картинкой | переход на `/Admin`; в `API/wwwroot/Images` появился файл `<guid>.jpg`, `MIME = image/jpeg` |
| Изменить с новой картинкой | старый `<guid>.jpg` удалён, появился новый |
| Изменить без картинки | картинка не изменилась |
| Удалить | пицца пропала (API → 404), её файл удалён; исходные 1.jpg…6.jpg на месте |
| `POST /api/Pizza` multipart напрямую | 201 Created, URL картинки `https://localhost:7002/Images/<guid>.jpg` |

### Ответы на контрольные вопросы

**1. Чем Razor Pages отличается от MVC?**

В MVC запрос обрабатывает действие контроллера, которое выбирает представление и передаёт ему модель; код разложен по типам (контроллеры, представления, модели — в разных папках). В Razor Pages каждая страница — самостоятельная конечная точка: разметка `.cshtml` + класс модели страницы `.cshtml.cs`, контроллера нет; код организован по функциям (всё для одной страницы — рядом). Razor Pages удобнее для страничных интерфейсов (формы, CRUD), MVC — для API и сложной логики с частичными ответами.

**Где в проекте:** в одном приложении `UI` есть оба подхода:

| | MVC — каталог | Razor Pages — админка |
|---|---|---|
| обработка запроса | [Controllers/Product.cs](WEB_453501_Kolesnikow.UI/Controllers/Product.cs) — `ProductController.Index` | [Areas/Admin/Pages/Index.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Index.cshtml.cs) — `IndexModel.OnGetAsync` |
| разметка | [Views/Product/Index.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/Index.cshtml) | [Areas/Admin/Pages/Index.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Index.cshtml) |
| адрес | `/Product` (маршрут `{controller}/{action}`) | `/Admin` (путь файла) |
| регистрация в [Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) | `AddControllersWithViews()` + `MapControllerRoute(...)` | `AddRazorPages()` + `MapRazorPages()` |

**2. Что такое модель страницы Razor?**

Класс-наследник `PageModel` в файле `.cshtml.cs`. Содержит обработчики запросов (`OnGet`, `OnPostAsync` …) и свойства с данными для разметки; зависимости получает через конструктор (DI). Аналог контроллера + view-model для одной страницы.

**Где в проекте:** [UI/Areas/Admin/Pages/Create.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml.cs):
```csharp
public class CreateModel : PageModel
{
    public CreateModel(IProductService productService, ICategoryService categoryService) { ... }   // DI

    [BindProperty] public Pizza Pizza { get; set; } = new();    // данные формы
    [BindProperty] public IFormFile? Image { get; set; }        // файл
    public SelectList Categories { get; set; } = default!;      // данные для разметки

    public async Task<IActionResult> OnGetAsync() { ... }       // обработчик GET
    public async Task<IActionResult> OnPostAsync() { ... }      // обработчик POST
}
```

**3. Как обрабатываются запросы к страницам Razor?**

Маршрутизация сопоставляет URL со страницей (по пути файла и шаблону из `@page`), затем вызывается обработчик по HTTP-методу: `OnGet`/`OnGetAsync`, `OnPost`/`OnPostAsync` и т.д. Именованные обработчики (`OnPostDelete`) выбираются параметром `?handler=Delete` (`asp-page-handler`). Обработчик возвращает `Page()` (отрисовать страницу), `RedirectToPage(...)`, `NotFound()` и т.п.

**Где в проекте:** [UI/Areas/Admin/Pages/Edit.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Edit.cshtml.cs):
```csharp
// GET /Admin/Edit/5 — показать форму
public async Task<IActionResult> OnGetAsync(int? id)
{
    ...
    if (!response.Successfull || response.Data == null)
        return NotFound();
    Pizza = response.Data;
    await LoadCategoriesAsync();
    return Page();
}

// POST /Admin/Edit/5 — отправка формы
public async Task<IActionResult> OnPostAsync()
{
    ...
    await _productService.UpdateProductAsync(Pizza.Id, Pizza, Image);
    return RedirectToPage("./Index");
}
```

**4. Как в разметке страницы получить доступ к свойствам модели страницы?**

Директива `@model` задаёт тип модели, обращение — `@Model.Свойство`; tag-helpers `asp-for="..."` принимают выражение относительно `Model`.

**Где в проекте:**
- [UI/Areas/Admin/Pages/Index.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Index.cshtml):
  ```razor
  @model WEB_453501_Kolesnikow.UI.Areas.Admin.Pages.IndexModel
  ...
  @foreach (var item in Model.Pizza) { ... }
  <li class="page-item @(i == Model.CurrentPage ? "active" : "")">
  ```
- [UI/Areas/Admin/Pages/Create.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml):
  ```razor
  <input asp-for="Pizza.Name" class="form-control" />
  <select asp-for="Pizza.CategoryId" asp-items="Model.Categories" class="form-select"></select>
  ```

**5. Как привязать данные формы к модели страницы?**

Атрибутом `[BindProperty]` над свойством модели (работает для POST; для GET — `[BindProperty(SupportsGet = true)]`) или параметрами метода-обработчика. Имена полей формы должны совпадать с путём к свойству — `asp-for="Pizza.Name"` генерирует `name="Pizza.Name"`.

**Где в проекте:**
- свойства — [Create.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml.cs) и [Edit.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Edit.cshtml.cs):
  ```csharp
  [BindProperty]
  public Pizza Pizza { get; set; } = new();      // поля name="Pizza.Name", "Pizza.Price", ...

  [BindProperty]
  public IFormFile? Image { get; set; }          // поле name="Image" (type="file")
  ```
- параметр обработчика — [Delete.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Delete.cshtml.cs): `public async Task<IActionResult> OnPostAsync(int? id)` — `id` из маршрута `/Admin/Delete/5`.

**6. Для чего `@page`?**

Делает файл `.cshtml` страницей Razor — конечной точкой, которая обрабатывает запросы сама, без контроллера. Должна быть первой директивой; может содержать шаблон маршрута.

**Где в проекте:** первая строка всех страниц в [UI/Areas/Admin/Pages](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages): `@page` в `Index.cshtml` и `Create.cshtml`, `@page "{id:int?}"` в `Details.cshtml`, `Edit.cshtml`, `Delete.cshtml`. Для сравнения, у представлений MVC в `Views/` директивы `@page` нет — их вызывает контроллер.

**7. Какой URL у страницы `Register`?**

Если шаблон в `@page` не задан, URL = путь файла относительно папки `Pages` без расширения: `Pages/Register.cshtml` → `/Register`. В области добавляется её имя: `Areas/Admin/Pages/Register.cshtml` → `/Admin/Register`. Для `Index` дополнительно работает адрес папки.

**Где в проекте:**

| Файл | URL |
|---|---|
| `UI/Areas/Admin/Pages/Index.cshtml` | `/Admin`, `/Admin/Index` |
| `UI/Areas/Admin/Pages/Create.cshtml` | `/Admin/Create` |
| `UI/Areas/Admin/Pages/Edit.cshtml` (`@page "{id:int?}"`) | `/Admin/Edit/5` |

**8. Чем отличается адрес `/index` от `./index`?**

`/Index` — абсолютное имя страницы, от корня папки `Pages` (в области — от `Areas/<Area>/Pages`). `./Index` — относительное: страница `Index` в той же папке, что и текущая страница (`../Index` — на уровень выше).

**Где в проекте:**
- относительное — на страницах админки, например [Create.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml.cs): `return RedirectToPage("./Index");` и [Create.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml): `<a asp-page="./Index" ...>К списку</a>`;
- абсолютное — в меню [UI/Views/Shared/_MenuPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_MenuPartial.cshtml) (файл вне папки `Pages`, относительный путь там не имеет смысла):
  ```razor
  <a class="nav-item nav-link ..." asp-area="Admin" asp-page="/Index">Администрирование</a>
  ```

**9. Как указать дополнительный сегмент маршрута (например `id`)?**

Шаблоном в директиве: `@page "{id:int?}"` — сегмент `id`, ограничение «целое число» (`:int`), необязательный (`?`). Значение приходит в параметр обработчика; ссылка `asp-route-id="5"` строит `/Admin/Edit/5`. Значения, которых нет в шаблоне, уходят в строку запроса.

**Где в проекте:**
- шаблон — [UI/Areas/Admin/Pages/Details.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Details.cshtml): `@page "{id:int?}"`;
- параметр — [Details.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Details.cshtml.cs): `public async Task<IActionResult> OnGetAsync(int? id)`;
- ссылки — [Index.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Index.cshtml):
  ```razor
  <a asp-page="./Details" asp-route-id="@item.Id" ...>        → /Admin/Details/5     (id — в шаблоне)
  <a class="page-link" asp-page="./Index" asp-route-pageNo="@i"> → /Admin?pageNo=2    (pageNo — нет в шаблоне)
  ```

**10. Какой Middleware передаёт статический контент клиенту?**

Middleware статических файлов — `app.UseStaticFiles()` (в .NET 9 также `app.MapStaticAssets()`); отдаёт файлы из `wwwroot`.

**Где в проекте:**
- [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs) — картинки пицц (`https://localhost:7002/Images/...`):
  ```csharp
  // отдавать файлы из wwwroot (изображения)
  app.UseStaticFiles();
  ```
- [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) — CSS, JS, иконки: `app.MapStaticAssets();`

**11. Какой интерфейс реализует объект файла при передаче клиент → сервер?**

`IFormFile` (`Microsoft.AspNetCore.Http`).

**Где в проекте:** везде, где файл принимается:
- UI — [Create.cshtml.cs](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml.cs): `[BindProperty] public IFormFile? Image { get; set; }`
- API — [PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs): `[FromForm] IFormFile? file`
- use-case — [SaveImage.cs](WEB_453501_Kolesnikow.API/Use-Cases/SaveImage.cs): `public sealed record SaveImage(IFormFile file) : IRequest<string>;`

**12. Что такое поставщики файлов (FileProvider)?**

Абстракция над файловой системой — интерфейс `IFileProvider` (`GetFileInfo`, `GetDirectoryContents`, `Watch` — подписка на изменения). Реализации: `PhysicalFileProvider` (папка на диске, доступ ограничен этой папкой), `ManifestEmbeddedFileProvider` (файлы внутри сборки), `CompositeFileProvider` (объединение нескольких). Через них работают Middleware статических файлов и движок Razor; готовый поставщик для `wwwroot` — `IWebHostEnvironment.WebRootFileProvider`.

**Где в проекте:** явно в коде не используется. Неявно — `app.UseStaticFiles()` в [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs) по умолчанию берёт `PhysicalFileProvider` над папкой `API/wwwroot` — поэтому файл `wwwroot/Images/1.jpg` доступен по адресу `/Images/1.jpg`. Другую папку можно было бы подключить так:
```csharp
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(Path.Combine(builder.Environment.ContentRootPath, "Uploads")),
    RequestPath = "/uploads"
});
```

**13. Как получить путь к `wwwroot`?**

Свойство `IWebHostEnvironment.WebRootPath` (сервис `IWebHostEnvironment` внедряется через DI); путь к корню проекта — `ContentRootPath`.

**Где в проекте:** [API/Use-Cases/SaveImage.cs](WEB_453501_Kolesnikow.API/Use-Cases/SaveImage.cs) (и так же в [DeleteImage.cs](WEB_453501_Kolesnikow.API/Use-Cases/DeleteImage.cs)):
```csharp
public class SaveImageHandler(IWebHostEnvironment env, IHttpContextAccessor httpContextAccessor) ...
{
    ...
    var imagesPath = Path.Combine(env.WebRootPath, "Images");
```

**14. В каком виде представлен переданный файл на сервере?**

Объектом, реализующим `IFormFile`: `FileName` (исходное имя), `Name` (имя поля формы), `ContentType` (MIME-тип), `Length` (размер), `OpenReadStream()` (поток для чтения), `CopyToAsync(stream)` (скопировать в другой поток). Содержимое — поток; небольшие файлы буферизуются в памяти, большие — во временном файле на диске.

**Где в проекте:**
- запись на диск — [SaveImage.cs](WEB_453501_Kolesnikow.API/Use-Cases/SaveImage.cs):
  ```csharp
  var extension = Path.GetExtension(request.file.FileName);              // FileName
  ...
  await request.file.CopyToAsync(stream, cancellationToken);            // CopyToAsync
  ```
- MIME-тип — [PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs): `newPizza.MIME = file.ContentType;`
- чтение потока и передача дальше — [ApiProductService.cs](WEB_453501_Kolesnikow.UI/Services/ProductService/ApiProductService.cs):
  ```csharp
  var streamContent = new StreamContent(formFile.OpenReadStream());     // OpenReadStream
  content.Add(streamContent, "file", formFile.FileName);
  ```

**15. Какой атрибут `<form>` нужен для передачи файлов?**

`enctype="multipart/form-data"` (вместе с `method="post"`). Без него браузер кодирует форму как `application/x-www-form-urlencoded` и отправляет только имя файла, а не содержимое.

**Где в проекте:** [Create.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml) и [Edit.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Edit.cshtml):
```razor
<form method="post" enctype="multipart/form-data">
    ...
    <input asp-for="Image" class="form-control" accept="image/*" />
```
Для сравнения, в [Delete.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Delete.cshtml) файла нет — там обычная `<form method="post">`.

---

## ЛР6. Аутентификация и авторизация

**Цель:** знакомство с механизмом аутентификации и авторизации в ASP.NET Core; аутентификация на удалённом сервере, ограничение доступа к API.

**Итог:** сервер аутентификации **Keycloak** (Docker, http://localhost:8080) хранит пользователей и роли. Сайт `UI` входит через Keycloak (OpenID Connect) и запоминает пользователя в cookie; админка доступна только роли `POWER-USER`. `API` проверяет JWT-токен: список пицц открыт всем, остальное — только `POWER-USER`. Есть своя страница регистрации с аватаром, в шапке — имя и аватар вошедшего.

**Учётные записи** (пароль у всех `123456`):

| Где | Логин | Роль |
|---|---|---|
| консоль Keycloak http://localhost:8080 (realm `master`) | `admin` | администратор Keycloak |
| сайт | `admin@example.com` | `POWER-USER` — админка доступна |
| сайт | `user@example.com` | обычный пользователь |
| сайт | `new@example.com` | зарегистрирован через страницу регистрации, с аватаром |

### Что сделано по пунктам задания

**3.1–3.2. Сервер аутентификации и термины.** Участвуют три программы:

| Участник | Как проверяет пользователя |
|---|---|
| Keycloak (8080) | хранит пользователей/роли, показывает страницу входа, выдаёт токены |
| `UI` (7001) | OpenID Connect: отправляет браузер на страницу входа Keycloak, получает токены, хранит вход в **cookie** |
| `API` (7002) | JWT Bearer: ждёт заголовок `Authorization: Bearer <access_token>` и проверяет подпись токена ключами Keycloak |

**Realm** — изолированное пространство (пользователи, роли, клиенты), у нас `Kolesnikow`. **Client** — приложение, которое пользуется Keycloak, у нас `KolesnikowUiClient`. **Scope** — область доступа, которую запрашивает клиент (`openid`, `roles`, …).

**4.3. Keycloak в Docker** — в [API/DB/compose.yaml](WEB_453501_Kolesnikow.API/DB/compose.yaml) добавлен сервис `keycloak` (образ `quay.io/keycloak/keycloak:26.0.4`, `command: start-dev`, БД — тот же контейнер `postgres`, база `keycloak`, порт 8080); логин/пароль администратора — `KEYCLOAK_ADMIN`/`KEYCLOAK_ADMIN_PASSWORD` в `postgres.env`. База `keycloak` создана вручную (`docker exec postgres psql -U admin -d postgres -c "CREATE DATABASE keycloak;"`), т.к. том `postgres_data` уже существовал. Запуск: `docker compose --env-file postgres.env up -d`.

**4.4. Настройка Keycloak** — выполнена скриптом [API/DB/keycloak-setup.sh](WEB_453501_Kolesnikow.API/DB/keycloak-setup.sh) через CLI `kcadm.sh` (это те же действия, что в консоли администратора; скрипт можно повторить на другой машине):

| Пункт | Что настроено | Где это в консоли Keycloak |
|---|---|---|
| 4.4.1 | realm `Kolesnikow`, email как имя пользователя, Access Token Lifespan = 1 час; в User profile удалены `firstName`/`lastName`, добавлен необязательный атрибут `avatar` | *Create realm*; *Realm settings → Login / Tokens / User profile* |
| 4.4.2 | клиент `KolesnikowUiClient` (OpenID Connect, Client authentication, Authorization, Standard/Direct access/Implicit flow, Service accounts), адреса `https://localhost:7001/*`; Client Secret | *Clients → Create client*; вкладка *Credentials* |
| 4.4.2 | служебной учётке клиента — роль `realm-management: manage-users` (для регистрации пользователей) | *Clients → … → Service accounts roles* |
| 4.4.3 | маппер `avatar` (User Attribute → claim `avatar`, в ID и access token) | *Clients → … → Client scopes → KolesnikowUiClient-dedicated* |
| 4.4.4 | роль клиента `POWER-USER` | *Clients → … → Roles* |
| 4.4.5 | маппер `role` (User Client Role, Multivalued, claim `role`, в ID и access token) | *Client scopes → roles → Mappers* |
| 4.4.6 | пользователи `user@example.com`, `admin@example.com` (email подтверждён, пароль не временный), администратору — роль `POWER-USER` | *Users*; вкладки *Credentials*, *Role mapping* |

**4.5. Проверка Keycloak** — `POST http://localhost:8080/realms/Kolesnikow/protocol/openid-connect/token` (тело `x-www-form-urlencoded`):
- `grant_type=password` + `username`/`password` → токен **пользователя**; у `admin@example.com` в нём `"role": ["POWER-USER"]`, `"aud": "account"`, `"preferred_username": "admin@example.com"`; у `user@example.com` роли нет;
- `grant_type=client_credentials` → токен **клиента** (`preferred_username: service-account-kolesnikowuiclient`, `aud` включает `realm-management`);
- неверный `client_secret` → 401.

**4.6. Настройка API**
- Пакет `Microsoft.AspNetCore.Authentication.JwtBearer` 9.0.20; в `appsettings.json` — `"AuthServer": { "Host": "http://localhost:8080", "Realm": "Kolesnikow" }`, класс [API/Models/AuthServerData.cs](WEB_453501_Kolesnikow.API/Models/AuthServerData.cs).
- [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs): `AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(...)` (`MetadataAddress`, `Authority`, `Audience = "account"`, `RequireHttpsMetadata = false`), политика `"admin"` → `RequireRole("POWER-USER")`, `app.UseAuthentication()` перед `app.UseAuthorization()`.
- [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs): на группе `.RequireAuthorization("admin")`, на списке `.AllowAnonymous()`. Контроллер категорий не защищён (категории читаются анонимно).

**4.6.1. Проверка API:**

| Запрос | Результат |
|---|---|
| `GET /api/Pizza` без токена | 200 (список открыт) |
| `GET /api/Pizza/1` без токена | **401** Unauthorized |
| `GET /api/Pizza/1` с токеном `admin@example.com` | 200 |
| `GET /api/Pizza/1` с токеном `user@example.com` | **403** Forbidden (вошёл, но нет роли) |
| `POST /api/Pizza` без токена | 401 |

**4.7. Настройка UI**
- Пакеты `Microsoft.AspNetCore.Authentication.OpenIdConnect` и `...JwtBearer` 9.0.20; в `appsettings.json` — секция `Keycloak` (`Host`, `Realm`, `ClientId`, `ClientSecret`), класс [UI/HelperClasses/KeycloakData.cs](WEB_453501_Kolesnikow.UI/HelperClasses/KeycloakData.cs), `builder.Services.Configure<KeycloakData>(...)` в `HostingExtensions`.
- [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs): `AddAuthentication` (`DefaultScheme` = Cookie, `DefaultChallengeScheme` = `"keycloak"`) + `.AddCookie()` + `.AddOpenIdConnect("keycloak", ...)` (`ResponseType = Code`, `SaveTokens = true`, `NameClaimType = "preferred_username"`), политика `"admin"`, `UseAuthentication()`/`UseAuthorization()`, `app.MapRazorPages().RequireAuthorization("admin")` — вся админка только для `POWER-USER`.
- Добавлен `app.UseStaticFiles()`: `MapStaticAssets()` в .NET 9 отдаёт только файлы, известные при сборке, а аватары появляются во время работы.

**4.7.1. Получение токена** — [UI/Services/Authentication/ITokenAccessor.cs](WEB_453501_Kolesnikow.UI/Services/Authentication/ITokenAccessor.cs) и [KeycloakTokenAccessor.cs](WEB_453501_Kolesnikow.UI/Services/Authentication/KeycloakTokenAccessor.cs): `SetAuthorizationHeaderAsync(httpClient, isClient)` кладёт в заголовок `Authorization: bearer ...` либо токен **пользователя** (`HttpContext.GetTokenAsync("keycloak", "access_token")` — сохранён в cookie благодаря `SaveTokens`), либо токен **клиента** (`POST` на token endpoint с `grant_type=client_credentials`). Регистрация: `AddHttpClient<ITokenAccessor, KeycloakTokenAccessor>()`, плюс `AddHttpContextAccessor()`.

**4.7.2–4.7.3. Токен в запросах к API** — в [ApiProductService.cs](WEB_453501_Kolesnikow.UI/Services/ProductService/ApiProductService.cs) внедрён `ITokenAccessor`; перед `GetProductByIdAsync`, `CreateProductAsync`, `UpdateProductAsync`, `DeleteProductAsync` вызывается `SetAuthorizationHeaderAsync(_httpClient, false)`. `GetProductListAsync` токен не передаёт: список открыт анонимно, и каталог должен работать у гостей. Проверено: вход как `admin@example.com` → в админке создание/удаление пиццы проходит через API.

**4.8. Регистрация**
- [UI/Services/FileService/LocalFileService.cs](WEB_453501_Kolesnikow.UI/Services/FileService/LocalFileService.cs) (`IFileService`) — сохраняет аватар в `UI/wwwroot/Images/<guid>.<расш.>`, возвращает `/Images/<имя>`.
- [UI/Models/RegisterUserViewModel.cs](WEB_453501_Kolesnikow.UI/Models/RegisterUserViewModel.cs) — `Email`, `Password`, `ConfirmPassword` (`[Compare]`), `IFormFile? Avatar`.
- [UI/Controllers/AccountController.cs](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs) — `Register` (GET — форма, POST — токен клиента → сохранить аватар → `POST {Host}/admin/realms/Kolesnikow/users` с JSON `CreateUserModel` в camelCase). Без файла аватар по умолчанию — иконка `/lib/bootstrap-icons/icons/person-circle.svg`.
- [UI/Views/Account/Register.cshtml](WEB_453501_Kolesnikow.UI/Views/Account/Register.cshtml) — форма `enctype="multipart/form-data"`.
- Проверено: регистрация `new@example.com` с картинкой → переход на главную, файл в `UI/wwwroot/Images`, пользователь в Keycloak с атрибутом `avatar`; после входа его аватар — в шапке.

**4.9. Меню**
- `AccountController`: `Login` — `HttpContext.ChallengeAsync("keycloak", …)` (переход на страницу входа Keycloak); `Logout` (только POST) — `SignOutAsync` для Cookie и для `"keycloak"` (завершает и сессию Keycloak); `AccessDenied` — сюда попадает вошедший без роли.
- [UI/Views/Shared/_UserInfoPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_UserInfoPartial.cshtml): гость видит «Войти»/«Зарегистрироваться»; вошедший — корзину, имя (claim `preferred_username`), аватар (claim `avatar`) и Logout (форма POST).

**Проверка (выполнена):** гость → «Войти»/«Зарегистрироваться», `/Admin` перенаправляет на Keycloak; `user@example.com` → в шапке его email, `/Admin` → «Доступ запрещён»; `admin@example.com` → админка работает; Logout → снова гостевое меню, повторный вход снова спрашивает пароль.

### Ответы на контрольные вопросы

**1. Какой механизм аутентификации имеет встроенную поддержку в ASP.NET Core?**

Аутентификация на основе **cookie**: Middleware аутентификации сериализует данные пользователя в зашифрованную cookie, браузер присылает её с каждым запросом, а Middleware восстанавливает из неё `HttpContext.User`. Работа идёт через сервис `IAuthenticationService` и «схемы» (Cookie, JwtBearer, OpenIdConnect…).

**Где в проекте:** [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) — cookie-схема по умолчанию; после входа через Keycloak пользователь хранится в cookie `.AspNetCore.Cookies`:
```csharp
.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;   // кто пользователь — по cookie
    options.DefaultChallengeScheme = "keycloak";                                 // как войти — через Keycloak
})
.AddCookie()
```

**2. Что описывают `ClaimsPrincipal` и `ClaimsIdentity`?**

`ClaimsIdentity` — одно «удостоверение» пользователя: набор утверждений (`Claim` — пара тип/значение: email, роль, id…) и тип аутентификации. `ClaimsPrincipal` — сам пользователь: содержит одну или несколько `ClaimsIdentity`; методы `FindFirst`, `FindAll`, `HasClaim`, `IsInRole`, свойство `Identity`. Текущий пользователь — `HttpContext.User` (тип `ClaimsPrincipal`).

**Где в проекте:** [UI/Views/Shared/_UserInfoPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_UserInfoPartial.cshtml) — `User` здесь `ClaimsPrincipal`, его claims пришли из ID-токена Keycloak:
```razor
@if (User.Identity?.IsAuthenticated == true)          @* Identity — ClaimsIdentity *@
{
    var name = User.Claims.FirstOrDefault(c => c.Type.Equals("preferred_username", ...))?.Value;
```
и [KeycloakTokenAccessor.cs](WEB_453501_Kolesnikow.UI/Services/Authentication/KeycloakTokenAccessor.cs): `var authSession = await context.AuthenticateAsync("keycloak"); if (authSession?.Principal == null) ...`

**3. Как подключить Middleware аутентификации/авторизации?**

Зарегистрировать сервисы (`AddAuthentication(...)` со схемами, `AddAuthorization(...)` с политиками) и добавить в конвейер `app.UseAuthentication()` и затем `app.UseAuthorization()` — после `UseRouting()` и до сопоставления конечных точек. Порядок важен: сначала определить пользователя, потом проверять его права.

**Где в проекте:** в обоих `Program.cs`:
- [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs):
  ```csharp
  app.UseRouting();
  // ЛР6: до UseAuthorization
  app.UseAuthentication();
  app.UseAuthorization();
  ```
- [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs):
  ```csharp
  // ЛР6: сначала аутентификация (кто это), потом авторизация (что можно)
  app.UseAuthentication();
  app.UseAuthorization();
  ```

**4. Приведите пример использования `HttpContext.User`.**

`HttpContext.User` — текущий пользователь; заполняется Middleware аутентификации. В контроллере доступен как `User`, в представлении — тоже `User`.

**Где в проекте:**
- UI: шапка сайта [_UserInfoPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_UserInfoPartial.cshtml) читает из `User` имя и аватар (см. вопросы 2, 6);
- API: `JwtBearer` заполняет `HttpContext.User` из токена, а политика `"admin"` ([API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs)) проверяет у него роль: `opt.AddPolicy("admin", p => p.RequireRole("POWER-USER"));` — поэтому `GET /api/Pizza/1` с токеном `user@example.com` даёт 403.

**5. Как проверить, что пользователь прошёл аутентификацию?**

`User.Identity?.IsAuthenticated == true`. Декларативно — атрибут `[Authorize]` / метод `.RequireAuthorization()`: неаутентифицированный получит 401 (API) или переход на вход (сайт).

**Где в проекте:**
- явная проверка — [_UserInfoPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_UserInfoPartial.cshtml): `@if (User.Identity?.IsAuthenticated == true) { ... } else { Войти / Зарегистрироваться }`;
- декларативно — [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs): `app.MapRazorPages().RequireAuthorization("admin")` (гость отправляется на страницу входа Keycloak).

**6. Как получить значение Claim?**

`User.FindFirst("тип")?.Value` или `User.Claims.FirstOrDefault(c => c.Type == "тип")?.Value`.

**Где в проекте:** [_UserInfoPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_UserInfoPartial.cshtml):
```csharp
var avatar = User
    .Claims
    .FirstOrDefault(c => c.Type.Equals("avatar", StringComparison.OrdinalIgnoreCase))?
    .Value ?? "/lib/bootstrap-icons/icons/person-circle.svg";
```
Claim `avatar` попадает в токен благодаря мапперу из п. 4.4.3 ([keycloak-setup.sh](WEB_453501_Kolesnikow.API/DB/keycloak-setup.sh)).

**7. Как получить Id аутентифицированного пользователя?**

Из claim `ClaimTypes.NameIdentifier`: `User.FindFirst(ClaimTypes.NameIdentifier)?.Value` (в Identity — `_userManager.GetUserId(User)`). В токене Keycloak id пользователя — claim `sub`; обработчик OpenIdConnect/JwtBearer по умолчанию переименовывает его в `ClaimTypes.NameIdentifier`.

**Где в проекте:** в коде id пока не нужен. Если понадобится (например, для корзины пользователя), в любом контроллере UI достаточно:
```csharp
var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;   // id пользователя в Keycloak (claim "sub")
```
Id видны в консоли Keycloak: *Users → admin@example.com → ID*.

**8. Как разрешить доступ к ресурсу только для пользователей с ролью «manager»?**

Атрибутом `[Authorize(Roles = "manager")]` или политикой: `opt.AddPolicy("managers", p => p.RequireRole("manager"))` + `[Authorize(Policy = "managers")]` / `.RequireAuthorization("managers")`.

**Где в проекте:** то же самое для роли `POWER-USER`:
- политика — в [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) и [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs): `opt.AddPolicy("admin", p => p.RequireRole("POWER-USER"));`
- применение в UI: `app.MapRazorPages().RequireAuthorization("admin");`
- применение в API — [PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs):
  ```csharp
  var group = routes.MapGroup("/api/Pizza")
                    .DisableAntiforgery()
                    .RequireAuthorization("admin");   // все точки группы — только POWER-USER
  group.MapGet("/{category?}", ...).AllowAnonymous(); // кроме списка
  ```
- роль попадает в токен как claim `role` (маппер п. 4.4.5), библиотека превращает его в `ClaimTypes.Role`, который и проверяет `RequireRole`.

**9. Как создать политику авторизации на основе Claim?**

`builder.Services.AddAuthorization(o => o.AddPolicy("имя", p => p.RequireClaim("тип", "значение1", "значение2")));` и применить `[Authorize(Policy = "имя")]`.

**Где в проекте:** политика `"admin"` — частный случай: `RequireRole` проверяет claim типа «роль». Политика по произвольному claim в этом проекте выглядела бы так (например, пускать только пользователей с аватаром):
```csharp
builder.Services.AddAuthorization(opt =>
{
    opt.AddPolicy("admin", p => p.RequireRole("POWER-USER"));   // уже есть
    opt.AddPolicy("withAvatar", p => p.RequireClaim("avatar"));  // пример: claim из маппера п. 4.4.3
});
```

**10. Как создать куки аутентификации через `HttpContext`?**

Собрать claims → `ClaimsIdentity` (со схемой Cookie) → `ClaimsPrincipal` → `await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal)`. Удалить — `HttpContext.SignOutAsync(...)`.

**Где в проекте:** cookie создаёт сам обработчик OpenIdConnect: после входа в Keycloak он получает токены и вызывает `SignInAsync` для схемы по умолчанию (Cookie). Явно в коде — удаление cookie в [AccountController.cs](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs):
```csharp
[HttpPost]
public async Task Logout()
{
    await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);   // удалить cookie сайта
    await HttpContext.SignOutAsync("keycloak", ...);                                      // выйти из Keycloak
}
```

**11. Как подключить `Microsoft.AspNetCore.Identity`?**

Пакет `Microsoft.AspNetCore.Identity.EntityFrameworkCore`; контекст БД наследуется от `IdentityDbContext`; регистрация `builder.Services.AddDefaultIdentity<IdentityUser>().AddRoles<IdentityRole>().AddEntityFrameworkStores<AppDbContext>();` и `app.UseAuthentication()`.

**Где в проекте:** Identity не используется — по заданию пользователей хранит внешний сервер Keycloak (в базе `keycloak` того же PostgreSQL). Роль «подключения» играет регистрация Cookie + OpenIdConnect в [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs).

**12. Как создать пользователя через Identity?**

`var result = await _userManager.CreateAsync(new IdentityUser { UserName = email, Email = email }, password);` (`UserManager<TUser>` внедряется через DI).

**Где в проекте:** аналог — регистрация через Admin API Keycloak в [AccountController.cs](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs):
```csharp
var requestUri = $"{options.Value.Host}/admin/realms/{options.Value.Realm}/users";
var response = await httpClient.PostAsync(requestUri, content);   // content — JSON CreateUserModel
```

**13. Как выполнить вход через Identity?**

`await _signInManager.PasswordSignInAsync(userName, password, isPersistent: false, lockoutOnFailure: false);` или `await _signInManager.SignInAsync(user, isPersistent: false);` — `SignInManager` проверяет пароль и создаёт cookie.

**Где в проекте:** аналог — вход через Keycloak в [AccountController.cs](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs) (пароль проверяет Keycloak на своей странице):
```csharp
public async Task Login()
{
    await HttpContext.ChallengeAsync("keycloak",
        new AuthenticationProperties { RedirectUri = Url.Action("Index", "Home") });
}
```

**14. Как добавить Claim пользователю через Identity?**

`await _userManager.AddClaimAsync(user, new Claim("тип", "значение"));`

**Где в проекте:** аналог — атрибут пользователя Keycloak, который маппер превращает в claim. При регистрации ([AccountController.cs](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs)):
```csharp
newUser.Attributes.Add("avatar", avatarUrl);   // атрибут avatar → claim "avatar" в токене (маппер п. 4.4.3)
```

**15. Какой интерфейс используется для доступа к хранилищу пользователей?**

`IUserStore<TUser>` (для ролей — `IRoleStore<TRole>`); готовая реализация на EF Core — `UserStore` из `Microsoft.AspNetCore.Identity.EntityFrameworkCore`. `UserManager` работает с хранилищем через этот интерфейс.

**Где в проекте:** не используется — хранилище пользователей находится в Keycloak, приложение обращается к нему по HTTP (Admin API) через `HttpClient` и `ITokenAccessor`.

---

## ЛР7. Вспомогательные классы тэгов (Tag Helpers), Ajax

**Цель:** знакомство с tag helper'ами и с асинхронными HTTP-запросами.

**Итог:** пейджер — собственный tag helper `<pager>`, общий для каталога и админки. В каталоге страницы переключаются через Ajax: сервер возвращает только список и пейджер, страница не перезагружается.

### Что сделано по пунктам задания

**3.1. Задание 1. Tag helper пейджера**
- [UI/TagHelpers/PagerTagHelper.cs](WEB_453501_Kolesnikow.UI/TagHelpers/PagerTagHelper.cs) — класс `PagerTagHelper : TagHelper` с `[HtmlTargetElement("pager")]`; атрибуты `current-page`, `total-pages`, `category`, `admin` связываются со свойствами `CurrentPage`, `TotalPages`, `Category`, `Admin`.
- В `Process` тег `<pager>` заменяется на `<nav>` (`output.TagName = "nav"`), разметка `<ul class="pagination">` / `<li class="page-item">` / `<a class="page-link">` собирается **методами `TagBuilder`** (`AddCssClass`, `MergeAttribute`, `InnerHtml.AppendHtml`); кнопки «‹ ›» — иконки `bi-chevron-left/right`, текущая страница — `active`, крайние — `disabled`; если страница одна — `output.SuppressOutput()`.
- Адреса — через внедрённый `LinkGenerator`: `GetPathByAction("Index", "Product", new { category, pageNo })` для каталога и `GetPathByPage(httpContext, "/Index", new { area = "Admin", pageNo })` для админки (`admin="true"`); `HttpContext` — из `IHttpContextAccessor` (зарегистрирован в ЛР6).
- Регистрация: `@addTagHelper *, WEB_453501_Kolesnikow.UI` в [Views/_ViewImports.cshtml](WEB_453501_Kolesnikow.UI/Views/_ViewImports.cshtml) и в [Areas/Admin/Pages/_ViewImports.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/_ViewImports.cshtml).
- Использование: в каталоге — `<pager current-page="@Model.CurrentPage" total-pages="@Model.TotalPages" category="@category"></pager>` (ручная разметка пагинации из ЛР3 удалена), в админке ([Areas/Admin/Pages/Index.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Index.cshtml)) — `<pager ... admin="true"></pager>` (пагинация из ЛР5 удалена).

**3.2. Задание 2. Переключение страниц через Ajax**
- Список карточек и пейджер вынесены в частичное представление [Views/Product/_ListPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/_ListPartial.cshtml); в [Views/Product/Index.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/Index.cshtml) остались заголовок, выбор категории и контейнер `<div id="productList"><partial name="_ListPartial" model="Model" /></div>`.
- Скрипт [wwwroot/js/site.js](WEB_453501_Kolesnikow.UI/wwwroot/js/site.js) (подключён в `_Layout.cshtml` после jQuery): делегированный обработчик `$("#productList").on("click", ".page-link", ...)` отменяет переход (`e.preventDefault()`) и загружает фрагмент `$("#productList").load(url)`. jQuery сам добавляет заголовок `X-Requested-With: XMLHttpRequest`.

**3.3. Задание 3. Расширяющий метод** — [UI/Extensions/HttpRequestExtensions.cs](WEB_453501_Kolesnikow.UI/Extensions/HttpRequestExtensions.cs): `public static bool IsAjaxRequest(this HttpRequest request)` проверяет заголовок `x-requested-with`. В [ProductController](WEB_453501_Kolesnikow.UI/Controllers/Product.cs): `if (Request.IsAjaxRequest()) return PartialView("_ListPartial", productResponse.Data);`, иначе — `View(...)`.

### Проверка (выполнена)

| Проверка | Результат |
|---|---|
| `/Product` — HTML страницы | вместо `<pager>` — `<nav aria-label="Переключение страниц"><ul class="pagination ...">`, ссылки `/Product?pageNo=1`, `/Product?pageNo=2` |
| `/Product?pageNo=2` с заголовком `X-Requested-With: XMLHttpRequest` | только карточки (пиццы 4–6) и пейджер, без `<html>`, `<header>`, `<footer>` |
| категория с >1 страницы | ссылки пейджера `/Product?category=classic&pageNo=2` |
| категория с 1 страницей | пейджер не выводится |
| `/Admin` (вход администратором) | ссылки `/Admin?pageNo=1`, `/Admin?pageNo=2`, текущая страница — `active` |

В браузере: F12 → **Network → Fetch/XHR** → щелчок по «2» → запрос `Product?pageNo=2` с заголовком `X-Requested-With`, в **Response** — только фрагмент; страница не мигает.

### Ответы на контрольные вопросы

**1. Что такое tag-helper?**

Класс C# (наследник `TagHelper`), который на сервере участвует в отрисовке HTML-элемента в разметке Razor: по тегу и/или атрибутам находит элемент и меняет его — атрибуты, содержимое, сам тег. Выглядит как обычный HTML, поэтому разметка остаётся читаемой.

**Где в проекте:**
- собственный — [UI/TagHelpers/PagerTagHelper.cs](WEB_453501_Kolesnikow.UI/TagHelpers/PagerTagHelper.cs): тег `<pager>` превращается в разметку Bootstrap Pagination;
- встроенные — `asp-for`, `asp-action`, `asp-page`, `<partial>` и др., например в [_ListPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/_ListPartial.cshtml):
  ```razor
  <a asp-controller="Cart" asp-action="Add" asp-route-id="@item.Id" asp-route-returnUrl="@returnUrl" ...>
  ```

**2. Какие tag-helpers тэга `<form>` вы знаете?**

`asp-controller`, `asp-action`, `asp-area`, `asp-page`, `asp-page-handler`, `asp-route-{параметр}`, `asp-route` (имя маршрута), `asp-all-route-data`, `asp-fragment`, `asp-antiforgery`. Кроме формирования `action`, tag helper `<form method="post">` автоматически добавляет скрытое поле antiforgery-токена.

**Где в проекте:**
- [_UserInfoPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_UserInfoPartial.cshtml) — выход:
  ```razor
  <form id="logoutForm" class="form-inline" method="post"
        asp-area="" asp-controller="Account" asp-action="LogOut">     → action="/Account/LogOut" + скрытый __RequestVerificationToken
  ```
- [Views/Account/Register.cshtml](WEB_453501_Kolesnikow.UI/Views/Account/Register.cshtml): `<form asp-action="Register" method="post" enctype="multipart/form-data">`;
- страницы админки ([Create.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Create.cshtml)): `<form method="post" ...>` без `asp-*` — отправка на текущую страницу, токен добавлен автоматически.

**3. Какие tag-helpers тэга `<a>` вы знаете?**

`asp-controller`, `asp-action`, `asp-area`, `asp-page`, `asp-page-handler`, `asp-route-{параметр}`, `asp-route`, `asp-all-route-data`, `asp-fragment`, `asp-host`, `asp-protocol`.

**Где в проекте:**
- [_MenuPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_MenuPartial.cshtml):
  ```razor
  <a ... asp-controller="Product" asp-action="Index">Каталог</a>                 → /Product
  <a ... asp-area="Admin" asp-page="/Index">Администрирование</a>                → /Admin
  ```
- [Admin/Pages/Index.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Index.cshtml): `<a asp-page="./Edit" asp-route-id="@item.Id" ...>` → `/Admin/Edit/5`;
- [Views/Product/Index.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/Index.cshtml): `asp-route-category="@item.NormalizedName"` → `/Product?category=cheese`.

**4. Как зарегистрировать использование tag-helper в проекте?**

Директивой `@addTagHelper <классы>, <сборка>` в `_ViewImports.cshtml` (действует на папку и вложенные) или в самом представлении; `*` — все tag helper'ы сборки. Отключить — `@removeTagHelper`, потребовать префикс — `@tagHelperPrefix`.

**Где в проекте:** обе папки разметки имеют свой `_ViewImports.cshtml`, поэтому строка добавлена в оба файла:
- [Views/_ViewImports.cshtml](WEB_453501_Kolesnikow.UI/Views/_ViewImports.cshtml) и [Areas/Admin/Pages/_ViewImports.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/_ViewImports.cshtml):
  ```razor
  @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers     @* встроенные *@
  @addTagHelper *, WEB_453501_Kolesnikow.UI                 @* собственные (PagerTagHelper) *@
  ```

**5. Как определить, что запрос выполнен по Ajax?**

По заголовку запроса `X-Requested-With: XMLHttpRequest` (jQuery добавляет его сам; при `fetch` его нужно указать вручную).

**Где в проекте:** [UI/Extensions/HttpRequestExtensions.cs](WEB_453501_Kolesnikow.UI/Extensions/HttpRequestExtensions.cs):
```csharp
public static bool IsAjaxRequest(this HttpRequest request)
{
    ArgumentNullException.ThrowIfNull(request);
    return string.Equals(request.Headers["x-requested-with"],
                         "XMLHttpRequest",
                         StringComparison.OrdinalIgnoreCase);
}
```

**6. Может ли MVC-контроллер вернуть частичное представление?**

Да — `return PartialView("имя", модель);`. Частичное представление отрисовывается без макета `_Layout`.

**Где в проекте:** [UI/Controllers/Product.cs](WEB_453501_Kolesnikow.UI/Controllers/Product.cs):
```csharp
// ЛР7: асинхронный запрос — вернуть только список и пейджер (без макета)
if (Request.IsAjaxRequest())
    return PartialView("_ListPartial", productResponse.Data);

// обычный запрос — полное представление с макетом
return View(productResponse.Data);
```

**7. Может ли контроллер вернуть ответ без тела сообщения? Пример.**

Да: `return NoContent();` (204), `return Ok();` (200 без тела), `return StatusCode(404);`, `return Unauthorized();`.

**Где в проекте:**
- [API/Controllers/CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs) — `PutCategory` и `DeleteCategory` возвращают `return NoContent();` (204);
- [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs) — PUT и DELETE: `return TypedResults.Ok();` (200 без тела);
- [UI/Controllers/AccountController.cs](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs) — `Register` при ошибке получения токена: `return Unauthorized();` (401 без тела).

**8. Пример, когда контроллер возвращает ответ, не являющийся представлением.**

`NotFound(сообщение)`, `BadRequest(...)`, `Content("текст")`, `Json(объект)`, `File(...)`, `Redirect(...)`/`RedirectToAction(...)`; в API-контроллерах — объекты (JSON).

**Где в проекте:**
- [ProductController](WEB_453501_Kolesnikow.UI/Controllers/Product.cs): `return NotFound(productResponse.ErrorMessage);` — 404 с текстом (например, `/Product?pageNo=50`);
- [AccountController](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs): `return Content("Доступ запрещён: нужна роль администратора (POWER-USER)");`, `return RedirectToAction("Index", "Home");`, `return BadRequest(response.StatusCode);`;
- [CategoriesController](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs): `return await _context.Categories.ToListAsync();` — JSON.

**9. Можно ли tag-helper'ом поменять тэг элемента?**

Да — свойством `output.TagName` (а `output.TagName = null` убирает тег, оставляя содержимое; `output.SuppressOutput()` убирает всё).

**Где в проекте:** [PagerTagHelper.cs](WEB_453501_Kolesnikow.UI/TagHelpers/PagerTagHelper.cs) — в браузер `<pager>` не попадает:
```csharp
// <pager> заменяется на <nav aria-label="..."> ... </nav>
output.TagName = "nav";
output.TagMode = TagMode.StartTagAndEndTag;
```

**10. Как в tag-helper получить URL конечной точки?**

Внедрить в конструктор `LinkGenerator` (зарегистрирован в DI по умолчанию) и вызвать `GetPathByAction` / `GetPathByPage` / `GetPathByName` (или `GetUriBy...` для абсолютного адреса). Другой способ — `IUrlHelperFactory` + `ViewContext`.

**Где в проекте:** [PagerTagHelper.cs](WEB_453501_Kolesnikow.UI/TagHelpers/PagerTagHelper.cs):
```csharp
public class PagerTagHelper(LinkGenerator linkGenerator,
                            IHttpContextAccessor httpContextAccessor) : TagHelper
...
if (Admin)
    return linkGenerator.GetPathByPage(httpContext, page: "/Index",
                                       values: new { area = "Admin", pageNo });    // → /Admin?pageNo=2
return linkGenerator.GetPathByAction("Index", "Product",
                                     new { category = Category, pageNo });          // → /Product?category=classic&pageNo=2
```

---

## ЛР8. Маршрутизация. Передача состояния (сессия, кэш, корзина)

**Цель:** знакомство с системой маршрутизации ASP.NET Core и механизмом сессии.

**Итог:** каталог открывается по адресам `/Catalog` и `/Catalog/{категория}`; список пицц в API кэшируется (HybridCache: память процесса + Redis); работает корзина заказов, которая хранится в сессии, а `Cart` внедряется в контроллер и компонент через DI.

### Что сделано по пунктам задания

**3.2. Задание 1. Маршрутизация** — атрибутная маршрутизация в [UI/Controllers/Product.cs](WEB_453501_Kolesnikow.UI/Controllers/Product.cs):
```csharp
// ЛР8: атрибутная маршрутизация — /Catalog и /Catalog/{категория}
[Route("Catalog/{category?}")]
public async Task<IActionResult> Index(string? category, int pageNo = 1)
```
- Все ссылки, построенные tag helper'ами (`asp-controller="Product" asp-action="Index"`) и `LinkGenerator` (пейджер ЛР7), сами стали вида `/Catalog/...` — разметку переписывать не пришлось.
- В меню [_MenuPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_MenuPartial.cshtml) у ссылки «Каталог» добавлено `asp-route-category="@null"` (иначе на `/Catalog/cheese` ссылка унаследовала бы текущую категорию) и `asp-area=""` (чтобы со страниц админки не уходить в область `Admin`).
- Метод с `[Route]` больше не доступен по общему маршруту: `/Product` и `/Product/Index` → 404.

**3.3. Задание 2. Кэширование ответов API**
- Redis в [API/DB/compose.yaml](WEB_453501_Kolesnikow.API/DB/compose.yaml): сервис `redis` (`redis:7.2-alpine`, порт 6379, том `redis_data`); проверка `docker exec redis redis-cli ping` → `PONG`.
- Пакеты `Microsoft.Extensions.Caching.Hybrid` 9.10.0 и `Microsoft.Extensions.Caching.StackExchangeRedis` 9.0.20; строка `"Redis": "localhost:6379"` — в разделе `ConnectionStrings` `appsettings.json`.
- [API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs): `AddHybridCache()` + `AddStackExchangeRedisCache(opt => { opt.InstanceName = "labs_"; opt.Configuration = ...GetConnectionString("Redis"); })` — Redis становится вторым уровнем (L2) HybridCache.
- [API/Endpoints/PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs) — точка списка оборачивает MediatR в `await cache.GetOrCreateAsync($"pizzas_{category}_{pageNo}_{pageSize}", ...)` с `Expiration = 1 мин`, `LocalCacheExpiration = 30 с`. В коде методички нет `await` — без него в ответ ушла бы незавершённая задача; ключ содержит все параметры запроса, иначе разные страницы получили бы одинаковый ответ.
- Проверка (запрос `GET /api/Pizza/cheese?pageNo=1&pageSize=2`):

| Момент | Откуда ответ | Время |
|---|---|---|
| первый запрос | БД (промах кэша), ключ `labs_pizzas_cheese_1_2` появился в Redis, TTL 59 с | 35 мс |
| через < 30 с | L1 — память процесса | 23 мс |
| через 35 с | L2 — Redis (TTL ключа 23 с) | 32 мс |
| через 65 с | ключа в Redis нет → снова БД | 39 мс |

Плата за кэш: после изменения пиццы в админке список может оставаться старым до 1 минуты.

**3.4. Задание 3. Корзина заказов**
- API: чтение пиццы по `Id` нужно корзине от имени **обычного** пользователя, а в ЛР6 вся группа требовала роль `admin` (обычный получил бы 403). Требование перенесено с группы на отдельные точки: `GET /{id}` — `.RequireAuthorization()` (любой вошедший), `POST`/`PUT`/`DELETE` — `.RequireAuthorization("admin")`, список — `.AllowAnonymous()`. Проверено: токен `user@example.com` → `GET /api/Pizza/1` 200, `DELETE /api/Pizza/999` 403; без токена `GET /api/Pizza/1` 401.
- Сессия в [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs): `AddDistributedMemoryCache()` + `AddSession(...)` (`IdleTimeout` 20 мин, `HttpOnly`, `IsEssential`) и `app.UseSession()` после `UseAuthorization()`.
- [Domain/Models/CartItem.cs](WEB_453501_Kolesnikow.Domain/Models/CartItem.cs) (пицца + количество) и [Domain/Models/Cart.cs](WEB_453501_Kolesnikow.Domain/Models/Cart.cs): словарь `CartItems` (ключ — `Id` пиццы), `virtual` методы `AddToCart` (повторное добавление увеличивает количество), `RemoveItems`, `ClearAll`, свойства `Count` и `TotalPrice` (у нас цена вместо калорий).
- [UI/Extensions/SessionExtensions.cs](WEB_453501_Kolesnikow.UI/Extensions/SessionExtensions.cs): `Set<T>`/`Get<T>` — объект хранится в сессии JSON-строкой (`ISession` умеет хранить только строки, числа и байты).
- [UI/Controllers/CartController.cs](WEB_453501_Kolesnikow.UI/Controllers/CartController.cs): `Index` (список), `Add` (`[Authorize]`, маршрут `[Route("[controller]/add/{id:int}")]`, возврат на `returnUrl` с проверкой `Url.IsLocalUrl`), `Remove` и `Clear` (POST + `[ValidateAntiForgeryToken]`).
- [UI/Views/Cart/Index.cshtml](WEB_453501_Kolesnikow.UI/Views/Cart/Index.cshtml) — таблица (картинка, название, количество, стоимость, «Удалить»), итог, «Очистить корзину».
- Компонент корзины перенесён в [UI/Components/CartViewComponent.cs](WEB_453501_Kolesnikow.UI/Components/CartViewComponent.cs) (старое пространство имён `...Views.Shared.Components.Cart` совпадало с именем класса `Cart`); [Default.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/Components/Cart/Default.cshtml) показывает реальные `TotalPrice` и `Count`.

**3.5. Задание 4. Сервис корзины** — контроллер и компонент не работают с сессией напрямую:
- [UI/Services/CartService/SessionCart.cs](WEB_453501_Kolesnikow.UI/Services/CartService/SessionCart.cs) — `SessionCart : Cart`: фабрика `GetCart(IServiceProvider)` загружает корзину из сессии (через `IHttpContextAccessor`), переопределённые методы после `base.Метод(...)` сохраняют корзину обратно в сессию; свойство `Session` помечено `[JsonIgnore]`.
- Регистрация в [HostingExtensions.cs](WEB_453501_Kolesnikow.UI/Extensions/HostingExtensions.cs): `builder.Services.AddScoped<Cart>(sp => SessionCart.GetCart(sp));` — один объект корзины на запрос.
- `CartController(IProductService productService, Cart cart)` и `CartViewComponent(Cart cart)` получают корзину через конструктор; в них нет обращений к `Session`.
- Поскольку Задание 4 требует убрать работу с сессией из контроллера, в итоговом коде сразу используется `SessionCart`. Для сравнения, вариант Задания 3 (как в методичке) выглядел бы так:
  ```csharp
  [Authorize]
  [Route("[controller]/add/{id:int}")]
  public async Task<ActionResult> Add(int id, string? returnUrl)
  {
      Cart cart = HttpContext.Session.Get<Cart>("cart") ?? new();    // читать из сессии
      var data = await _productService.GetProductByIdAsync(id);
      if (data.Successfull && data.Data != null)
      {
          cart.AddToCart(data.Data);
          HttpContext.Session.Set<Cart>("cart", cart);               // записать обратно
      }
      return Redirect(returnUrl);
  }
  ```
- Дополнительно: при выходе (`AccountController.Logout`) корзина очищается — сессия не привязана к пользователю, и следующий вошедший иначе увидел бы чужую корзину.

### Проверка (выполнена)

| Действие | Результат |
|---|---|
| `/Catalog`, `/Catalog/cheese` | 200; `/Product`, `/Product/Index` — 404 |
| ссылки на странице | меню «Каталог» → `/Catalog`; категории → `/Catalog/classic`, `/Catalog/cheese`…; пейджер → `/Catalog?pageNo=2`; «В корзину» → `/Cart/add/1?returnUrl=%2FCatalog` |
| Ajax `/Catalog?pageNo=2` | только фрагмент (пиццы 4–6), без `<html>`/`<header>` |
| «В корзину» без входа | перенаправление на страницу входа Keycloak |
| вход `user@example.com`, пицца 1 ×2 и пицца 5 ×1 | возврат на ту же страницу каталога; в шапке `61.00 руб (3)`; в `/Cart` две строки, у первой количество 2 |
| «Удалить» пиццу 5 / «Очистить корзину» | `42.00 руб (2)` / «Корзина пуста», `0.00 руб (0)` |
| POST `/Cart/Clear` без antiforgery-токена | 400 |
| Logout → вход другим пользователем | корзина пустая |
| cookies браузера | `.AspNetCore.Session` (id сессии), `.AspNetCore.Cookies` (вход) |
| админка после переноса авторизации | создание и удаление пиццы работают |

### Ответы на контрольные вопросы

**1. Какие ещё механизмы сохранения состояния есть в ASP.NET Core, кроме сессии?**

Строка запроса (query string), скрытые поля формы, cookie, `TempData` (переживает один redirect), `HttpContext.Items` (в пределах одного запроса), кэш (в памяти, распределённый), база данных.

**Где в проекте:**

| Механизм | Где используется |
|---|---|
| строка запроса | номер страницы: `/Catalog?pageNo=2` ([PagerTagHelper.cs](WEB_453501_Kolesnikow.UI/TagHelpers/PagerTagHelper.cs)); адрес возврата `?returnUrl=` у «В корзину» ([_ListPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/_ListPartial.cshtml)) |
| скрытые поля формы | [Admin/Pages/Edit.cshtml](WEB_453501_Kolesnikow.UI/Areas/Admin/Pages/Edit.cshtml): `<input type="hidden" asp-for="Pizza.ImageFilePath" />` |
| cookie | вход пользователя — `.AspNetCore.Cookies` (ЛР6, [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) `.AddCookie()`) |
| кэш | ответы API — HybridCache + Redis ([PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs)) |
| БД | пиццы и категории — PostgreSQL (ЛР4) |
| сессия | корзина ([SessionCart.cs](WEB_453501_Kolesnikow.UI/Services/CartService/SessionCart.cs)) |

**2. Где хранятся данные сессии?**

На сервере — в хранилище `IDistributedCache` (в памяти процесса, Redis, SQL Server…). Клиент получает только cookie с идентификатором сессии.

**Где в проекте:** [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) — хранилище в памяти процесса `UI`:
```csharp
builder.Services.AddDistributedMemoryCache();   // где лежат данные сессии
builder.Services.AddSession(...);
```
В браузере — только cookie `.AspNetCore.Session`; сама корзина (JSON) лежит на сервере под ключом `"cart"`.

**3. Сколько времени хранятся данные сессии?**

Пока не истечёт время бездействия `IdleTimeout` (по умолчанию 20 минут; каждый запрос продлевает срок). Cookie сессии по умолчанию сеансовая — удаляется при закрытии браузера. При хранилище в памяти данные пропадают и при перезапуске приложения.

**Где в проекте:** [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs):
```csharp
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(20); // время жизни при бездействии
    ...
});
```

**4. Как прочитать / записать данные Cookie?**

Чтение — `Request.Cookies["имя"]`; запись — `Response.Cookies.Append("имя", "значение", new CookieOptions { Expires = ... })`; удаление — `Response.Cookies.Delete("имя")`.

**Где в проекте:** вручную cookie не читаются и не пишутся — их создают Middleware аутентификации (`.AspNetCore.Cookies`) и сессии (`.AspNetCore.Session`). Если бы, например, нужно было запоминать выбранную категорию, в `ProductController.Index` это выглядело бы так:
```csharp
Response.Cookies.Append("lastCategory", category ?? "", new CookieOptions { Expires = DateTimeOffset.Now.AddDays(7) });
var last = Request.Cookies["lastCategory"];
```

**5. Сколько времени хранятся данные Cookie?**

Без `Expires`/`MaxAge` — сеансовая cookie, живёт до закрытия браузера. С `Expires`/`MaxAge` — постоянная, до указанного срока (или пока её не удалят). Размер — до ~4 КБ.

**Где в проекте:** `.AspNetCore.Session` — сеансовая (закрыли браузер → корзина «потерялась»). Cookie входа `.AspNetCore.Cookies` тоже сеансовая (вход не сохраняется как «постоянный»), а срок самого билета входа внутри неё — 14 дней по умолчанию `CookieAuthenticationOptions.ExpireTimeSpan`.

**6. Какие преимущества даёт распределённый (Distributed) кэш?**

Он общий для всех экземпляров приложения (одинаковые данные на всех серверах при масштабировании), переживает перезапуск и обновление приложения, не занимает память процесса, разгружает БД.

**Где в проекте:** Redis в [compose.yaml](WEB_453501_Kolesnikow.API/DB/compose.yaml) — второй уровень HybridCache ([API/Program.cs](WEB_453501_Kolesnikow.API/Program.cs)):
```csharp
builder.Services.AddHybridCache();                      // L1 — память процесса
builder.Services.AddStackExchangeRedisCache(opt =>      // L2 — Redis, общий для всех экземпляров API
{
    opt.InstanceName = "labs_";
    opt.Configuration = builder.Configuration.GetConnectionString("Redis");
});
```
Видно и в проверке: через 35 с запись в памяти уже истекла, но ответ пришёл из Redis, а не из БД.

**7. Что такое «статический сегмент маршрута»? Пример.**

Часть шаблона маршрута — буквальный текст, который должен совпасть с адресом точно (в отличие от параметра в `{ }`).

**Где в проекте:**
- `Catalog` в [ProductController](WEB_453501_Kolesnikow.UI/Controllers/Product.cs): `[Route("Catalog/{category?}")]` — `Catalog` статический, `{category?}` — параметр;
- `add` в [CartController](WEB_453501_Kolesnikow.UI/Controllers/CartController.cs): `[Route("[controller]/add/{id:int}")]`;
- `api` в [CategoriesController](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs) и группе `/api/Pizza`.

**8. Как в контроллере получить значение сегмента маршрута?**

Параметром метода действия с тем же именем (привязка модели) или через `RouteData.Values["имя"]` (`HttpContext.GetRouteValue("имя")`).

**Где в проекте:**
- параметр — [ProductController](WEB_453501_Kolesnikow.UI/Controllers/Product.cs): `Index(string? category, int pageNo = 1)` — для `/Catalog/cheese` `category = "cheese"`; [CartController](WEB_453501_Kolesnikow.UI/Controllers/CartController.cs): `Add(int id, ...)` из `/Cart/add/5`;
- `RouteData` — в представлении [_ListPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/_ListPartial.cshtml):
  ```csharp
  string? category = ViewContext.RouteData.Values["category"]?.ToString()
                     ?? request.Query["category"].FirstOrDefault();
  ```

**9. Как в `<a>` с tag-helper указать значение сегмента маршрута?**

Атрибутом `asp-route-{имя}="значение"`: если имя есть в шаблоне маршрута — значение станет сегментом пути, если нет — попадёт в строку запроса.

**Где в проекте:**
- [Views/Product/Index.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/Index.cshtml): `asp-route-category="@item.NormalizedName"` → `/Catalog/cheese` (сегмент);
- [_ListPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Product/_ListPartial.cshtml): `asp-route-id="@item.Id" asp-route-returnUrl="@returnUrl"` → `/Cart/add/1?returnUrl=%2FCatalog` (`id` — сегмент, `returnUrl` — строка запроса);
- [_MenuPartial.cshtml](WEB_453501_Kolesnikow.UI/Views/Shared/_MenuPartial.cshtml): `asp-route-category="@null"` — явно «без категории» → `/Catalog`.

**10. Как зарегистрировать маршрут в `Program`?**

`app.MapControllerRoute(name: "...", pattern: "...", defaults: new { ... }, constraints: new { ... })`; более специфичные маршруты регистрируются раньше общих.

**Где в проекте:** [UI/Program.cs](WEB_453501_Kolesnikow.UI/Program.cs) — общий маршрут:
```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}")
```
Вместо атрибута у каталога можно было бы зарегистрировать (до `default`):
```csharp
app.MapControllerRoute(name: "catalog", pattern: "Catalog/{category?}",
    defaults: new { controller = "Product", action = "Index" });
```

**11. Как зарегистрировать маршрут через атрибуты?**

Атрибутом `[Route("шаблон")]` над контроллером и/или действием (или `[HttpGet("шаблон")]` и т.п.); токены `[controller]`, `[action]` подставляют имена. Действие с атрибутным маршрутом не доступно по общим маршрутам из `Program.cs`.

**Где в проекте:**
- [ProductController](WEB_453501_Kolesnikow.UI/Controllers/Product.cs): `[Route("Catalog/{category?}")]`;
- [CartController](WEB_453501_Kolesnikow.UI/Controllers/CartController.cs): `[Route("[controller]/add/{id:int}")]`;
- [API/Controllers/CategoriesController.cs](WEB_453501_Kolesnikow.API/Controllers/CategoriesController.cs): `[Route("api/[controller]")]` на классе + `[HttpGet("{id}")]` на методе.

**12. Что такое ограничения (constraints) маршрута?**

Условия на значение параметра маршрута; если условие не выполнено, маршрут не совпадает. Примеры: `{id:int}`, `{name:alpha}`, `{x:minlength(3)}`, `{code:regex(...)}`, необязательный `{id?}`, значение по умолчанию `{action=Index}`.

**Где в проекте:**
- [CartController](WEB_453501_Kolesnikow.UI/Controllers/CartController.cs): `add/{id:int}` — `/Cart/add/abc` не совпадёт;
- [PizzaEndpoints.cs](WEB_453501_Kolesnikow.API/Endpoints/PizzaEndpoints.cs): `"/{id:int}"` и `"/{category?}"` — число уходит в «пицца по Id», текст — в «список категории»; без `:int` адрес `/api/Pizza/cheese` подходил бы под оба;
- страницы админки: `@page "{id:int?}"`.

**13. Как получить адрес конечной точки в коде контроллера / представления?**

В контроллере и представлении — `Url.Action("действие", "контроллер", new { ... })`, `Url.Page(...)`, `Url.RouteUrl(...)`; в разметке — tag helpers `asp-*`; вне контроллеров и представлений — сервис `LinkGenerator` (`GetPathByAction`, `GetPathByPage`).

**Где в проекте:**
- контроллер — [AccountController](WEB_453501_Kolesnikow.UI/Controllers/AccountController.cs): `RedirectUri = Url.Action("Index", "Home")` (куда вернуться после входа/выхода); [CartController](WEB_453501_Kolesnikow.UI/Controllers/CartController.cs): `Url.IsLocalUrl(returnUrl)` и `RedirectToAction("Index", "Product")` → `/Catalog`;
- tag helper — [PagerTagHelper.cs](WEB_453501_Kolesnikow.UI/TagHelpers/PagerTagHelper.cs): `linkGenerator.GetPathByAction("Index", "Product", new { category = Category, pageNo })` → `/Catalog/classic?pageNo=2` (после ЛР8 адрес изменился сам, код пейджера не менялся).
