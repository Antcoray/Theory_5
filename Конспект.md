# Конспект по курсу «Современные платформы прикладной разработки» (ASP.NET Core)

Конспект собран из всех лекционных материалов (.pdf/.pptx) курса СППР и структурирован по лабораторным работам. Всего в курсе **13 лабораторных работ** (ЛР1–ЛР13) — это подтверждено сквозным чтением всех 38 исходных файлов и совпадает с ожиданием заказчика. Материалы по GraphQL, gRPC и углублённому Blazor (файлы `14_`, `15_`, `17_0`…`17_4`) заданий лабораторных работ внутри не содержат — это чисто лекционные темы, вынесенные в конспекте в раздел «Дополнительные темы» после ЛР13.

## Как устроен конспект

- Каждый раздел `## Лабораторная работа №N` начинается с **полного текста задания** (все технические шаги, без сокращений), затем идёт вся теория, необходимая для его выполнения.
- Новые понятия сопровождаются курсивной пометкой *«Где пригодится»* — в какой лабораторной это понадобится в первую очередь и как это используется на практике/на аттестации.
- Вопросы к аттестации (документ «Вопросы к аттестации.pdf», 60 вопросов) сопоставлены с темами — рядом с соответствующим материалом стоит пометка **[Атт. №N]**.
- Сквозные темы (DI — Dependency Injection, внедрение зависимостей, полное определение см. в разделе ЛР3; MediatR, аутентификация Keycloak, Blazor) развиваются через несколько лабораторных подряд — даны кросс-ссылки «см. также ЛР№X».
- Там, где лекции не дают материала в объёме, достаточном для сдачи лабы (нюансы конфигурации, актуальный синтаксис .NET 8/9), конспект дополнен актуальными сведениями по ASP.NET Core/C#.

## Сквозной проект курса

Лабораторные работы выполняются на **одном сквозном проекте** — интернет-магазине/меню (пример предметной области у автора курса — «меню кафе»: сущности `Dish` — блюдо и `Category` — категория блюда). Решение (solution) развивается от лабы к лабе и к ЛР9 обычно состоит из проектов:

| Проект | Тип | Назначение | Появляется в |
|---|---|---|---|
| `XXX.UI` | ASP.NET Core Web App (MVC — Model-View-Controller, паттерн разделения приложения на модель/представление/контроллер, подробнее ниже + Razor Pages) | основной сайт: контроллеры, представления, Razor Pages администратора | ЛР1 |
| `XXX.Domain` | Class Library | сущности (`Entities`), вспомогательные модели (`ResponseData<T>`, `ListModel<T>`) | ЛР3 |
| `XXX.API` | ASP.NET Core Web API | REST API + Minimal API, EF Core, БД (PostgreSQL/SQLite) | ЛР4 |
| `XXX.Tests` | xUnit Test Project | модульные тесты контроллеров и сервисов | ЛР9 |
| `XXX.Blazor.SSR` | Blazor Web App (Server) | Blazor-версия интерфейса (SSR / Interactive Server) | ЛР11 |
| `XXX.BlazorWasm` | Blazor WebAssembly Standalone App | клиентское SPA-приложение на Blazor WASM | ЛР12 |

*Расшифровка сокращений из таблицы (подробные определения — в соответствующих разделах ниже):* **API** (Application Programming Interface — программный интерфейс приложения, через который одна программа обращается к функциям другой) и **REST** (архитектурный стиль построения API, полное определение — в разделе ЛР4) вместе описывают веб-сервис `XXX.API`; **EF Core** (Entity Framework Core) — **ORM**-библиотека (Object-Relational Mapping — технология, которая отображает таблицы БД на классы C#, чтобы работать с базой через объекты, а не писать SQL вручную); **xUnit** — библиотека/фреймворк для модульного (юнит-)тестирования кода на .NET (подробно — в ЛР9); **SSR** (Server-Side Rendering — формирование готовой HTML-страницы на сервере) и **WASM** (WebAssembly — технология выполнения скомпилированного .NET-кода прямо в браузере) — два режима работы Blazor (подробно — в ЛР11–ЛР12); **SPA** (Single Page Application — приложение с одной HTML-страницей, где переход между разделами идёт без полной перезагрузки) — то, что получается на Blazor WebAssembly.

`XXX` — имя решения вида `WEB_GGGG_NNN` (GGGG — номер группы, NNN — фамилия). Сервер аутентификации — отдельное **Docker**-приложение (Docker — платформа для запуска приложений в изолированных «контейнерах», не зависящих от окружения хост-машины) **Keycloak** (открытая система управления идентификацией и доступом — сервер, который берёт на себя аутентификацию/авторизацию пользователей; появляется в ЛР6 и используется дальше во всех работах, где нужна авторизация).

---

## Лабораторная работа №1: Знакомство с приложением ASP.NET Core

### Текст задания

**Цель работы.** Знакомство со структурой приложения ASP.NET MVC Core. Знакомство с представлениями (Views) и страницами-макетами (Layouts).

**Задача работы.** Научиться создавать представления, использующие страницы-макеты. Время выполнения — 4 часа (2 занятия).

**3.1. Подготовка к работе**
В Visual Studio создайте новый проект ASP.NET Core, шаблон **«Web Application (Model-View-Controller)»**. Имя решения `WEB_GGGG_NNN`, имя проекта в решении — `WEB_GGGG_NNN.UI` (GGGG — номер группы, NNN — фамилия).
В VS Code: `dotnet new mvc -o <имя проекта>`.

Для иконок подключите библиотеку стилей (например `bootstrap-icons`, `font-awesome` или `open-iconic`). В VS Code библиотека подключается через **LibMan** (Library Manager — лёгкий инструмент для загрузки клиентских библиотек типа Bootstrap без npm):
```
dotnet tool install -g Microsoft.Web.LibraryManager.Cli
libman init
libman install bootstrap-icons --provider cdnjs --destination wwwroot/lib/bootstrap-icons
```
В Visual Studio — правой кнопкой по `wwwroot/lib` → **Add → Client-side library**.

**3.2. Знакомство с проектом.** Найдите папку статических файлов, папки контроллеров и представлений. Откройте `Program.cs` — найдите регистрацию сервисов и конвейер обработки запросов. Откройте `appsettings.json`.

**3.3. Подготовка проекта.** Удалите `Controllers/HomeController.cs` и папку `Views/Home`. В проекте будут использоваться порты **7001** и **5001** — измените их в `Properties/launchSettings.json`:
```json
"applicationUrl": "https://localhost:7001;http://localhost:5001",
"environmentVariables": { "ASPNETCORE_ENVIRONMENT": "Development" }
```

**3.4. Создание контроллера и представления.** Создайте контроллер `Home`. Создайте представление для метода `Index` **без использования макета**, выводящее статический текст `"Hello World!"` (папку `Views/Home` создать вручную). Запустите проект и проверьте результат.

**3.5. Разметка представления Index.** На `Index` подключите стили `lib/bootstrap/dist/css/bootstrap.min.css`, `lib/bootstrap-icons/font/bootstrap-icons.min.css`, `css/site.css`, а также скрипты `lib/jquery/dist/jquery.min.js`, `lib/bootstrap/dist/js/bootstrap.bundle.min.js` (за образец взять `Views/Shared/_Layout.cshtml`). Пример разметки:
```html
@{ Layout = null; }
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Index</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body>
    Hello World!
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```
В `site.css` удалите всё содержимое и опишите:
```css
header, footer { background-color: rgba(var(--bs-dark-rgb)); }
.nav-color { color: rgba(255, 255, 255, 0.5) }
```

**3.6. Разметка структуры страницы.** Получить структуру: `<header>` (панель навигации) / `<main class="container">` (содержимое) / `<footer>` (подвал). Внутри `<header>` и `<footer>` — `<div class="container">` для ограничения ширины блока.

**3.7. Оформление заголовка страницы.** Заголовок содержит: **меню сайта** (бейдж с номером зачётки — ссылка на ЛР1, кнопки навигации «Лб1», «Каталог», «Администрирование» — классы Bootstrap navbar) и **информацию пользователя** (ссылка на корзину с иконкой `bi bi-cart` и текстом суммы/количества товаров; выпадающее меню с аватаркой, именем пользователя и кнопкой Logout). Аватарки хранить в `wwwroot/Images`. **Важно: пункт Logout — это форма, отправляющая данные методом POST.** Пример разметки навигации и dropdown — см. код в оригинале задания (классы `navbar`, `dropdown`, `dropdown-menu`, `dropdown-item-text`).

**3.8. Разметка подвала страницы.** Навигационная панель, смещённая вправо (`ms-auto`), с иконками соцсетей (`bi-facebook`, `bi-twitter`, `bi-telegram`).

**3.9. Разметка содержимого страницы.** HTML-разметка с упорядоченным списком `<ol>` (нумерация заглавными буквами), радиокнопками/чекбоксами/полями ввода **внутри `<form>`** с обязательными атрибутами `name` у всех элементов формы. Метод отправки формы указать `get` — чтобы видеть передаваемые данные в строке запроса. Использовать классы Bootstrap для горизонтальной формы.

**3.10. Использование Developer Tools.** Смените метод формы на `POST`. В браузере (F12 → Network) введите данные формы, отправьте, найдите свою страницу в списке запросов, откройте Headers → Form Data — убедитесь, что данные отправлены.

**3.12. Создание страницы-макета.** Удалите/переименуйте `Views/Shared/_Layout.cshtml`, создайте новую страницу макета. В `<head>` подключите стили (как на Index). В `<body>` перенесите разметку `<body>` из Index, но внутри `<main>` вместо содержимого поставьте `@RenderBody()`. В конце `<body>`, после скриптов, добавьте необязательную асинхронную секцию `"Scripts"`. В `Views/_ViewStart.cshtml` проверьте, что указано использование `_Layout.cshtml`.

**3.13. Использование страницы-макета.** В Index удалите всю разметку кроме содержимого `<main>`, добавьте кодовый блок:
```razor
@{ ViewBag.Title = "Index"; }
```
Запустите проект, убедитесь, что внешний вид не изменился.

**4. Вопросы для самопроверки** (пригодятся дословно для аттестации, см. также раздел «Дополнительные темы» и список 60 вопросов в конце конспекта):
1. В какой папке хранятся файлы представлений контроллера? 2. Где должен находиться статический контент? 3. Где создаётся конвейер обработки запросов? 4. Где размещаются классы контроллеров? 5. Для чего `_ViewStart.cshtml`? 6. Для чего `_ViewImports.cshtml`? 7. Для чего файл макета (Layout)? 8. Как в Layout указать место вставки разметки представления? 9. Что такое «Section» на макете? 10. Как использовать секцию в представлении? 11/12. Как указать/отменить использование Layout в представлении? 13. Почему «HTTP не поддерживает сохранение состояний»? 14. Как в `<form>` указать адрес отправки? 15. В каком виде данные передаются клиент→сервер? 16. Чем отличается передача данных в Post и Get?

### Теория для выполнения ЛР1

#### HTTP — протокол без сохранения состояния

Веб-приложение — сайт с частично/полностью динамически формируемым содержимым. Когда браузер запрашивает статическую страницу, сервер отдаёт её как есть; для динамической — сервер передаёт страницу специальной программе, которая генерирует итоговый HTML и удаляет серверный код со страницы. Клиенту всегда приходит только **HTML + JS**.

Схема простого взаимодействия: `Браузер --(1. GET)--> Web-сервер --(2. обработка, 3. генерация)--> --(4. отправка)--> Браузер --(5. отображение)-->`.

*Определение.* **HTTP** (HyperText Transfer Protocol) — протокол прикладного уровня передачи гипертекстовых данных, основан на модели «клиент-сервер». *Где пригодится: база для понимания абсолютно любой веб-лабы (ЛР1–ЛР13); часто спрашивают на аттестации* **[Атт. №1, №2]**.

**Особенность №1 (ключевая):** HTTP **не поддерживает сохранение состояний** (stateless) — нет сохранения промежуточного состояния между парами «запрос-ответ», каждый запрос обрабатывается независимо. *Где пригодится: объясняет, зачем в ЛР8 нужны Session/Cookie/TempData — иначе сервер «забывает» пользователя между запросами.* **[Атт. №1]**

Формат HTTP-сообщения — три части в строгом порядке: 1) **стартовая строка** (тип сообщения, например `GET /default.aspx HTTP/1.1`); 2) **заголовки** (Headers); 3) **тело сообщения** (Body), отделяется от заголовков пустой строкой.

- `GET /default.aspx?param1=value1&param2=value2 HTTP/1.1` — запрос ресурса, параметры через `?` в **URI** (Uniform Resource Identifier — строка, идентифицирующая ресурс; **URL**, Uniform Resource Locator, — её частный случай, адрес ресурса с указанием способа его получения, например `http://...`).
- `POST` — передача пользовательских данных ресурсу (данные — в теле, а не в URL).

Пример заголовка запроса: `Host`, `Last-Modified`, `Content-Type`, `Content-Language`. Пример ответа (`IIS` — Internet Information Services, веб-сервер Microsoft; здесь это сервер, который сформировал ответ):
```
HTTP/1.1 200 OK
Server: Microsoft-IIS/7.0
Content-Type: text/plain; charset=windows-1251
Content-Length: 51

<html><body><h1>Some text</h1></body></html>
```

**Коды состояния HTTP** *(таблица для запоминания, спрашивают на аттестации [Атт. №2])*:

| Класс | Значение |
|---|---|
| 1xx Informational | информационные |
| 2xx Success | успех (200 OK, 201 Created) |
| 3xx Redirection | перенаправление |
| 4xx Client Error | ошибка клиента (400, 401, 404) |
| 5xx Server Error | ошибка сервера |

Доступ к элементам HTTP-запроса на сервере — сервер **Kestrel** (встроенный в ASP.NET Core кроссплатформенный веб-сервер, который принимает HTTP-запросы) слушает запросы и передаёт их приложению в виде объекта **`HttpContext`**, из которого доступны свойства `Session`, `Request`, `Response`, `Server`, `User` (например `HttpContext.Request`). *Где пригодится: `HttpContext` — сквозной API, используется в контроллерах (ЛР3), Middleware (ЛР10), сессии (ЛР8), аутентификации (ЛР6).*

#### Основы HTML, CSS, Bootstrap

Ключевые теги HTML: `<html>/<head>/<body>`; `<b>/<strong>`, `<i>/<em>` (`<em>` — от emphasize, «выделить», смысловое, а не только визуальное выделение; в HTML5 предпочтительны семантические `<strong>`/`<em>` вместо чисто визуальных `<b>`/`<i>`); `<div>` (сокращение от division — «раздел») — блочный элемент для стилизуемых фрагментов; списки `<ul><li>` (маркированный) и `<ol><li>` (нумерованный, атрибуты `type`, `reversed`, `start`); ссылки/якоря `<a href="URL">` / `<a name="id">`; таблицы `<table><tr><td>` (`colspan` для объединения ячеек); формы `<form method="get|post" action="...">` с элементами `<input type="radio|checkbox|text|submit">`.

**Метод GET** формы — данные видны в адресной строке (`?имя=значение`), не подходит для чувствительных/больших данных. **Метод POST** — данные в теле запроса, используется почти всегда в формах и для загрузки файлов. *Где пригодится: ЛР1 п.3.9–3.10 наглядно показывает разницу в Developer Tools; выбор метода — частый вопрос на аттестации [Атт. №5].*

**CSS** — механизм управления представлением документов. Синтаксис: `selector { свойство: значение; }`. Селекторы: по тегу (`b`), по классу (`.warning`, `p.warning`), по id (`#left_panel`), по атрибуту (`[required]`), псевдо-классы/элементы (`p:hover`, `p::first-letter`). Комбинации: вложенный `#div1 p`, дочерний `#div1 > p`, родственный `table ~ div`, соседний `table + div`. Подключение: `<link rel="stylesheet" type="text/css" href="...">` в `<head>`.

*Определение.* **Bootstrap** — свободный набор инструментов (HTML/CSS-шаблоны + JS) для типографики, форм, кнопок, навигации. Особенности: поддержка тем, **адаптивный дизайн**, 12-колоночная **сетка (grid)**, готовые компоненты (кнопки, breadcrumbs, навигация, метки). *Где пригодится: вся вёрстка курса (ЛР1–ЛР13) идёт на Bootstrap 5.3; знание сетки/navbar/dropdown/card/pagination обязательно.*

#### Платформа ASP.NET Core, шаблон MVC, структура приложения

*Определение.* **ASP.NET (Active Server Pages)** — технология создания веб-приложений/веб-сервисов от Microsoft, часть платформы .NET. **ASP.NET Core** выполняется в CLR, код может быть на любом языке .NET. Выпущена в 2002 как Web Forms (каждая страница = физический файл).

**Шаблон MVC (Model-View-Controller)** — разделение приложения на три компонента, чтобы изменение одного минимально влияло на остальные:
- **Model** — классы данных и бизнес-логики (в ASP.NET: инкапсулируют данные БД/представления + код для манипуляции ими). НЕ должна: работать с UI, содержать логику отображения.
- **View** — шаблон динамической генерации HTML. НЕ должна: содержать сложную логику, логику сохранения/изменения модели.
- **Controller** — координатор между View и Model: принимает запрос клиента, обращается к модели, выбирает представление для ответа. НЕ должен: управлять отображением или хранимыми данными.

Преимущества MVC: несколько View на одну Model без изменения модели; смена реакции на действия пользователя через другой Controller; раздельное проектирование модели/интерфейса; удобное модульное тестирование. *Где пригодится: базовая архитектура для ЛР1–ЛР10 (MVC-часть проекта); частый вопрос на аттестации* **[Атт. №9]**.

**Структура приложения ASP.NET Core** — это обычное консольное приложение, создающее веб-сервер в `Main`:
- **Content Root** — базовый путь ко всему контенту приложения (по умолчанию = корень проекта).
- **Web Root** (`wwwroot`) — директория для открытых статических ресурсов (css/js/изображения); Middleware статических файлов по умолчанию отдаёт файлы только из неё (и подпапок). *Где пригодится: ЛР1 (стили/скрипты), ЛР3 (изображения блюд), ЛР5 (файлы), почти все лабы* **[Атт. №31]**.
- `Program.cs` — точка входа, где строится конвейер Middleware и настраиваются сервисы (DI).

**Конвейер Middleware** — ПО, собранное в цепочку обработки запросов/ответов приложения; каждый компонент решает, передавать ли запрос дальше, и может выполнять код до/после следующего компонента (`Middleware1 → Middleware2 → ... → Application`). *Где пригодится: подробно в ЛР10 (создание своего Middleware для логирования), а базовое понимание нужно уже с ЛР1, т.к. `app.UseStaticFiles()`, `app.UseRouting()` и т.д. — это регистрация встроенных Middleware* **[Атт. №42]**. Подробнее о создании своего Middleware — см. раздел ЛР10.

Пример `Program.cs`:
```csharp
var builder = WebApplication.CreateBuilder(args);
// Регистрация сервисов (DI-контейнер)
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddControllersWithViews();
var app = builder.Build();
// Конвейер обработки HTTP-запросов
app.UseStaticFiles();
app.UseRouting();
app.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}");
app.Run();
```

**Конфигурация приложения** — `appsettings.json` (формат ключ-значение), другие поставщики: переменные среды, Azure Key Vault, аргументы командной строки, объекты в памяти. Пример:
```json
{
  "ConnectionStrings": { "DefaultConnection": "Data source = Test.db" },
  "ApiData": { "Url": "https://somedata.by/api/", "ClientName": "dataConsumer" },
  "Logging": { "LogLevel": { "Default": "Information" } },
  "AllowedHosts": "*"
}
```
Чтение значений:
```csharp
var apiUrl = builder.Configuration["ApiData:Url"];
var apiUrl2 = builder.Configuration.GetSection("ApiData:Url").Value;
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection"); // спец. метод для строк подключения
```
Привязка секции к **POCO**-классу (Plain Old CLR Object — обычный простой класс без атрибутов и базовых классов, специфичных для конкретного фреймворка):
```csharp
public class ConfigData { public string Uri { get; set; } public string ClientName { get; set; } }
var config = builder.Configuration.GetSection("ApiData").Get<ConfigData>();
```
**`IOptions<T>`** — «правильный» способ получать конфигурацию через DI (не напрямую из `IConfiguration`, что упрощает тестирование):
```csharp
builder.Services.Configure<ConfigData>(builder.Configuration.GetSection("ApiData"));
// в конструкторе класса:
public HomeController(IOptions<ConfigData> config) { ConfigData cData = config.Value; }
```
*Где пригодится: `IOptions<T>` активно используется в ЛР6 (`KeycloakData`), ЛР8 (Redis/кэш), везде, где нужна типизированная конфигурация из `appsettings.json`* **[Атт. №7]**.

---

## Лабораторная работа №2: Razor. Частичные представления и компоненты представлений

### Текст задания

**Цель.** Знакомство с языком Razor. Изучение частичных представлений и компонентов представлений (2 часа).
**Задача.** Научиться создавать/использовать частичные представления и компоненты представлений. Научиться передавать данные от контроллера к представлению.

**3.1. Tag-helpers для формирования адресов ссылок.** Измените разметку главного меню, используя tag-helpers для ссылок:

| Пункт меню | Area | Controller | Action | Page |
|---|---|---|---|---|
| Лб1 | — | Home | Index | — |
| Каталог | — | Product | Index | — |
| Администрирование | — | Admin | — | /Index |

| Ссылка | Area | Controller | Action |
|---|---|---|---|
| Корзина | — | Cart | Index |
| Logout | — | Account | LogOut |

Запустите проект и проверьте результат (контроллеры `Product`, `Cart`, `Account` и страницы администратора пока отсутствуют — ссылки работать не будут).

**3.2. Передача данных от контроллера к представлению. Язык Razor.** а) Оформите список (`<ol>`) с помощью цикла `for`. б) Проверьте результат. в) Через `ViewData` передайте в Index текст **«Лабораторная работа №2»** вместо заголовка `<h1>Лабораторная работа 1</h1>`. г) На Index внутри формы создайте `<select>` на основе `SelectList`; класс элемента списка:
```csharp
public class ListDemo { public int Id { get; set; } public string Name { get; set; } }
```
Список отображает `Name`, а передаёт `Id` (атрибут `value`); список передаётся представлению как модель; для генерации `<select>` использовать tag-helper `asp-items`. д) Проверить в Developer Tools правильность разметки и передаваемых данных (Network → Payload).

**3.3. Создание частичных представлений.** а) Меню сайта и информацию пользователя (из ЛР1) оформить в виде частичных представлений. б) В меню предусмотреть класс `active` для текущего пункта (проверка `ViewContext.RouteData.Values["controller"]`):
```razor
@{
    var controller = ViewContext.RouteData.Values["controller"]?.ToString()?.ToLower() ?? String.Empty;
}
<a class="nav-item nav-link @(controller=="home"?"active":"")" asp-controller="home" asp-action="index">Лб 1</a>
```
в) Применить частичные представления на макете:
```html
<nav class="navbar bg-dark navbar-expand-md" data-bs-theme="dark">
    <partial name="_MenuPartial"/>
    <partial name="_UserInfoPartial"/>
</nav>
```
г) Убедиться, что вид страницы не изменился.

**3.4. Создание компонента представления.** а) Информацию о корзине заказа оформить в виде компонента представления (`ViewComponent`), использовать его в `_UserInfoPartial`: `@await Component.InvokeAsync("Cart")`. б) Проверить, что вид не изменился.

**4. Контрольные вопросы:** 1. Для чего tag-helper `asp-action`? 2. Что такое частичное представление? 3. Как разместить частичное представление в разметке? 4. Чем компонент представления отличается от частичного представления? 5. Как разместить компонент представления в разметке? 6. Где располагается файл представления компонента? 7. Чем отличаются `ViewData` и `ViewBag`? 8. Что такое модель представления? 9. Как указать в представлении класс модели? 10. Как получить доступ к членам класса модели представления?

### Теория для выполнения ЛР2

#### Представления и язык Razor

*Определение.* **Представление (View)** отвечает за пользовательский интерфейс; файлы `.cshtml` содержат HTML + код Razor. Располагаются в `~/Views/<Контроллер>` (для конкретного контроллера) или `~/Views/Shared` (для нескольких контроллеров).

*Определение.* **Razor** — синтаксис разметки для серверного кода на веб-страницах: смесь Razor-разметки, C# и HTML. Любой код Razor начинается с `@`; если после `@` идёт зарезервированное слово Razor — код транслируется в разметку, иначе — воспринимается как код C#.

- **Неявные выражения** — код C# сразу после `@` без пробелов: `@DateTime.Now.ToShortDateString()`. В них **нельзя** использовать обобщения (угловые скобки воспримутся как HTML-тег).
- **Явные выражения** — в круглых скобках после `@`: `@(DateTime.Now + TimeSpan.FromDays(1))`. Можно использовать для обобщений.
- **Кодовые блоки** — в фигурных скобках `@{ ... }`, не преобразуются в разметку:
```razor
@using XXX.Models
@{
    var books = new List<Book> { new Book{ BookId=1, Name="Book1", Author="Author1" } };
}
<table>
@foreach(var book in books) { <tr><td>@book.BookId</td><td>@book.Name</td></tr> }
</table>
```
- **`@functions { ... }`** — блок для описания методов, вызываемых из разметки.

`_ViewImports.cshtml` — общие `@using` и `@addTagHelper` для всех представлений папки:
```razor
@using XXX.UI
@using XXX.UI.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

#### Макеты (Layouts)

*Определение.* **Layout** — шаблон базовой разметки страницы (скрипты, CSS, навигация, контейнеры), задающий место, куда представление вставит своё содержимое: `@RenderBody()` — единственное место содержимого представления; `@RenderSection("Header", required: false)` — именованная необязательная/обязательная секция. *Где пригодится: ЛР1 (создание макета), везде далее.* **[Атт. №12]**

Использование в представлении:
```razor
@{
    ViewData["Title"] = "LayoutDemo";
    Layout = "~/Views/Shared/_Layout1.cshtml";
}
<div>Разметка из представления</div>
@section Footer { <h2>Footer из представления</h2> }
```
`_ViewStart.cshtml` — задаёт Layout по умолчанию для всех представлений: `@{ Layout = "_Layout"; }`. Чтобы отключить макет для конкретного представления: `@{ Layout = null; }`.

**Изоляция стилей CSS** — можно ограничить область действия стилей отдельной страницей/компонентом, чтобы избежать конфликтов и зависимости от глобальных стилей (файл `Имя.cshtml.css` рядом с представлением/компонентом).

#### ViewData, ViewBag, TempData, Model

| | ViewData | ViewBag | TempData |
|---|---|---|---|
| Тип | словарь `ViewDataDictionary` (ключ-строка → значение) | «обёртка» над ViewData с динамическими свойствами | как ViewData, но переживает редирект |
| Доступ | `ViewData["Name"]` | `ViewBag.Name` (через точку) | `TempData["Name"]` |
| Приведение типов | нужно (кроме string) | не требуется, легко проверять на null (`ViewBag.Person?.Name`) | нужно |
| Время жизни | только текущий запрос, уничтожается при Redirect | как ViewData | сохраняется через redirect (хранится в cookie/сессии) |

`TempData.Peek("x")` / `TempData.Keep("x")` — прочитать без удаления в конце запроса. *Где пригодится: `ViewData` — базовый способ передачи данных ЛР2/ЛР3 (список категорий, текущая категория); `TempData` пригодится при сообщениях после Redirect (ЛР6, ЛР10).* **[Атт. №14]**

**Model** — самый надёжный способ передачи данных: `@model IEnumerable<Book>` в начале представления объявляет тип, экземпляр передаётся из контроллера через `return View(model)`. *Где пригодится: используется во всех лабах со списками (ЛР2 `ListDemo`, ЛР3 `ListModel<Dish>`).*

Внедрение зависимостей прямо в представление: `@inject SignInManager<IdentityUser> SignInManager` — *где пригодится: ЛР6, для проверки `SignInManager.IsSignedIn(User)` в разметке меню*.

#### Частичные представления (Partial Views)

*Определение.* **Частичное представление** — файл Razor, генерирующий HTML-разметку **внутри другой разметки**; позволяет разбить большую разметку на части и убрать дублирование. Имя файла принято начинать с `_` и заканчивать `Partial`, например `_ListPartial.cshtml`. *Где пригодится: ЛР2 (меню, информация пользователя), ЛР7 (Ajax-обновление списка через частичное представление).* **[Атт. №11]**

Способы вставки: тег `<partial name="_RowPartial" model="@book"/>`; хелперы `@await Html.PartialAsync("_RowPartial", book)` (возвращает строку) или `await Html.RenderPartialAsync(...)` (пишет сразу в выходной поток, быстрее). Контроллер тоже может вернуть частичное представление: `return PartialView();`.

Поиск частичного представления (если указано только имя без пути), MVC-вариант: 1) `/Areas/<Area>/Views/<Controller>` 2) `/Areas/<Area>/Views/Shared` 3) `/Views/Shared` 4) `/Pages/Shared`.

Частичное представление получает **копию** `ViewData` родителя — изменения в нём не сохраняются в родителе.

#### Компоненты представлений (View Components)

*Определение.* **View Component** — класс, реализующий логику приложения в поддержку частичного представления или для вставки небольшого фрагмента HTML/JSON в родительское представление; заменяет `@Html.Action` из старых версий ASP.NET. В отличие от частичного представления, может содержать **сложную бизнес-логику** и параметры, работает только с частью ответа, использует преимущества разделения ответственности (как между контроллером и представлением) и тестируемости. *Где пригодится: ЛР2 — компонент корзины `Cart`; типичные применения — динамические меню, «облако тегов», панель входа, корзина.* **[Атт. №13]**

```csharp
public class CartSummary : ViewComponent
{
    private IRepository repository;
    public CartSummary(IRepository repo) { repository = repo; }
    public IViewComponentResult Invoke() => View(repository.GetAll());
}
```
Вызов на странице: `@await Component.InvokeAsync("CartSummary")`. Поиск представления компонента (имя представления по умолчанию — `Default.cshtml`):
`/Views/{Controller}/Components/{ИмяКомпонента}/{ИмяПредставления}` → `/Views/Shared/Components/{ИмяКомпонента}/{ИмяПредставления}` → `/Pages/Shared/Components/...` → `/Areas/{Area}/Views/Shared/Components/...`.

#### HTML Helpers и встроенные Tag Helpers

**HTML Helpers** (устаревающий, но встречающийся подход) — методы, генерирующие разметку:
```razor
@using (Html.BeginForm("Search", "Home", FormMethod.Get)) { <input type="text" name="q" /> }
@Html.TextBox("Title", Model.Color.Name)
@Html.HiddenFor(m => m.AlbumId)
@Html.LabelFor(m => m.Color, "Цвет")
@Html.ActionLink("Зарегистрироваться", "Register", "Home", new { @class = "btn" })
@Html.Partial("AlbumDisplay")
@Url.Action("Browse", "Store", new { genre = "Jazz" })   // выведет строку "/Store/Browse?genre=Jazz"
```
Строго типизированные варианты (`Html.xxxFor`) используют рефлексию по модели: `@Html.TextBoxFor(m => m.Color.Name)`.

*Определение.* **Tag Helpers** — классы C#, манипулирующие HTML-элементами (добавляют/меняют содержимое элемента), основываясь на имени элемента/атрибута/родителе; пришли на смену HTML Helpers. *Где пригодится: основа разметки во всех лабах с формами; создание собственных — тема ЛР7.* **[Атт. №18, №19]**

Основные встроенные tag-helpers:

| Тег | Атрибуты | Назначение |
|---|---|---|
| `<a>` | `asp-controller`, `asp-action`, `asp-route-{param}`, `asp-area`, `asp-protocol`, `asp-page` | генерация ссылки `<a href="...">` |
| `<form>` | `asp-controller`, `asp-action`, `asp-area`, `asp-antiforgery`, `asp-route-{param}` | адрес отправки формы + anti-forgery токен (скрытое поле-«секрет», защищающее от CSRF — Cross-Site Request Forgery, подделки межсайтового запроса, когда чужой сайт пытается отправить форму от имени залогиненного пользователя) |
| `<input>` | `asp-for`, `asp-format` | привязка к свойству модели, формат |
| `<label>` | `asp-for` | подпись поля по свойству модели |
| `<select>` | `asp-for`, `asp-items` (`IEnumerable<SelectListItem>`) | список выбора |
| `<span>/<div>` (валидация; `<span>` — строчный (inline) контейнер без смысловой нагрузки, в отличие от блочного `<div>`) | `asp-validation-for`, `asp-validation-summary` (`None`/`ModelOnly`/`All`) | вывод сообщений валидации |
| `<script>/<link>` | `asp-append-version`, `asp-src-include/exclude`, `asp-fallback-*` | версия файла (кэш-бастинг), **CDN** (Content Delivery Network — сеть серверов-зеркал для быстрой раздачи статических файлов ближе к пользователю) + fallback (запасной локальный файл, если CDN недоступен) |
| `<cache>` | — | кэширование содержимого (по умолчанию 20 минут) |
| `<environment>` | `names="Development"` / `"Staging,Production"` | разное содержимое по окружению |

Пример `<select>` из перечисления:
```csharp
public enum Directions { [Display(Name="Вверх")] Up, [Display(Name="Вниз")] Down }
```
```razor
<select asp-for="Direction" asp-items="@Html.GetEnumSelectList<Directions>()"></select>
```
Шаблоны путей файлов: `?` — один любой символ (кроме `/`); `*` — любое число символов (кроме `/`); `**` — любое число символов, включая `/` (рекурсивный поиск).

Подключение библиотек tag-helpers — в `_ViewImports.cshtml`: `@addTagHelper "*, Microsoft.AspNetCore.Mvc.TagHelpers"`.

**Scaffolding** — автоматическая генерация представления по модели (шаблоны `List`, `Create`, `Edit`, `Details`, `Delete`). *Где пригодится: активно используется в ЛР3–ЛР5 для быстрого создания **CRUD**-страниц (CRUD — Create, Read, Update, Delete: базовые операции создания/чтения/изменения/удаления данных).*

> Создание **своих** Tag Helpers (класс `TagHelper`, `TagBuilder`, `LinkGenerator` внутри хелпера) — детально см. в разделе **ЛР7**, там это прямое задание лабы.

---
## Лабораторная работа №3: Работа с данными

### Текст задания

**Цель.** Изучение механизмов обмена данными между контроллером и представлением. **Задача.** Научиться подготавливать данные в контроллере и передавать их представлению. Время — 2 часа. Используется проект из ЛР2.

**3.1. Описание предметной области.** Выберите предметную область. Добавьте в решение новый проект — библиотеку классов .NET **`XXX.Domain`**, в нём папку `Entities`. Для одной сущности создайте класс со свойствами: `ID`, `Название`, `Описание`, `Категория`, числовой параметр для матем. обработки (Цена/Вес/Расстояние), `Изображение` (путь к файлу), `Mime-тип изображения`. В той же папке — класс категории, отношение один-ко-многим (одна категория — много объектов). У `Category` свойство `NormalizedName` — англ. имя в kebab-case для маршрута фильтрации по категории. *(Сквозной пример курса: `Dish` — блюдо, `Category` — категория блюда меню кафе; `Image` и навигационное свойство `Category` в `Dish` — nullable.)*

В `XXX.UI` — ссылка на `XXX.Domain`. В `_ViewImports.cshtml`: `@using XXX.Domain.Entities`. В файл `GlobalUsings.cs` — глобальное подключение `XXX.Domain.Entities`.

**3.2. Вспомогательные классы.** В `XXX.Domain` — папка `Models`. Данные контроллеру будут поставляться сервисами. Опишите:
```csharp
public class ResponseData<T>
{
    public T? Data { get; set; }
    public bool Successfull { get; set; } = true;
    public string? ErrorMessage { get; set; }
    public static ResponseData<T> Success(T data) => new ResponseData<T> { Data = data };
    public static ResponseData<T> Error(string message, T? data = default) =>
        new ResponseData<T> { ErrorMessage = message, Successfull = false, Data = data };
}

public class ListModel<T>
{
    public List<T> Items { get; set; } = new();
    public int CurrentPage { get; set; } = 1;
    public int TotalPages { get; set; } = 1;
}
```
Подключите `GlobalUsings.cs` → `XXX.Domain.Models`.

**3.3. Подготовка к регистрации пользовательских сервисов.** В `XXX.UI/Extensions/HostingExtensions.cs` опишите расширяющий метод для `WebApplicationBuilder`:
```csharp
public static class HostingExtensions
{
    public static void RegisterCustomServices(this WebApplicationBuilder builder) { }
}
```
В `Program.cs`: `builder.RegisterCustomServices();`

**3.4. Описание сервисов.** В `XXX.UI/Services` — папка `CategoryService` с интерфейсом `ICategoryService`:
```csharp
public interface ICategoryService
{
    public Task<ResponseData<List<Category>>> GetCategoryListAsync();
}
```
Реализация `MemoryCategoryService` (имитация реальных данных, коллекция в памяти). Зарегистрировать как **scoped** сервис в `HostingExtensions`. Подключить в `GlobalUsings.cs`.

Аналогично — папка `ProductService`, интерфейс `IProductService`:
```csharp
public interface IProductService
{
    Task<ResponseData<ListModel<Dish>>> GetProductListAsync(string? categoryNormalizedName, int pageNo = 1);
    Task<ResponseData<Dish>> GetProductByIdAsync(int id);
    Task UpdateProductAsync(int id, Dish product, IFormFile? formFile);
    Task DeleteProductAsync(int id);
    Task<ResponseData<Dish>> CreateProductAsync(Dish product, IFormFile? formFile);
}
```
Реализация `MemoryProductService` — данные в коллекции, заполняются в конструкторе; для связи с категориями внедрить `ICategoryService`. Изображения объектов — в `wwwroot/Images`. Зарегистрировать как **scoped**; подключить в `GlobalUsings.cs`.

**3.5. Вывод списка объектов на страницу.** В `Controllers` — контроллер `Product`, внедрить `IProductService` и `ICategoryService`:
```csharp
public async Task<IActionResult> Index()
{
    var productResponse = await _service.GetProductListAsync(null);
    if (!productResponse.Successfull) return NotFound(productResponse.ErrorMessage);
    return View(productResponse.Data.Items);
}
```
Создать представление `Index` (шаблон **List**, модель — объект предметной области, генерация через `dotnet-aspnet-codegenerator` для VS Code/Mac). Заменить вывод имени файла на `<img src="@item.Image"/>`.

**3.6. Оформление списка объектов** — карточки Bootstrap (`card`), по 3 в ряд: изображение, название (Card title), описание, кнопка «Добавить в корзину» → метод `Add` контроллера `Cart` (будет позже), с `Id` объекта и `returnurl` (текущий URL):
```csharp
var request = ViewContext.HttpContext.Request;
var returnUrl = request.Path + request.QueryString.ToUriComponent();
```

**3.7. Фильтр по категориям.** Выпадающий список категорий (Bootstrap Dropdown/Nav-item Dropdown). При выборе в контроллер передаётся `NormalizedName`. Метод контроллера: `Index(string? category)`. В `Index` получить список категорий, текущую категорию, передать в представление через `ViewData`/`ViewBag`. В `MemoryProductService.GetProductListAsync` выполнить **LINQ**-фильтрацию (LINQ — Language Integrated Query, встроенный в C# язык запросов к коллекциям/БД, по возможностям похожий на SQL): `_dishes.Where(d => categoryNormalizedName == null || d.Category.NormalizedName.Equals(categoryNormalizedName)).ToList();`

**3.8. Разбиение на страницы.** `Index` выводит не более 3 объектов; номер страницы — параметр `pageNo`; размер страницы — в `appsettings.json` (`"ItemsPerPage": 3`). Модель представления — `ListModel<Dish>`. Метод: `Index(string? category, int pageNo = 1)`. В `MemoryProductService` внедрить `IConfiguration`, получить размер страницы, вычислить `TotalPages`, выбрать нужную страницу через LINQ `Skip`/`Take`.

**3.9. Кнопки переключения страниц** — компонент Bootstrap Pagination, класс `active` для текущей страницы; корректно ограничить кнопки «Предыдущая»/«Следующая» границами `[1, TotalPages]`. Имя категории для ссылок получить из `Request.Query["category"]`.

**4. Контрольные вопросы:** 1. Как зарегистрировать сервис? 2. Чем Transient отличается от Scoped? 3. Как внедрить сервис в Action? 4. Как передать данные от клиента в метод контроллера? 5. Где Model Binding ищет нужные значения? 6. Как получить данные из `appsettings.json`? 7. Как передать данные через `IOptions`? 8. Как прочитать значение из строки запроса? 9. Как в `<a>` передать доп. данные через tag-helper? 10. Что такое «Явное выражение Razor»?

### Теория для выполнения ЛР3

#### Контроллеры и результаты действий (Action Results)

*Определение.* **Контроллер** реагирует на действия пользователя и взаимодействует с представлением, моделью и уровнем доступа к данным (DAL). Это класс с методами-**действиями (Actions)**, которые вызывает маршрутизация. По соглашению: находится в `Controllers`, наследуется от `Microsoft.AspNetCore.Mvc.Controller`. Формально контроллер — публичный класс, у которого выполняется хотя бы одно: имя с суффиксом `Controller`, наследование от класса с суффиксом `Controller`, атрибут `[Controller]`.

В «правильной» архитектуре контроллер **не** содержит прямого доступа к данным/бизнес-логике — делегирует сервисам (см. ниже DI). По умолчанию адрес `{controller}/{action}`: `http://xxx/Home/Index` → метод `Index` класса `HomeController`.

```csharp
public class HomeController : Controller
{
    public IActionResult Index() => View();
}
```

Действия обычно возвращают `IActionResult` (или `Task<IActionResult>`). *Где пригодится: база для ЛР3–ЛР10, вопрос на аттестации* **[Атт. №22]**.

**Типы результатов действий** (таблица для запоминания):

| Категория | Методы / классы | Что возвращают |
|---|---|---|
| Пустое тело, код состояния | `BadRequest()` (400), `NotFound()` (404), `Ok()` (200) | только `StatusCodeResult`; при передаче объекта — уже не просто код, а содержимое |
| Перенаправление | `RedirectToAction("Action")`, `Redirect("Url")` | `RedirectResult`/`RedirectToActionResult` — отличаются наличием заголовка `Location` |
| Непустое тело | `View()`, `Json(value)`, `File(path)` | HTML-представление / JSON / файл |
| Согласование по `Accept` | `Ok(value)` | формат по заголовку `Accept` запроса (JSON/XML) |

*(Здесь и далее: **JSON** — JavaScript Object Notation, текстовый формат данных в виде пар «ключ: значение», основной формат обмена данными между API и клиентом.)*

#### Внедрение зависимостей (Dependency Injection, DI)

*Определение.* **DI** — шаблон проектирования, при котором объект получает свои зависимости извне (через конструктор/параметр/сервис), а не создаёт их сам; реализуется через встроенный **IoC**-контейнер (**IoC** — Inversion of Control, «инверсия управления»: не сам объект создаёт/ищет свои зависимости, а внешний контейнер создаёт и передаёт их ему) ASP.NET Core. *Где пригодится: без DI не зарегистрировать ни один сервис курса — ЛР3 (`ICategoryService`/`IProductService`), ЛР6 (сервис аутентификации), ЛР8 (Cart-сервис), ЛР9 (тестируемость через интерфейсы); в реальной разработке DI — основа архитектуры любого ASP.NET Core приложения. Частый вопрос на аттестации* **[Атт. №20, №21]**.

Регистрация в `Program.cs`:
```csharp
builder.Services.AddTransient<IDataService, EfDataService>();
builder.Services.AddScoped<ICategoryService, MemoryCategoryService>();
builder.Services.AddSingleton<HelperClass>();
```

**Жизненные циклы сервисов** (таблица — частый вопрос на экзамене):

| Метод | Когда создаётся новый экземпляр | Типичное применение |
|---|---|---|
| `AddTransient` | при каждом разрешении зависимости | лёгкие stateless-сервисы |
| `AddScoped` | один экземпляр на один HTTP-запрос | `DbContext`, сервисы работы с БД/корзиной |
| `AddSingleton` | один экземпляр на всё время жизни приложения | конфигурация, кэш, `LinkGenerator` |

Способы внедрения:
```csharp
// через конструктор
public HomeController(IDataService dataService, ILogger<HomeController> logger) { ... }
// в действие
public IActionResult Index([FromServices] IDataService dataService) { ... }
```
Получение сервиса вне DI-цепочки (вручную из контейнера):
```csharp
var dataService = app.Services.GetService<IDataService>();       // null, если сервис не найден
var dataService2 = app.Services.GetRequiredService<IDataService>(); // исключение, если не найден
```
`GetRequiredService` предпочтительнее — не нужно дублировать проверку на `null` в каждом месте вызова, ошибка (`InvalidOperationException`) явно указывает на проблему, вместо позднего `NullReferenceException`.

Получение **Scoped**-сервиса из Singleton-контекста (например, при заполнении БД в `Program.cs`) — через явно созданную область (`scope`):
```csharp
using var scope = app.Services.CreateScope();
using var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
```
*Где пригодится: ровно этот паттерн используется в ЛР4 (`DbInitializer.SeedData`).*

#### Model Binding — привязка данных к методу контроллера

*Определение.* **Model Binding** — механизм, автоматически извлекающий данные запроса (маршрут, форма, строка запроса, тело, файлы) и преобразующий их в параметры действия/свойства модели, чтобы не писать этот код вручную. *Где пригодится: любая лаба с формами/API (ЛР3–ЛР8); ключевая тема аттестации* **[Атт. №23]**.

```csharp
public IActionResult ShowValues(int id, string name) { return View(); }
// https://host/home/showvalues/10?name=bob → id из RouteData, name из строки запроса
```
Источники по умолчанию: поля формы, тело запроса (для контроллеров с `[ApiController]`), данные маршрута, строка запроса, загруженные файлы. Для сложного типа: публичный конструктор без параметров + публичные доступные для записи свойства; поиск имени — по шаблону `prefix.property_name`, если не найдено — просто `property_name`.

Явное указание источника — атрибуты: `[FromQuery]`, `[FromRoute]`, `[FromForm]`, `[FromBody]`, `[FromHeader]`.
```csharp
public IActionResult EditBook([FromForm] Book book) { ... }
public IActionResult ShowCompanies(string[] companies) { ... } // массив из нескольких <input name="companies">
public IActionResult CreateBook([Bind("Name,Author")] Book book) { ... } // ограничить привязываемые свойства
```
Типы **`record`** поддерживаются моделью привязки через единственный конструктор: `public record BookDto(int Id, string Name, string Author);`

**Привязка вручную**: `TryUpdateModelAsync<Book>(newBook, "Book", b => b.Name, b => b.Author)` — возвращает `false` при сбое.

**Пользовательская привязка модели** — реализация `IModelBinder`:
```csharp
public class BookIdBinder : IModelBinder
{
    public Task BindModelAsync(ModelBindingContext bindingContext)
    {
        var value = bindingContext.ValueProvider.GetValue(bindingContext.ModelName).FirstValue;
        if (!int.TryParse(value, out var id)) { /* ошибка */ }
        var model = _dbContext.Books.Find(id);
        bindingContext.Result = ModelBindingResult.Success(model);
        return Task.CompletedTask;
    }
}
// применение: [ModelBinder(typeof(BookIdBinder), Name = "id")] Book book
```
Поставщик биндеров — `IModelBinderProvider`, регистрация: `opt.ModelBinderProviders.Insert(0, new BookIdBinderProvider())`.

#### Аннотации и валидация данных

*Определение.* **Data Annotations** (`System.ComponentModel.DataAnnotations`) — атрибуты, задающие отображение и правила проверки свойств модели. *Где пригодится: обязательны в формах создания/редактирования (ЛР3 создание блюда, ЛР5 Razor Pages CRUD, ЛР6 регистрация пользователя)* **[Атт. №25]**.

Отображение: `[Display(Name="Название книги")]`, `[ScaffoldColumn(false)]` (скрыть при Scaffold), `[DataType(DataType.Password|Url|EmailAddress|Currency|...)]`, `[DisplayFormat(DataFormatString="{0:dd-MM-yyyy}", ApplyFormatInEditMode=true)]`.

Валидация: `[Required]`, `[StringLength(160, MinimumLength=3)]`, `[RegularExpression(@"...")]`, `[EmailAddress]`, `[Range(35,44)]` / `[Range(typeof(decimal), "0.00", "49.99")]`, `[Compare("Email")]`. Свои сообщения: `[Required(ErrorMessage = "Поле обязательно")]`.

**Валидация на клиенте** — скрипты `jquery.validate.min.js` + `jquery.validate.unobtrusive.min.js` (подключаются партиалом `_ValidationScriptsPartial.cshtml` в секции `Scripts`); предотвращает отправку формы, пока данные невалидны, экономит обращения к серверу. Tag-helpers: `asp-validation-for`, `asp-validation-summary="ModelOnly|All|None"`.

**Валидация на сервере** — выполняется **всегда**, независимо от клиентской. Объект `ModelState` хранит ошибки: 1) от **привязки модели** (ошибки конвертации типов, напр. `"x"` в `int`), 2) от **валидации модели** (нарушение бизнес-правил после привязки). Проверка:
```csharp
[HttpPost] [ValidateAntiForgeryToken]
public async Task<IActionResult> Edit(int id, [Bind("BookId,Name,Author")] Book book)
{
    if (ModelState.IsValid) { /* сохранить */ return RedirectToAction(nameof(Index)); }
    return View(book); // повторный показ формы с ошибками
}
```
Принудительный повторный запуск валидации — `TryValidateModel(book)`. Добавление ошибки вручную: `ModelState.AddModelError(nameof(book.Name), "Сообщение");`

Атрибут `[Remote(action:"VerifyBook", controller:"Books")]` — проверка на клиенте через AJAX-вызов серверного метода (метод возвращает `Json(true)` при успехе или `Json("сообщение об ошибке")`).

Пользовательская валидация: атрибут (наследник `ValidationAttribute`, переопределяет `IsValid`) или интерфейс `IValidatableObject` (метод `Validate`, может вернуть несколько ошибок сразу, для нескольких полей).

---

## Лабораторная работа №4: Работа с REST API

### Текст задания

**Цель.** Знакомство с принципом работы сервисов REST. **Задача.** Научиться создавать REST API сервисы и взаимодействовать с ними из приложений. Время — 2 часа.

**3.1. Создание проекта.** Добавьте в решение проект **ASP NET Core Web API**, имя `XXX.Api`, ссылку на `XXX.Domain`. Порты **7002/5002** — изменить в `launchSettings.json`. Пакеты **NuGet** (NuGet — стандартный менеджер пакетов для .NET, аналог npm/pip: через него подключаются сторонние библиотеки): `Microsoft.EntityFrameworkCore`, `Microsoft.EntityFrameworkCore.Tools`, `Microsoft.VisualStudio.Web.CodeGeneration.Design`. Папка `wwwroot/Images` (скопировать изображения из основного проекта). **Внимание:** для доступа к файлам изображений в `Program.cs` добавить Middleware статических файлов. В настройках решения запускать оба проекта одновременно (VS) / вручную (VS Code). В `launchSettings.json` закомментировать `"launchBrowser": true`.

**3.2. Выбор БД.** Варианты: MS SQL Server Express, SQLite, PostgreSQL, MySQL (в примерах — PostgreSQL). Для SQLite достаточно пакета `Microsoft.EntityFrameworkCore.Sqlite`; для PostgreSQL — `Aspire.Npgsql.EntityFrameworkCore.PostgreSQL`.

**3.3. Установка сервера PostgreSQL** — можно локально (postgresql.org, pgAdmin4) или в Docker. Пример `compose.yaml` — файла **Docker Compose** (инструмента для описания и одновременного запуска нескольких контейнеров одной командой) с сервисами `postgres` и `pgadmin` (переменные — в `postgres.env`: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `PGADMIN_DEFAULT_EMAIL`, `PGADMIN_DEFAULT_PASSWORD`). Запуск: `docker compose --env-file postgres.env up -d`.

**3.4. Создание контекста БД.** В `XXX.Api/Data` — класс `AppDbContext` (конструктор принимает `DbContextOptions<AppDbContext>`), свойства `DbSet<T>` для сущностей `XXX.Domain`.

**3.5. Начальная миграция.** В `appsettings.json` — строка подключения:
```json
"ConnectionStrings": { "Postgres": "Server=127.0.0.1; Port=5432; Database=MenuDvb_V01; User Id=admin; Password=123456" }
```
Регистрация контекста: `options.UseNpgsql(connString)`. Создать начальную миграцию, выполнить `Update` (создание БД), проверить в pgAdmin.

**3.6. Заполнение начальными данными.** В `Data/DbInitializer.cs` — статический `public static async Task SeedData(WebApplication app)`. Данные — как в `MemoryProductService`, но: записываются в контекст БД с `SaveChanges`; `Id` не указывается (назначает БД); изображения — в `wwwroot/Images` проекта `XXX.Api`. Получение Scoped-контекста БД из `WebApplication app`:
```csharp
using var scope = app.Services.CreateScope();
var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
await context.Database.MigrateAsync();
```
Адрес приложения `XXX.Api` (например `https://localhost:7002`) для формирования URL изображения — вынести в `appsettings.json`, получать через `app.Configuration`.

**3.7. Создание контроллера API.** В `Controllers` — REST API контроллер для категорий: VS → **Add → New scaffolded item → API → API controller with actions, using EntityFramework**; VS Code/Mac — пакеты `Microsoft.VisualStudio.Web.CodeGeneration.Design`, `Microsoft.EntityFrameworkCore.Design`, `Microsoft.EntityFrameworkCore.SqlServer`, затем:
```
dotnet aspnet-codegenerator controller -name [ИМЯ] -async -api -m [Модель] -dc [Контекст] -outDir Controllers
```
В `Program.cs`: `builder.Services.AddControllers();` и `app.MapControllers();`.

**3.8. Создание Minimal API.** Для объектов предметной области — папка `EndPoints`, **Add → New scaffolded item → API → API with read/write endpoints, using Entity Framework**. Регистрация: `app.MapDishEndpoints();`. Через PowerShell (если сущность в другой сборке):
```
dotnet aspnet-codegenerator minimalapi -m [Модель] -e [ИмяКлассаEndpoints] -dc [Контекст] -dbProvider postgres
```

**3.9. Проверка API** — запросы через адресную строку (для GET) или **Postman/Insomnia** (программы-клиенты для ручной отправки HTTP-запросов и просмотра ответов, удобны для тестирования API без браузера) (POST/PUT/DELETE); убедиться, что данные приходят в JSON.

**3.10. Доработка `group.MapGet("/")`.** Конечная точка должна возвращать `ResponseData` (см. ЛР3 п.3.2), с фильтрацией по категории и постраничным выводом (номер/размер страницы — из строки запроса, имя категории — сегмент маршрута), ограничение максимального размера страницы:
```csharp
private readonly int _maxPageSize = 20;
```
Так как конечная точка не должна содержать много кода — используется **библиотека MediatR** (паттерн **CQRS** — Command Query Responsibility Segregation, разделение операций изменения (Command) и чтения (Query) данных: запрос + обработчик):
```csharp
public sealed record GetListOfProducts(string? categoryNormalizedName, int pageNo = 1, int pageSize = 3)
    : IRequest<ResponseData<ListModel<Dish>>>;

public class GetListOfProductsHandler(AppDbContext db)
    : IRequestHandler<GetListOfProducts, ResponseData<ListModel<Dish>>>
{
    private readonly int _maxPageSize = 20;
    // ...
}
```
```csharp
group.MapGet("/{category:alpha?}", async (IMediator mediator, string? category, int pageNo = 1) =>
{
    var data = await mediator.Send(new GetListOfProducts(category, pageNo));
    return TypedResults.Ok(data);
}).WithName("GetAllDishes").WithOpenApi(); // WithOpenApi — регистрирует эндпоинт в OpenAPI-описании (стандарт машиночитаемого описания REST API, на основе которого строится автогенерируемая документация; раньше известный как Swagger)
```

**3.13. Вертикальные срезы (Vertical Slice Architecture) — необязательно.** Получение списка и добавление объекта реализовать как «вертикальные срезы» (весь функционал одной фичи — в одном файле/папке). Загрузить **FluentValidation** (сторонняя библиотека для описания правил валидации в виде кода, а не атрибутов) `FluentValidation.DependencyInjectionExtensions`, зарегистрировать: `builder.Services.AddValidatorsFromAssembly(typeof(Program).Assembly);`. Интерфейс маркер:
```csharp
public interface IEndpoint { void MapEndpoint(IEndpointRouteBuilder app); }
```
Папка `Features`, класс среза (запись **DTO** — Data Transfer Object, простой объект только для переноса данных между слоями приложения, без бизнес-логики + валидатор + статический `Handler` + класс `Endpoint : IEndpoint`), пример `AddNewProduct` — см. полный код в тексте лабы (валидатор через `FluentValidation.AbstractValidator<T>`, проверка категории, `TypedResults.Created(...)`). Регистрация всех `IEndpoint` через рефлексию — расширение `WebApplication.MapIEndpoints()`, ищет все нефинальные/неабстрактные типы, реализующие `IEndpoint`, создаёт экземпляр и вызывает `MapEndpoint`.

**4. Контрольные вопросы:** 1. Чем контроллер API отличается от обычного? 2. Как выбирается Action контроллера API? 3. Где и в каком виде передаются данные в контроллер API? 4. Как получить Scoped-сервис в коде? 5. Что могут возвращать методы контроллера API? 6. Что такое Minimal API? 7. Как зарегистрировать конечную точку Minimal API? 8. Как зарегистрировать группу конечных точек?

### Теория для выполнения ЛР4

#### REST — принципы и свойства

*Определение.* **REST** (REpresentation State Transfer) — архитектурный стиль ПО для распределённых систем, предложенный Роем Филдингом в 2000 г. в докторской диссертации. **REST не является протоколом или стандартом.** *Где пригодится: основа ЛР4 и всех дальнейших API-лаб; сравнение с GraphQL/gRPC — частый вопрос на аттестации.*

Семь описательных свойств REST-архитектуры: производительность, масштабируемость, простота унифицированного интерфейса, модифицируемость компонентов, видимость (чёткая связь между компонентами), переносимость кода, надёжность.

Всё в REST — это **ресурсы** (объекты со своими данными). Ограничения ресурсов: идентификация ресурсов как «запросов» независимо от языка/интерпретации; манипулирование ресурсами через представление данных; **самоописательные сообщения**; **гипермедиа** (**HATEOAS** — Hypermedia As The Engine Of Application State, «гипермедиа как двигатель состояния приложения») — клиент меняет состояние системы только через действия, динамически определённые в гипермедиа сервера, а не предполагает наличие операции заранее.

**Базовые принципы REST**: разделение клиент-сервер (независимая разработка); **stateless** (каждая пара запрос-ответ независима от других); cacheable (ответы явно помечают кэшируемость); многоуровневые системы (посредники прозрачны для клиента); код по требованию (необязательное ограничение — сервер может передавать исполняемый код, например JS).

**HTTP-методы REST** (глаголы действия):

| Метод | Действие |
|---|---|
| `POST` | добавить ресурс |
| `GET` | извлечь данные без изменения |
| `PUT` | сохранить/обновить ресурс по URI |
| `DELETE` | удалить указанный ресурс |
| `PATCH` | частично изменить ресурс |

#### REST API контроллеры vs Minimal API

Контроллеры API похожи на обычные, но методы отдают **объекты данных без HTML**. Не все клиенты API — браузеры.
```csharp
[Route("api/[controller]")]
[ApiController]
public class DogsController : ControllerBase
{
    [HttpGet] public IEnumerable<Dog> GetDogs() => _context.Dogs;
    [HttpGet("{id}")] public async Task<IActionResult> GetDog([FromRoute] int id) { ... }
    [HttpPut("{id}")] public async Task<IActionResult> PutDog([FromRoute] int id, [FromBody] Dog dog) { ... }
}
```
*Определение.* Атрибут **`[ApiController]`** включает автоматическую проверку `ModelState` (400 при невалидных данных без ручной проверки), вывод источника привязки по умолчанию из тела для сложных типов и др. удобства для API. *Где пригодится: ЛР4/ЛР6 — все REST-контроллеры* **[Атт. №40, №41]**.

Формат ответа: строка передаётся клиенту как есть; C#-объект сериализуется в JSON. Методы для кода состояния: `StatusCode`, `Ok` (200), `Created`/`CreatedAtAction`/`CreatedAtRoute` (201, заголовок `Location`), `BadRequest` (400), `Unauthorized` (401), `NotFound` (404).

*Определение.* **Minimal API** — упрощённый способ разработки JSON API без классов контроллеров: меньше «церемоний» обработки запроса (маршрутизация → инициализация контроллера → привязка/фильтры → фильтры результата, 17 шагов у классического MVC), примерно на **40% быстрее** возвращает простой ответ и на ~15 МБ меньше расходует памяти. *Где пригодится: центральная тема ЛР4 (`EndPoints`) и ЛР8 (кэшируемая конечная точка).* **[Атт. №6, №40]**
```csharp
app.MapGet("/", () => "Hello World!");
app.MapGet("/api/books/", async (ApplicationDbContext db) => await db.Books.ToListAsync())
   .WithName("GetAllBooks").WithOpenApi();

public static class BookEndpoints
{
    public static void MapBookEndpoints(this IEndpointRouteBuilder routes)
    {
        var group = routes.MapGroup("/api/Book").WithTags(nameof(Book));
        group.MapGet("/", async (ApplicationDbContext db) => { ... });
        group.MapGet("/{id}", async Task<Results<Ok<Book>, NotFound>> (int id, ApplicationDbContext db) => { ... });
    }
}
// в Program.cs: app.MapBookEndpoints();
```

> **MediatR / CQRS** — паттерн «команда/запрос + обработчик», убирающий бизнес-логику из тела endpoint'а/контроллера в отдельный класс-обработчик (`IRequestHandler<TRequest,TResponse>`); сквозная тема, начинается в ЛР4 (`GetListOfProducts`) — см. также ЛР5 (`SaveImage`). *Где пригодится: чистая архитектура реальных API-проектов; сдаётся в ЛР4, используется дальше.*

---

## Лабораторная работа №5: Razor Pages. Передача файлов

### Текст задания

**Цель.** Знакомство со сценариями построения приложений на страницах (Razor Pages). Знакомство с передачей файлов клиент→сервер, сервер→API. **Задача.** Научиться создавать страницы Razor, передавать файлы на сервер и в REST-сервис. Время — 2 часа.

**Часть 1. Страницы администратора.**

**3.1. Постановка задачи.** В `XXX.UI` создать страницы администратора (CRUD для объектов предметной области — не для категорий) по сценарию **Razor Pages**, размещённые в **области (Area)** `Admin` (см. ЛР2, п.3.1).

**3.2. Подготовка проекта.** Чтобы Scaffold мог сгенерировать страницы, во временный `DbContext` (напрямую в `XXX.UI`) нужно **временно** описать контекст БД:
```csharp
public class TempDbContext : DbContext
{
    public DbSet<Dish> Dishes { get; set; }
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.UseSqlite("");
}
```
(пакеты `Microsoft.EntityFrameworkCore` + провайдер, например `Microsoft.EntityFrameworkCore.Sqlite`). После генерации страниц контекст можно удалить.

**3.3. Создание страниц администратора.** Сгенерировать Razor Pages для CRUD через Scaffold. В коде моделей всех страниц **заменить** использование контекста БД на использование `IProductService`. Скопировать `_ViewImports.cshtml`/`_ViewStart.cshtml` в папку страниц админа. На странице Index вместо имени файла изображения — тег вывода изображения; оформить пагинацию иконками/стилями Bootstrap. В `Program.cs`: `builder.Services.AddRazorPages();` и `app.MapRazorPages();`. Проверить работу страниц администрирования.

**Часть 2. Передача файлов.**

**4.1. Постановка задачи.** Страницы создания/редактирования должны передавать изображение объекта. Изображения сохраняются в `XXX.API/wwwroot/Images`. Имена файлов — случайные (без дублей). При удалении/замене изображения — старый файл удаляется. Соглашение: имя поля файла в HTTP — `file`.

**4.2. Use-case сохранения файла (`XXX.API`).** В `Use-Cases/SaveImage.cs`:
```csharp
public sealed record SaveImage(IFormFile file) : IRequest<string>;

public class SaveImageHandler(IWebHostEnvironment env, IHttpContextAccessor httpContextAccessor)
    : IRequestHandler<SaveImage, string>
{
    public Task<string> Handle(SaveImage request, CancellationToken cancellationToken) { /* ... */ }
}
```
`IWebHostEnvironment` — путь к `wwwroot`; `IHttpContextAccessor` — для формирования абсолютного URL изображения на хосте `XXX.API`. Запрос возвращает URL сохранённого изображения.

**4.3. Конечная точка API для объекта с файлом изображения.** Добавить в `XXX.API/Program.cs` Middleware статических файлов (`app.MapStaticAssets()`/`app.UseStaticFiles()`). Данные с файлом передаются как **`multipart/form-data`** (а не JSON тела); нужно отключить Antiforgery для группы: `.DisableAntiforgery()`. Пример:
```csharp
group.MapPost("/", async ([FromForm] string dish, [FromForm] IFormFile? file, AppDbContext db, IMediator mediator) =>
{
    var newDish = JsonSerializer.Deserialize<Dish>(dish);
    if (file != null) newDish.Image = await mediator.Send(new SaveImage(file));
    db.Dishes.Add(newDish);
    await db.SaveChangesAsync();
    return TypedResults.Created($"/api/Dish/{newDish.Id}", newDish);
}).WithName("CreateDish").WithOpenApi();
```

**4.4. Доработка `ApiProductService` (`XXX.UI`).** Методы создания/редактирования должны передавать файл в запросе к API через `MultipartFormDataContent`:
```csharp
var content = new MultipartFormDataContent();
if (formFile != null)
{
    var streamContent = new StreamContent(formFile.OpenReadStream());
    content.Add(streamContent, "file", formFile.FileName);
}
content.Add(new StringContent(JsonSerializer.Serialize(product)), "dish");
request.Content = content;
var response = await _httpClient.SendAsync(request, CancellationToken.None);
```

**4.5. Доработка страниц Create/Update.** В модели страницы:
```csharp
[BindProperty] public IFormFile? Image { get; set; }
```
Передать `Image` в сервис (`await _service.CreateProduct(dish, Image);`). В разметке — тег выбора файла с `asp-for="Image"`, форма с `enctype="multipart/form-data"`, оформление классами Bootstrap (`form-control` для `<input type="file">`).

**5. Контрольные вопросы:** 1. Чем Razor Pages отличается от MVC? 2. Что такое модель страницы Razor? 3. Как обрабатываются запросы к страницам Razor? 4. Как в разметке страницы получить доступ к свойствам модели страницы? 5. Как привязать данные формы к модели страницы? 6. Для чего `@page`? 7. Какой URL у страницы `Register`? 8. Чем отличается адрес `/index` от `./index`? 9. Как указать доп. сегмент маршрута (например `id`)? 10. Какой Middleware передаёт статический контент клиенту? 11. Какой интерфейс реализует объект файла при передаче клиент→сервер? 12. Что такое поставщики файлов (FileProvider)? 13. Как получить путь к `wwwroot`? 14. В каком виде представлен переданный файл на сервере? 15. Какой атрибут `<form>` нужен для передачи файлов?

### Теория для выполнения ЛР5

#### Razor Pages

*Определение.* **Razor Pages** — сценарий ASP.NET Core, упрощающий разработку страничных (page-centric) приложений; каждая страница — пара файлов: `.cshtml` (разметка + Razor) и `.cshtml.cs` (**модель страницы**, класс `PageModel`, обрабатывает события страницы). Располагаются в `/Pages` или `/Areas/{Area}/Pages`. *Где пригодится: центральная тема ЛР5 (админка); используется на аттестации* **[Атт. №45, №46, №47]**.

Регистрация: `builder.Services.AddRazorPages();` / `app.MapRazorPages();`.

Директива **`@page`** превращает файл в MVC-действие — страница обрабатывает запрос напрямую, без контроллера; должна быть **первой** директивой Razor на странице. Директива **`@model`** задаёт класс модели страницы (в `.cshtml.cs`).
```razor
@page
@model ХХХ.Areas.Admin.Pages.IndexModel
```
```csharp
namespace ХХХ.Areas.Admin.Pages { public class IndexModel : PageModel { public void OnGet() { } } }
```
Чтение данных модели в разметке — через `@Model.Свойство`.

**Обработка запросов** — методы, имя которых совпадает с HTTP-методом: `OnGet()`, `OnGetAsync()`, `OnPostAsync(int? id)`, и т.д. Несколько обработчиков на странице различаются суффиксом (`OnPostEdit`, `OnPostCreate`), выбор — через `asp-page-handler="Create"` (в URL — `?handler=Create`).

**Привязка данных** — атрибут `[BindProperty]` (для всех методов кроме GET; для GET нужно `[BindProperty(SupportsGet = true)]`):
```csharp
[BindProperty] public InputModel Input { get; set; }
```
`[ViewData]`/`[TempData]` — как и в MVC, но объявляются как атрибуты над свойством модели страницы.

**Маршрутизация страниц.** Первая строка может содержать `@page "{id:int}"` — шаблон маршрута. Относительное имя страницы (`./Index`) — используется для URL внутри той же папки; абсолютное (`/Index`) — от корня `Pages`. `Url.Page("./Index")`, `<a asp-page="./Index">`, `RedirectToPage("./Index")`.

**Razor Pages vs MVC** (таблица — по мнению автора курса + документации Microsoft):

| | Razor Pages | MVC контроллеры |
|---|---|---|
| Рекомендуется для | серверных приложений, где рендерится представление | Web API |
| Организация кода | по **функциям** (вся страница — в одном месте) | по **типу** (контроллер / представление / view-model раздельно) |
| Данные для View | свойства модели страницы (`PageModel`) | явные **view-model**'и (ViewModel — отдельный класс, специально созданный для передачи данных в конкретное представление, в отличие от передачи «сырой» модели из БД), передаваемые в `View(model)` |
| Один класс — сколько действий | одна страница = одна модель | контроллер может разрастись, много действий, много зависимостей |
| Частичные AJAX-обновления | сложнее | удобнее (можно вернуть `PartialView`) |
| Миграция существующего MVC | не имеет смысла переписывать | оставить как есть |

*Где пригодится: прямой вопрос на аттестации «чем Razor Pages отличается от MVC»* **[Атт. №45]**.

#### Области (Areas)

*Определение.* **Areas** — механизм группировки связанного функционала: пространство имён для маршрутизации + структура папок для представлений/страниц. Позволяют разделить приложение на независимые функциональные части (у каждой — свои контроллеры/страницы/представления/модели). *Где пригодится: ровно то, что просит ЛР5 — админка в area `Admin`* **[Атт. №44]**.

Контроллеры — атрибут `[Area("Admin")]`; маршрут: `app.MapAreaControllerRoute(name: "AreaAdmin", areaName: "Admin", pattern: "Admin/{controller=Home}/{action=Index}/{id?}");` или общий шаблон `"{area:exists}/{controller=Home}/{action=Index}/{id?}"`. Ссылка: `<a asp-area="Admin" asp-controller="Home" asp-action="Index">`. Для Razor Pages areas — `Areas/{Area}/Pages`.

Общий `_ViewStart.cshtml` для всех областей — держать в корне приложения; `_ViewImports.cshtml` — либо в корне (применяется ко всем), либо копировать в папку области.

#### Работа со статическими файлами и передача файлов

**Статические файлы** (HTML/CSS/изображения/JS) отдаются напрямую клиенту из `wwwroot` (Web Root). Нужен пакет `Microsoft.AspNetCore.StaticFiles` (уже входит в SDK) и `app.UseStaticFiles();` в `Program.cs`. Другая папка вместо `wwwroot` (например `node_modules`):
```csharp
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(Path.Combine(builder.Environment.ContentRootPath, "node_modules")),
    RequestPath = "/npm"
});
```

*Определение.* **Поставщики файлов (File Providers)** — абстракция над файловой системой, интерфейс **`IFileProvider`** (методы получения `IFileInfo`, `IDirectoryContents`, подписка на изменения через `IChangeToken`). `PhysicalFileProvider` — обёртка над `System.IO.File`, ограничивает доступ конкретным каталогом. *Где пригодится: явно спрашивается в контрольных вопросах ЛР5.*

**Передача файла клиент → сервер.** Форма должна иметь `enctype="multipart/form-data"`. Файл привязывается через Model Binding к типу, реализующему **`IFormFile`**:
```csharp
public interface IFormFile
{
    string ContentType { get; } string FileName { get; } long Length { get; }
    Stream OpenReadStream();
    Task CopyToAsync(Stream target, CancellationToken cancellationToken = default);
}
```
```csharp
[HttpPost]
public async Task<IActionResult> Upload(IFormFile uploadedFile)
{
    var path = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", uploadedFile.FileName);
    using var stream = new FileStream(path, FileMode.Create);
    await uploadedFile.CopyToAsync(stream);
    return Ok();
}
```
*Где пригодится: прямое задание ЛР5 (`4.2`–`4.5`), также ЛР6 (аватар пользователя).* **[Атт. №29]**

**Передача файла сервер → клиенту** — абстрактный класс `FileResult`, реализации: `FileContentResult` (массив байт), `VirtualFileResult` (виртуальный путь), `FileStreamResult` (поток `Stream`), `PhysicalFileResult` (реальный физический путь). *Где пригодится: если нужно отдавать файлы не напрямую через статику, а с проверкой доступа* **[Атт. №30]**.
```csharp
[Route("Image/{id?}")]
public IActionResult GetImage([FromServices] IWebHostEnvironment env)
{
    var provider = env.WebRootFileProvider;
    var fInfo = provider.GetFileInfo(Path.Combine("images", "Picture1.jpg"));
    var extProvider = new FileExtensionContentTypeProvider();
    return File(fInfo.CreateReadStream(), extProvider.Mappings[Path.GetExtension(fInfo.Name)]);
}
```

---

## Лабораторная работа №6: Аутентификация и авторизация

### Текст задания

**Цель.** Знакомство с механизмом аутентификации и авторизации в ASP.NET Core. **Задача.** Научиться выполнять аутентификацию/авторизацию на удалённом сервере, ограничивать доступ к API для незарегистрированных пользователей. Время — 6 часов (3 занятия).

**3.1. Выбор сервера аутентификации.** Из-за ограниченного доступа к готовым облачным сервисам (Azure AD и т.п.) используется собственный сервер — **Keycloak** (`https://www.keycloak.org/`). Схема: `Клиент → (аутентификация) → Сервер аутентификации → (4. запрос на подтверждение токена, 5. ответ с подтверждением) → API-сервис`.

**3.2. Термины Keycloak.** **Realm** — домен безопасности/администрирования (пользователи, приложения, роли, изолированно). **API scope** (**OAuth 2.0** — открытый протокол авторизации, позволяющий приложению получить ограниченный доступ к ресурсу от имени пользователя, не зная его пароль) — область доступа, запрашиваемая клиентом. **Client** — приложение/сервис, взаимодействующее с Keycloak для аутентификации/авторизации; создаётся внутри Realm.

**4.2. Установка Keycloak и БД (локально)** — OpenJDK + дистрибутив Keycloak (`kc.bat start-dev`), подключение PostgreSQL (создать БД и пользователя, раскомментировать/настроить строки подключения в конфиге keycloak). **4.3. В Docker** — добавить в `compose.yaml` сервис `keycloak` (образ `quay.io/keycloak/keycloak`, `command: start-dev`, переменные `KC_DB`, `KC_DB_URL_HOST`, `KEYCLOAK_ADMIN(_PASSWORD)`, порт `8080:8080`), в `.env` — соответствующие переменные. Т.к. `docker compose down` не пересоздаёт БД — создать вручную в pgAdmin.

**4.4. Настройка Keycloak.**
- **Realm**: создать (имя — фамилия), на вкладке Login — использовать email как имя пользователя; на User Profile — удалить `FirstName`/`LastName`, добавить необязательный атрибут `avatar`.
- **Client**: тип **OpenID Connect** (OIDC — надстройка над OAuth 2.0, добавляющая полноценную аутентификацию пользователя, а не только выдачу прав доступа), имя `[Фамилия]UiClient`; разрешить Client Authentication, Authorization, Implicit Flow; указать URL приложения; на вкладке Credentials — `Client Secret`; на «Service Account roles» назначить `manage-users` (чтобы приложение могло регистрировать пользователей).
- **Передача атрибута avatar в JWT** (**JWT** — JSON Web Token, компактный подписанный токен, в котором закодированы утверждения (claims) о пользователе; сервер API проверяет подпись и доверяет данным внутри токена, не обращаясь к БД пользователей): Client scopes → `xxx-dedicated` → Add mapper → By configuration → User Attribute → `avatar`.
- **Роль администратора**: на вкладке Roles добавить `POWER-USER`.
- **Передача роли как Claim**: Client scopes → `roles` → Mappers → Add mapper → By configuration → «User client role», `Multivalued=On`, `Token claim name=role`, `Add to access token=On`.
- **Пользователи**: создать обычного и администратора (email подтверждён, пароль не временный); администратору назначить роль `POWER-USER` (Role Mapping).

**4.5. Проверка сервера Keycloak** (Postman/Insomnia). `POST` на `token_endpoint`, тело формы: `client_id`, `username` (email), `grant_type=password`, `password`, `client_secret` → получить JWT-токен **пользователя**. С `grant_type=client_credentials` (без `username`/`password`) → JWT-токен **клиента**.

**4.6. Настройка `XXX.API`.** Пакет `Microsoft.AspNetCore.Authentication.JwtBearer`. В `appsettings.json`:
```json
"AuthServer": { "Host": "http://localhost:8080", "Realm": "[Realm]" }
```
```csharp
var authServer = builder.Configuration.GetSection("AuthServer").Get<AuthServerData>();
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, o =>
    {
        o.MetadataAddress = $"{authServer.Host}/realms/{authServer.Realm}/.well-known/openid-configuration";
        o.Authority = $"{authServer.Host}/realms/{authServer.Realm}";
        o.Audience = "account";
        o.RequireHttpsMetadata = false; // для локального http-Keycloak; в проде — true
    });
builder.Services.AddAuthorization(opt => opt.AddPolicy("admin", p => p.RequireRole("POWER-USER")));
// ...
app.UseAuthentication();
app.UseAuthorization();
```
Разрешить анонимное чтение, но требовать политику `"admin"` на изменение:
```csharp
var group = routes.MapGroup("/api/Dish").WithTags(nameof(Dish)).DisableAntiforgery().RequireAuthorization("admin");
group.MapGet("/{category:alpha?}", async (...) => { ... }).WithName("GetAllDishes").WithOpenApi().AllowAnonymous();
```
Проверка: без токена — код **401**; с заголовком `Authorization: Bearer <access-token>` (роль `POWER-USER`) — данные получены.

**4.7. Настройка `XXX.UI`.** Пакеты `Microsoft.AspNetCore.Authentication.OpenIdConnect`, `...JwtBearer`. В `appsettings.json` секция `Keycloak` (Host/Realm/ClientId/ClientSecret). Регистрация Cookie + OpenIdConnect:
```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = "keycloak";
})
.AddCookie()
.AddOpenIdConnect("keycloak", options =>
{
    options.Authority = $"{keycloakData.Host}/auth/realms/{keycloakData.Realm}";
    options.ClientId = keycloakData.ClientId;
    options.ClientSecret = keycloakData.ClientSecret;
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.Scope.Add("openid");
    options.SaveTokens = true;
    options.RequireHttpsMetadata = false;
});
builder.Services.AddAuthorization(opt => opt.AddPolicy("admin", p => p.RequireRole("POWER-USER")));
app.MapRazorPages().RequireAuthorization("admin");
```
**4.7.1. Получение токена аутентификации.** Нужны два токена: **токен приложения** (client_credentials — для регистрации пользователей через Admin API) и **токен пользователя** (для обращения к API от его имени). Сервис `ITokenAccessor`:
```csharp
public interface ITokenAccessor
{
    Task SetAuthorizationHeaderAsync(HttpClient httpClient, bool isClient);
}
```
Токен пользователя — из `HttpContext`: `await context.GetTokenAsync("keycloak", "access_token")` (после `AddHttpContextAccessor()`). Токен клиента — прямой `POST` на token endpoint с `client_credentials`. Регистрация: `builder.Services.AddHttpClient<ITokenAccessor, KeycloakTokenAccessor>();` Перед каждым запросом к API — вызов `await _tokenAccessor.SetAuthorizationHeaderAsync(_httpClient, false);` в сервисах (`ApiProductService` и т.д.).

**4.8. Регистрация на сервере аутентификации.** Своя страница регистрации (Keycloak Admin API не поддерживает загрузку аватара «из коробки»): `POST /admin/realms/{realm}/users`, заголовок `Authorization: Bearer {access-token клиента}`, тело — сериализованный `CreateUserModel` (`Attributes["avatar"]`, `Username`, `Email`, `Enabled`, `EmailVerified`, `Credentials`). Файл аватара сохраняется через `IFileService.SaveFileAsync`, если не передан — путь по умолчанию `/images/default-profile-picture.png`. Контроллер `AccountController` (методы `Register` GET/POST). Представление — Scaffold по шаблону Create, `enctype="multipart/form-data"`.

**4.9. Доработка `XXX.UI` — меню.** Если пользователь **не** вошёл — кнопки «Войти»/«Зарегистрироваться»; если вошёл — корзина, имя, аватар, рабочий Logout.
```csharp
public async Task Login() =>
    await HttpContext.ChallengeAsync("keycloak", new AuthenticationProperties { RedirectUri = Url.Action("Index", "Home") });

[HttpPost]
public async Task Logout()
{
    await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    await HttpContext.SignOutAsync("keycloak", new AuthenticationProperties { RedirectUri = Url.Action("Index", "Home") });
}
```
В `_UserInfoPartial`: `User.Identity.IsAuthenticated`; имя и аватар — из Claims: `User.Claims.FirstOrDefault(c => c.Type.Equals("preferred_username", StringComparison.OrdinalIgnoreCase))?.Value`.

**5. Контрольные вопросы:** 1. Какой механизм аутентификации имеет встроенную поддержку в ASP.NET Core? 2. Что описывают `ClaimsPrincipal`/`ClaimsIdentity`? 3. Как подключить Middleware аутентификации/авторизации? 4. Пример использования `HttpContext.User`. 5. Как проверить, что пользователь прошёл аутентификацию? 6. Как получить значение Claim? 7. Как получить Id аутентифицированного пользователя? 8. Как разрешить доступ только роли «manager»? 9. Как создать политику авторизации на основе Claim? 10. Как создать куки аутентификации через `HttpContext`? 11. Как подключить `Microsoft.AspNetCore.Identity`? 12. Как создать пользователя через Identity? 13. Как выполнить вход через Identity? 14. Как добавить Claim пользователю через Identity? 15. Какой интерфейс используется для доступа к хранилищу пользователей?

### Теория для выполнения ЛР6

#### Аутентификация и авторизация — базовые понятия

*Определение.* **Аутентификация** — проверка подлинности (например, сравнение пароля с БД). **Авторизация** — предоставление прав на действия и проверка этих прав при попытке выполнения. *Где пригодится: разграничение этих двух понятий — типовой вопрос на аттестации* **[Атт. №36, №39]**.

ASP.NET Core имеет **встроенную поддержку аутентификации на основе Cookie** *(частый вопрос на аттестации)* **[Атт. №36]**. Аутентификация обрабатывается службой **`IAuthenticationService`**, регистрируется как сервис, используется Middleware аутентификации: компонент сериализует данные пользователя в зашифрованные аутентификационные куки и передаёт клиенту; при получении запроса с такими куки — валидация, десериализация, инициализация свойства **`HttpContext.User`**.

*Определение.* **`User`** (тип `ClaimsPrincipal`) — представляет текущего пользователя, содержит набор **Claims** (утверждений). **Claim** — информация о пользователе (тип/значение), используемая в аутентификации/авторизации. Свойства `Claim`: `Type` (обычно URI — семантика claim), `Subject` (`ClaimsIdentity` пользователя), `Value`.

Методы `ClaimsPrincipal`: `Identity` (→ `ClaimsIdentity`), `FindAll(type|predicate)`, `FindFirst(type|predicate)`, `HasClaim(type,value)`, `IsInRole(name)`. Методы `ClaimsIdentity`: `Claims`, `AddClaim(claim)`, `AddClaims(claims)`, `RemoveClaim(claim)`. *Где пригодится: прямой вопрос аттестации «что описывают ClaimsPrincipal и ClaimsIdentity»* **[Атт. №2 общая часть]**.

Регистрация двух схем сразу (JWT для API + Cookie для сайта):
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options => builder.Configuration.Bind("JwtSettings", options))
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options => builder.Configuration.Bind("CookieSettings", options));
```
Чтение Claims в действии:
```csharp
[Authorize] public IActionResult Claims() => View(User?.Claims);
var id = User.FindFirst(ClaimTypes.NameIdentifier);
```

#### ASP.NET Core Identity

*Определение.* **ASP.NET Core Identity** — система членства: регистрация учётных записей, ролей, назначение ролей — для реализации аутентификации/авторизации своими силами (альтернатива внешнему Keycloak — в лабе используется Keycloak, но Identity часто спрашивают на аттестации и он же используется во многих реальных проектах). *Где пригодится: не является прямым заданием ЛР6 (там — Keycloak), но это стандартный ответ на 4 вопроса аттестации* **[Атт. №37, №38]**.

Классы (`Microsoft.AspNetCore.Identity`): `IdentityUser`, `IdentityRole`, `IdentityUserClaim`. Менеджеры: `UserManager<TUser>` (добавление/удаление/поиск/роли), `RoleManager<TRole>`, `SignInManager<TUser>` (вход/выход, формирует куки аутентификации).
```csharp
var result = await _userManager.CreateAsync(newUser, password);
var user = await _userManager.FindByEmailAsync("xxx");
var newClaim = new Claim("company", "Microsoft");
await _userManager.AddClaimAsync(newUser, newClaim);
await _signInManager.SignInAsync(newUser, isPersistent: false);
var result2 = await _signInManager.PasswordSignInAsync("name", "password", isPersistent: false, lockoutOnFailure: false);
```
Хранилище — интерфейсы `IUserStore<TUser>`, `IRoleStore<TRole>`; готовая EF Core реализация — `Microsoft.AspNetCore.Identity.EntityFrameworkCore` (классы `UserStore`, `RoleStore`, `IdentityDbContext`). *Где пригодится: вопрос «какой интерфейс используется для доступа к хранилищу пользователей» — ответ: `IUserStore<TUser>`* **[Атт. п.15 списка вопросов лабы]**.
```csharp
builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
app.UseAuthentication();
```
`AddDefaultIdentity` ≈ `AddIdentity` + `AddDefaultUI` + `AddDefaultTokenProviders`. Настройка требований к паролю: `opt.Password.RequiredLength = 4; opt.Password.RequireNonAlphanumeric = false; ...`. Собственный `DbContext` наследуется от `IdentityDbContext` (или `IdentityDbContext<ApplicationUser>` при кастомном пользователе, `ApplicationUser : IdentityUser`).

#### Авторизация: роли и политики

```csharp
[Authorize] public IActionResult ShowCompanies(...) { ... }
[Authorize(Roles = "admin, poweruser")] public IActionResult ShowCompanies(...) { ... }

builder.Services.AddAuthorization(options =>
    options.AddPolicy("DepartmentCheck", policy => policy.RequireClaim("Department", "marketing", "advertizing")));
[Authorize(Policy = "DepartmentCheck")] public IActionResult ShowCompanies(...) { ... }
```
*Где пригодится: политика `"admin"` через `RequireRole` — ровно то, что используется в ЛР6/ЛР8/ЛР12 для защиты API/страниц.* **[Атт. №39]**

Внешние провайдеры (Google и др.):
```csharp
builder.Services.AddAuthentication().AddGoogle(options =>
{
    options.ClientId = Configuration["Authentication:Google:ClientId"];
    options.ClientSecret = Configuration["Authentication:Google:ClientSecret"];
});
```

#### JWT вручную (для понимания механизма, который делает Keycloak «под капотом»)

```csharp
var claims = new[] {
    new Claim(JwtRegisteredClaimNames.Sub, _configuration["Jwt:Subject"]),
    new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
    new Claim("UserId", user.Id.ToString()),
};
var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
var signIn = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
var token = new JwtSecurityToken(_configuration["Jwt:Issuer"], _configuration["Jwt:Audience"], claims,
    expires: DateTime.UtcNow.AddMinutes(10), signingCredentials: signIn);
return Ok(new JwtSecurityTokenHandler().WriteToken(token));
```

**Cookie vs JWT — сравнение** (таблица для запоминания):

| | Cookie-аутентификация | JWT (Bearer) |
|---|---|---|
| Где хранится состояние | сервер (сессия/шифрованная кука) | нет состояния на сервере — вся информация в токене |
| Типичный сценарий | классический сайт (браузер + сервер, тот же домен) | API, SPA, мобильные клиенты, межсерверное взаимодействие |
| Передача | автоматически браузером с каждым запросом | вручную, заголовок `Authorization: Bearer <token>` |
| Отзыв токена | легко (удалить сессию/куку) | сложно до истечения `exp` (нужен доп. механизм blacklist/refresh) |
| В курсе используется для | `XXX.UI` (Cookie + OpenIdConnect к Keycloak) | `XXX.API`, `XXX.BlazorWasm` (JwtBearer, `IAccessTokenProvider`) |

---

## Лабораторная работа №7: Вспомогательные классы тэгов (Tag Helpers), Ajax

### Текст задания

**Цель.** Знакомство с тэг-хелперами. Знакомство с механизмом асинхронных запросов HTTP. **Задача.** Научиться создавать и регистрировать tag-helpers, посылать и обрабатывать асинхронные запросы. Время — 4 часа.

**3.1. Задание 1.** Создать tag-helper, выводящий кнопки пейджера:
```html
<Pager current-page="@Model.CurrentPage" total-pages="@Model.TotalPages" category="@category"></Pager>
<!-- или -->
<Pager current-page="@Model.PageNo" total-pages="@Model.TotalPages" admin="true"></Pager>
```
**Обязательно:** для формирования тегов использовать методы класса **`TagBuilder`**. Рекомендации: для адресов страниц внедрить в конструктор tag-helper **`LinkGenerator`**; атрибут `admin` нужен, т.к. на странице `Admin/Index` ссылки должны вести на страницы (`GetPathByPage`), а не на методы контроллеров; `HttpContext` для `GetPathByPage` получить через `IHttpContextAccessor`. Зарегистрировать tag-helper в `_ViewImports.cshtml`.

**3.2. Задание 2.** Переключение страниц через **Ajax** — должен подгружаться только список объектов и пейджер (без перезагрузки всей страницы). Рекомендации: выделенный фрагмент — в частичное представление; в контроллере `Product` проверить, был ли запрос асинхронным (заголовок `x-requested-with: xmlhttprequest`) → вернуть `PartialView`, иначе — полное `View`. В `wwwroot/scripts/site.js` — подписка на `click` всех `<a>` пейджера (класс `page-link`), использовать `$("xxx").load()` (jQuery).

**3.3. Задание 3.** Описать расширяющий метод для `HttpRequest`, проверяющий асинхронность запроса, использовать в контроллере `Product`:
```csharp
public async Task<IActionResult> Index([FromServices] IConfiguration config, int pageNo = 1, int category = 0)
{
    var page = ProductPageViewModel.GetModel(data, pageNo, pageSize);
    if (Request.IsAjaxRequest()) return PartialView("_ListPartial", page);
    return View(page);
}
```
Класс `HttpRequestExtensions` — в папке `Extensions`.

**4. Контрольные вопросы:** 1. Что такое tag-helper? 2. Какие tag-helpers тэга `<form>` вы знаете? 3. Какие tag-helpers тэга `<a>` вы знаете? 4. Как зарегистрировать использование tag-helper в проекте? 5. Как определить, что запрос выполнен по Ajax? 6. Может ли MVC-контроллер вернуть частичное представление? 7. Может ли контроллер вернуть ответ без тела сообщения? Пример. 8. Пример, когда контроллер возвращает ответ, не являющийся представлением. 9. Можно ли tag-helper'ом поменять тэг элемента? 10. Как в tag-helper получить URL конечной точки?

### Теория для выполнения ЛР7

#### Создание собственных Tag Helpers

Класс tag-helper наследуется от `Microsoft.AspNetCore.Razor.TagHelpers.TagHelper` и переопределяет:
```csharp
public override void Process(TagHelperContext context, TagHelperOutput output)
// или async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
```
Пример — тег `<email mail-to="gogogo">` → `<a href="mailto:gogogo@mail.ru">gogogo@mail.ru</a>`:
```csharp
public class EmailTagHelper : TagHelper
{
    private string domain = "@mail.ru";
    public string MailTo { get; set; } // связывается с атрибутом mail-to (kebab-case ← PascalCase)
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "a";
        var address = MailTo + domain;
        output.Attributes.Add("href", "mailto:" + address);
        output.Content.SetContent(address);
    }
}
```
Изменение тега вокруг содержимого (`[HtmlTargetElement(Attributes = "bold")]`, `output.Attributes.RemoveAll(...)`, `output.PreElement.SetHtmlContent("<strong>")`, `output.PostElement.SetHtmlContent("</strong>")`) — да, **tag-helper может поменять тег элемента** (ответ на контрольный вопрос №9).

**`TagBuilder`** — программное построение тегов без строковой конкатенации HTML (обязательное требование задания 3.1):
```csharp
public class UserTagHelper : TagHelper
{
    public User props { get; set; }
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "table";
        var div = new TagBuilder("div");
        foreach (var p in typeof(User).GetProperties())
        {
            var tr = new TagBuilder("tr");
            tr.InnerHtml.AppendHtml($"<td>{p.Name}</td><td>{p.GetValue(props)}</td>");
            div.InnerHtml.AppendHtml(tr);
        }
        output.Attributes.Add("class", "table table-condensed");
        output.TagMode = TagMode.StartTagAndEndTag;
        output.Content.SetHtmlContent(div);
    }
}
```

**Генерация адресов внутри tag-helper** (для кнопок пейджера — ключевая часть задания 3.1). Несколько вариантов, самый современный (с версии 2.2) — класс **`LinkGenerator`**, зарегистрированный как сервис по умолчанию, внедряется через конструктор:
```csharp
[HtmlTargetElement(tag: "img", Attributes = "img-action,img-controller")]
public class ImageTagHelper : TagHelper
{
    LinkGenerator _linkGenerator;
    public ImageTagHelper(LinkGenerator linkGenerator) { _linkGenerator = linkGenerator; }
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        var uri = _linkGenerator.GetPathByAction(ImgAction, ImgController);
        output.Attributes.Add("src", uri);
    }
}
```
Для ссылок на Razor Page — `_linkGenerator.GetPathByPage(...)`, для которого нужен `HttpContext` — его получают из внедрённого `IHttpContextAccessor.HttpContext` (сервис нужно зарегистрировать: `builder.Services.AddHttpContextAccessor();`).

Подключение — в `_ViewImports.cshtml`: `@addTagHelper TagHelperSamples.Web.TagHelpers.DemoTagHelper, TagHelperSamples.Web` (или `@addTagHelper "*, Сборка"` для всех хелперов сборки).

#### Ajax, JavaScript, jQuery, JSON

*Определение.* **Ajax** (изначально Asynchronous JavaScript and XML) — использование JS для асинхронного HTTP-запроса к серверу и динамического обновления части страницы без полной перезагрузки. Современные приложения вместо XML почти всегда используют **JSON**. *Где пригодится: прямое задание ЛР7 (частичное обновление списка + пейджера); используется также в ЛР11/ЛР12 (Blazor обращается к API асинхронно, хотя и без ручного jQuery).* **[Атт. №32, №33]**

**Определение асинхронного запроса на сервере** — по заголовку:
```csharp
if (Request.Headers["x-requested-with"].ToString().ToLower().Equals("xmlhttprequest"))
    return PartialView();
return View();
```

**jQuery** — библиотека для работы с **DOM** (Document Object Model — представление HTML-документа в виде дерева объектов, с которым можно работать из JS), манипуляций стилями/контентом, обработки событий, Ajax-запросов, анимации. Поиск элементов: `$('div')`, `$('#MyID')`, `$('.MyClass')`. Изменение: `.attr()`, `.css()`, `.addClass()`/`.removeClass()`/`.toggleClass()`. Контент: `.append()/.prepend()/.before()/.after()`, `.html()/.text()`. События: `.on('click', fn)` (предпочтительнее устаревшего `.bind`).

**Ajax-запросы**:
```javascript
$.ajax({ url: 'xxx', dataType: 'html', data: { id: 1 } }).success(function (data) { ... });
$("#container").load('@Url.Action("PartialMethod","Controller")/' + id); // самый простой способ подгрузить HTML
```
Асинхронная отправка формы:
```javascript
$('#MyForm').submit(function (event) {
    event.preventDefault();
    var data = $(this).serialize();
    var url = $(this).attr('action');
    $.post(url, data, function (response) { $('#comments').append(response); });
});
```
**Работа с JSON**: контроллер — `return Json(await _context.Books.ToListAsync());`, клиент — `$.getJSON(url, function(data) { ... });` или `$.ajax` c `success` (первый параметр функции — уже десериализованный объект).

---

## Лабораторная работа №8: Маршрутизация. Передача состояния (сессия, кэш, корзина)

### Текст задания

**Цель.** Знакомство с системой маршрутизации ASP.NET Core и механизмом сессии. Время — 6 часов. Используется проект из ЛР7.

**3.2. Задание 1. Маршрутизация.** Зарегистрировать маршрут, чтобы обращение к `Index` контроллера `Product` выглядело как `localhost:xxxx/Catalog` или `localhost:xxxx/Catalog/имя_категории` (вместо стандартного `?category=...`). Рекомендация: использовать **маршрутизацию через атрибуты**.

**3.3. Задание 2. Кэширование ответов API.** В `XXX.API` кэшировать ответ при запросе списка объектов: **Hybrid Cache** (.NET 8+) или **Distributed Memory Cache** (более старые версии); внешнее распределённое хранилище — **Redis**. В `compose.yaml` добавить сервис `redis` (`image: redis:7.2-alpine`, порт `6379:6379`, volume). В `appsettings.json`: `"Redis": "localhost:6379"`. Пакеты `Microsoft.Extensions.Caching.Hybrid`, `Microsoft.Extensions.Caching.StackExchangeRedis`. Регистрация:
```csharp
builder.Services.AddHybridCache();
builder.Services.AddStackExchangeRedisCache(opt =>
{
    opt.InstanceName = "labs_";
    opt.Configuration = builder.Configuration.GetConnectionString("Redis");
});
```
Кэширование конечной точки списка:
```csharp
group.MapGet("/{category:alpha?}", async (IMediator mediator, HybridCache cache, string? category, int pageNo = 1) =>
{
    var data = cache.GetOrCreateAsync($"dishes_{category}_{pageNo}",
        async token => await mediator.Send(new GetListOfProducts(category, pageNo)),
        options: new HybridCacheEntryOptions { Expiration = TimeSpan.FromMinutes(1), LocalCacheExpiration = TimeSpan.FromSeconds(30) });
    return TypedResults.Ok(data);
}).WithName("GetAllDishes").WithOpenApi().AllowAnonymous();
```
Проверить время отклика при разных интервалах между запросами: < 30 сек — из локального кэша (быстро), 30 сек – 1 мин — из распределённого (Redis), > 1 мин — из БД.

**3.4. Задание 3. Корзина заказов.** По клику «В корзину» — объект добавляется в корзину, в меню пользователя правильно отображается информация о корзине. Добавление — **только для вошедших** пользователей. По клику на корзину — список объектов заказа с удалением. Для знакомства с сохранением состояний — корзину хранить **в сессии** (в реальности обычно — в БД). Регистрация сессии:
```csharp
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession();
// ...
app.UseSession();
```
Контроллер `Cart`; в `XXX.Domain` — `CartItem` (объект + количество) и `Cart` (словарь `CartItems`, методы `AddToCart`, `RemoveItems`, `ClearAll`, свойства `Count`, `TotalCalories`). Хранение в сессии через расширяющие методы (сериализация JSON):
```csharp
[Authorize]
[Route("[controller]/add/{id:int}")]
public async Task<ActionResult> Add(int id, string returnUrl)
{
    Cart cart = HttpContext.Session.Get<Cart>("cart") ?? new();
    var data = await _productService.GetProductByIdAsync(id);
    if (data.Success) { cart.AddToCart(data.Data); HttpContext.Session.Set<Cart>("cart", cart); }
    return Redirect(returnUrl);
}
```
`CartViewComponent.Invoke` — получить корзину из сессии, передать в представление компонента.

**3.5. Задание 4. Сервис корзины заказов.** Объект `Cart` должен **внедряться** в конструкторы `CartController`/`CartViewComponent` через DI — они не должны напрямую работать с сессией (слабая связанность, проще тестировать/рефакторить):
```csharp
public class CartController : Controller
{
    private readonly Cart _cart;
    public CartController(IProductService productService, Cart cart) { _cart = cart; }
    public async Task<ActionResult> Add(int id, string returnUrl)
    {
        var data = await _productService.GetProductByIdAsync(id);
        if (data.Success) _cart.AddToCart(data.Data);
        return Redirect(returnUrl);
    }
}
```
Рекомендация: сервис `SessionCart : Cart`, переопределяющий виртуальные методы так, чтобы при изменении данные сохранялись в сессию; зарегистрировать `Cart` как **Scoped**-сервис. (Пример реализован в Adam Freeman, *Pro ASP.NET Core 6*, гл. 9 «SportsStore: Completing the Cart».)

**4. Контрольные вопросы:** 1. Какие ещё механизмы сохранения состояния есть в ASP.NET Core, кроме сессии? 2. Где хранятся данные сессии? 3. Сколько времени хранятся данные сессии? 4. Как прочитать/записать данные Cookie? 5. Сколько времени хранятся данные Cookie? 6. Какие преимущества даёт распределённый (Distributed) кэш? 7. Что такое «статический сегмент маршрута»? Пример. 8. Как в контроллере получить значение сегмента маршрута? 9. Как в `<a>` с tag-helper указать значение сегмента маршрута? 10. Как зарегистрировать маршрут в `Program`? 11. Как зарегистрировать маршрут через атрибуты? 12. Что такое ограничения (constraints) маршрута? 13. Как получить адрес конечной точки в коде контроллера/представления?

### Теория для выполнения ЛР8

#### Маршрутизация ASP.NET Core

*Определение.* **Маршрутизация (Routing)** отвечает за сопоставление входящих HTTP-запросов с исполняемыми **конечными точками (Endpoints)** приложения; выполняет две функции: 1) сопоставляет запрос с endpoint'ом, 2) генерирует исходящие URL, соответствующие endpoint'ам. Регистрируется как **Middleware** в конвейере (`app.UseRouting()`). *Где пригодится: прямая тема ЛР8 (задание 1), используется во всех лабах с контроллерами/страницами.* **[Атт. №26, №27, №28]**

```csharp
app.MapGet("/", async context => await context.Response.WriteAsync("Hello World!"));
```
Специальные методы для подключения частей платформы к маршрутизации: `MapRazorPages` (Razor Pages), `MapControllers`/`MapControllerRoute` (MVC), `MapHub<THub>` (SignalR).

**Регистрация маршрутов MVC** (conventional routing):
```csharp
app.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}");
```
Компоненты маршрута: **имя**, **шаблон (pattern)**, **значения по умолчанию (defaults)**, **ограничения (constraints)**.
```csharp
app.MapControllerRoute(name: "default", pattern: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });
app.MapControllerRoute("", "Public/{controller}/{action}", new { controller = "Home", action = "Index" }); // статический сегмент "Public"
app.MapControllerRoute(name: "events", pattern: "{year}/{month}/{day}", defaults: new { controller = "Events", action = "Show", day = 1 });
app.MapControllerRoute(name: "MyRoute", pattern: "{controller=Home}/{action=Index}/{*therest}"); // catch-all сегмент
```
**Ограничения (constraints):** необязательный параметр `{id?}`, тип `{id:int}`, регулярное выражение `{code:regex(выражение)}`, комбинация `{id:alpha:minlength(6)?}`. **Порядок регистрации маршрутов важен** — более специфичные маршруты регистрируются раньше общих.

Пример из лабы — красивый URL для страницы каталога:
```csharp
app.MapControllerRoute(name: "pager1", pattern: "Catalog/Show/Page_{pageNo:int}", defaults: new { controller = "Home", action = "Index" });
app.MapControllerRoute(name: "pager", pattern: "Catalog/Show", defaults: new { controller = "Home", action = "Index" });
```

**Маршрутизация через атрибуты** — основной атрибут `[Route]`; если у метода есть `[Route]`, он **больше не доступен** через conventional-маршруты из `Program.cs`.
```csharp
[Route("Catalog")]
public class HomeController : Controller
{
    [Route("Show")]
    [Route("Show/Page_{pageNo:int}")]
    public IActionResult Index(int pageNo = 1) => View();
}
```
Можно комбинировать `[Route]` на классе (общий префикс) и на методах (`[Route("{reviewId}")]`). Изменение имени действия: `[ActionName("MyAction")]`.

**Генерация URL**: `<a asp-controller="Home" asp-action="index" asp-route-pageNo="2">` → `/Catalog/Show/Page_2`; в коде — `Url.Action("Index","Home", new { id=100 })`; вне контроллера/представления — сервис **`LinkGenerator`** (singleton, внедряется в конструктор): `_linkGenerator.GetPathByAction("index", "home", new { pageNo = 2 });`

#### Сохранение состояния между запросами

Поскольку HTTP не хранит состояние (см. ЛР1), ASP.NET Core предлагает несколько механизмов — **сводная таблица (важно для аттестации и практики)** **[Атт. №43]**:

| Механизм | Где хранится | Время жизни | Типичное применение |
|---|---|---|---|
| **Query string** | URL (видно клиенту) | пока действует ссылка | «шарить» ссылку с состоянием (email, соцсети); никогда не хранить чувствительные данные |
| **Скрытые поля формы** | HTML-форма | один переход (submit) | многостраничные формы; данные обязательно перепроверять на сервере (клиент может подделать) |
| **TempData** | Cookie или сессия (провайдер TempData) | переживает один Redirect | сообщения после перенаправления |
| **Cookie** | браузер клиента | задаётся вручную (`CookieOptions.Expires`), лимит ~4096 байт | «долгая» память о клиенте (например язык, тема) |
| **Session** | сервер (кэш, `IDistributedCache`), клиенту — только id сессии в cookie | настраиваемый таймаут (обычно 20 мин простоя) | временные данные текущего визита — **корзина заказа в ЛР8** |
| **Кэширование** (in-memory/distributed/response) | сервер / Redis | настраиваемый **TTL** (Time To Live — «время жизни»: сколько кэш хранит значение, прежде чем считать его устаревшим) | данные, не привязанные к конкретному пользователю — **ответы API в ЛР8** |

**TempData** — `TempData.Peek("Message")` / `TempData.Keep("Message")` для чтения без удаления в конце запроса.

**Cookie** — чтение `Request.Cookies`, запись `Response.Cookies`:
```csharp
if (!Request.Cookies.ContainsKey("Name")) Response.Cookies.Append("Name", "Bob");
Response.Cookies.Delete("Name");
Response.Cookies.Append("Name", "Bob", new CookieOptions { Expires = DateTime.Now.AddDays(2) });
```

**Сессия (Session)** — сценарий для временных данных, доступных пока пользователь работает с приложением. За сессию отвечает Middleware; клиенту выдаётся cookie с id сессии. Требуется: реализация `IDistributedCache` (резервное хранилище) + `AddSession()` + `UseSession()`:
```csharp
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession();
app.UseSession();
```
Доступ — `HttpContext.Session` (реализация `ISession`), расширяющие методы для `int`/`string`:
```csharp
HttpContext.Session.SetString("_name", "Bob");
HttpContext.Session.SetInt32("_age", 18);
```
Сложные типы сериализуются вручную (обычно JSON) — расширяющие `Set<T>`/`Get<T>`:
```csharp
public static void Set<T>(this ISession session, string key, T value) => session.SetString(key, JsonSerializer.Serialize(value));
public static T Get<T>(this ISession session, string key)
{
    var value = session.GetString(key);
    return value == null ? default : JsonSerializer.Deserialize<T>(value);
}
```

**Кэширование** — эффективный способ хранения/извлечения данных, время жизни контролируется приложением; кэш **не привязан** к конкретному запросу/пользователю/сессии — нельзя кэшировать персональные данные, которые могут «утечь» другому пользователю. Варианты: in-memory/distributed memory, кэширование ответа, кэширование запроса; tag-helper `<cache>@DateTime.Now</cache>` (по умолчанию 20 минут). *Где пригодится: HybridCache/Redis в ЛР8 — распределённый кэш для всех экземпляров приложения (важно при горизонтальном масштабировании — в отличие от in-memory, который у каждого инстанса свой).*

---

## Лабораторная работа №9: Модульное тестирование

### Текст задания

**Цель.** Знакомство с xUnit. Использование xUnit для тестирования классов контроллера.

**Класс `Assert`** (методы-утверждения): `Contains`/`DoesNotContain`, `DoesNotThrow`, `Equal`/`NotEqual`, `True`/`False`, `InRange`/`NotInRange`, `IsAssignableFrom`, `IsType`/`IsNotType`, `NotEmpty`, `NotNull`/`Null`, `Same`/`NotSame`, `Throws`.
```csharp
[Fact]
public void StackLimitIsControlled()
{
    var stack = new StackDemo(); stack.StackLimit = 0;
    Action testCode = () => stack.Push("test");
    Assert.Throws<StackDemo.StackFullException>(testCode);
}
```
**`[Fact]`** — тест, критерий которого всегда истинен независимо от данных. **`[Theory]`** — тест, зависящий от набора данных (не пройдёт для одних данных, пройдёт для других), источники данных: `[InlineData(...)]` (простые значения), `[ClassData(typeof(...))]` (класс — источник, наследник `IEnumerable<object[]>`), `[MemberData(nameof(...))]` (статический метод, возвращающий `IEnumerable<object[]>`).

**3.1. Исходные данные.** Используется проект из ЛР8. **3.2. Подготовка проекта.** Добавить проект xUnit `XXX.Tests`, пакеты: `NSubstitute` (имитация классов/интерфейсов), `Microsoft.AspNetCore.Mvc` (тестирование контроллеров), `Microsoft.EntityFrameworkCore.InMemory` или `...Sqlite` (имитация контекста БД). Ссылки — на основной проект, `XXX.Api`, `XXX.Domain`.

**3.3. Задание 1. Тестирование контроллера `Product`.** Тесты для метода `Index` (код метода см. в тексте лабы — получает категории, проверяет успешность, кладёт во `ViewData`, получает список объектов, проверяет Ajax-заголовок → `PartialView`/`View`). Проверить: возврат **404**, если список категорий не получен; возврат **404**, если список объектов не получен; при успехе — во `ViewData` передан список категорий и правильное текущее имя категории (`"Все"`, если категория не указана); в представление передана модель — список объектов. (Для Ajax-варианта тест можно не делать.) Сервисы `ICategoryService`/`IProductService` имитировать через **NSubstitute** (мокать только используемые методы). Свойство `Request` — см. пример в лекции (мок `HttpContext`).

**3.4. Задание 2. Тестирование `ProductService` (`XXX.Api`).** Тесты для `GetProductListAsync`: 1) по умолчанию возвращает 1-ю страницу из 3 объектов и правильно считает `TotalPages`; 2) правильно выбирает заданную страницу; 3) правильно фильтрует по категории; 4) не позволяет задать размер страницы больше максимального (см. ЛР4, п.3.7); 5) возвращает `Success=false`, если номер страницы больше общего количества. Т.к. `IWebHostEnvironment`/`IHttpContextAccessor` не используются в этом методе — можно передать `null` в конструктор. Имитация БД: **In-memory провайдер** (не рекомендуется официально) или **SQLite in-memory** (предпочтительно). Пример теста:
```csharp
[Fact]
public void ServiceReturnsFirstPageOfThreeItems()
{
    using var context = CreateContext();
    var service = new ProductServise(context, null, null);
    var result = service.GetProductListAsync(null).Result;
    Assert.IsType<ResponseData<ListModel<Dish>>>(result);
    Assert.True(result.Successfull);
    Assert.Equal(1, result.Data.CurrentPage);
    Assert.Equal(3, result.Data.Items.Count);
    Assert.Equal(2, result.Data.TotalPages);
}
```

**4. Контрольные вопросы:** 1. Для чего используют модульные тесты? 2. Что означает шаблон AAA в модульном тестировании? 3. Для чего используется библиотека NSubstitute? 4. Как в xUnit проверить, что значение `int`, возвращённое методом, верно? 5. Как протестировать значение, передаваемое во `ViewData`? 6. Как в xUnit проверить, что метод генерирует исключение? 7. Как в NSubstitute создать имитацию метода класса?

### Теория для выполнения ЛР9

#### Принципы модульного тестирования

*Определение.* **Модульное тестирование (Unit Testing)** — проверка приложения на самом низком уровне: каждый тест проверяет один низкоуровневый компонент (класс/метод), часто — специфичное условие для метода. «Правильный» тест должен быть **FAIR**: *Fast* (быстрый), *Atomic* (атомарный — проверяет одну небольшую часть), *Independent* (независим от других систем/тестов, не влияет на них), *Repeatable* (повторяемый — одинаковый результат всегда). *Где пригодится: прямая тема ЛР9, стандартный вопрос на аттестации* **[Атт. №34]**.

**Шаблон AAA** (структура теста): **Arrange** (подготовка исходных данных) → **Act** (вызов тестируемого метода) → **Assert** (проверка соответствия результата ожиданиям). *Где пригодится: прямой вопрос на аттестации «что означает шаблон AAA».* **[Атт. №34]**

#### xUnit

```csharp
public class UnitTest1
{
    [Fact]
    public void Test1()
    {
        // Arrange
        var value = "test value"; var stack = new Stack<string>(); stack.Push(value);
        // Act
        string result = stack.Pop();
        // Assert
        Assert.Equal(value, result);
    }
}
```
`[Theory]` + `[InlineData(2,4,.5)]` — достоинство: простота; недостаток: нельзя передавать объекты классов. `[ClassData]`/`[MemberData]` — можно передавать сложные объекты:
```csharp
public class DataSource : IEnumerable<object[]>
{
    private List<object[]> datas = new() { new object[]{2,4,.5}, new object[]{2,4,1} };
    public IEnumerator<object[]> GetEnumerator() => datas.GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
    public static IEnumerable<object[]> GetTestData() { yield return new object[] { 2, 4, .5 }; }
}
[Theory] [ClassData(typeof(DataSource))]
public void CanDivide(int a, int b, double c) { Assert.Equal(c, new Calculator().Div(a, b)); }
```

**Тестирование контроллера** — проверка типа результата и модели:
```csharp
[Fact]
public void IndexReturnsView()
{
    var controller = new HomeController();
    var result = controller.Index().Result;
    Assert.NotNull(result);
    var viewResult = Assert.IsType<ViewResult>(result);
    Assert.IsType<Book>(viewResult.Model);
    // string someData = viewResult.ViewData["SomeData"].ToString(); — так проверяется ViewData
}
```

#### Moq / NSubstitute — имитация зависимостей (mock-объекты)

*Определение.* Чтобы проверять класс **независимо** от других (например, от БД), используют **макеты/фиктивные объекты (mock/fake objects)**, симулирующие поведение реальных зависимостей. Библиотеки: **Moq** (в лекции) и **NSubstitute** (в тексте самой лабы). *Где пригодится: прямое требование п.3.3–3.4 ЛР9 — иначе не изолировать `ProductController` от реальных сервисов.* **[Атт. №35]**

Moq:
```csharp
var moq = new Mock<IRepository>();
moq.Setup(m => m.GetPersons()).Returns(new List<Person> { new Person { Name = "John", Age = 25 } });
moq.Setup(m => m.GetAge(It.IsAny<int>())).Returns<int>(m => 30);
```
Методы `It`: `Is<T>(predicate)`, `IsAny<T>()`, `IsInRange<T>(min,max,kind)`, `IsRegex(expr)`.

**Мок `HttpContext`/`Request`** (для проверки Ajax-заголовка в контроллере, как требует ЛР9 п.3.3):
```csharp
var header = new Dictionary<string, StringValues>() { ["x-requested-with"] = "XMLHttpRequest" };
var moqHttpContext = new Mock<HttpContext>();
moqHttpContext.Setup(c => c.Request.Headers).Returns(new HeaderDictionary(header));
var controllerContext = new ControllerContext { HttpContext = moqHttpContext.Object };
var controller = new ProductController() { ControllerContext = controllerContext };
```

**In-memory `DbContext` для тестов**:
```csharp
_options = new DbContextOptionsBuilder<ApplicationDbContext>().UseInMemoryDatabase("testDb").Options;
using (var context = new ApplicationDbContext(_options)) { /* тест */ }
using (var context = new ApplicationDbContext(_options)) { context.Database.EnsureDeleted(); } // очистка
```
Официально Microsoft рекомендует **SQLite in-memory** вместо EF InMemory-провайдера — он точнее эмулирует поведение реальной реляционной БД (constraints, транзакции).

---

## Лабораторная работа №10: Логирование, Middleware

### Текст задания

**Цель.** Знакомство с библиотекой **Serilog**. Изучение работы компонентов Middleware.

**3.1. Исходные данные.** Проект из ЛР9. **3.2. Подготовка проекта.** Пакет NuGet `Serilog.AspNetCore`, настроить конфигурацию Serilog в `Program.cs` (см. `serilog-settings-configuration`). **Обязательно: настройки Serilog должны храниться в `appsettings.json`.**

**3.3. Задание.** Выводить в консоль и сохранять в файл информацию обо **всех** запросах, ответ на которые имеет код состояния, отличный от **2XX**. Информация должна содержать URL и код состояния:
```
2023-07-03 11:19:49.996 +03:00 [INF] ---> request /menu returns 404
2023-07-03 11:23:00.364 +03:00 [INF] ---> request /Cart/add/1 returns 302
```
Рекомендация: чтобы не логировать это во всех контроллерах/страницах — описать компонент **Middleware**, который проверяет ответ и при необходимости логирует.

**4. Вопросы для самопроверки:** 1. Для чего используются Middleware? 2. Для чего используется объект `RequestDelegate`, передаваемый в конструктор класса Middleware? 3. Как работает конвейер обработки HTTP-запросов? 4. Может ли Middleware обработать ответ, передаваемый клиенту? 5. Как добавить Middleware в конвейер обработки запросов?

### Теория для выполнения ЛР10

#### Middleware — создание собственного компонента

*Определение.* **Middleware** — ПО, собранное в конвейер приложения для обработки запросов и ответов; см. также базовое определение в разделе ЛР1. Каждый компонент решает, передавать ли запрос дальше по конвейеру, и может выполнять код **до и после** следующего компонента — а значит **может обработать и ответ**, возвращаемый клиенту (ответ на контрольный вопрос №4), просто разместив код после `await _next.Invoke(context)`. *Где пригодится: прямая тема ЛР10; понимание конвейера критично для отладки любого ASP.NET Core приложения — топ-вопрос аттестации.* **[Атт. №42]**

```csharp
public class MyMiddleware
{
    RequestDelegate _next;
    public MyMiddleware(RequestDelegate next) { _next = next; } // next — следующий делегат конвейера
    public async Task Invoke(HttpContext context)
    {
        // Предварительные действия (до следующего Middleware, т.е. до обработки запроса)
        await _next.Invoke(context);
        // Завершающие действия (после — можно проверить context.Response.StatusCode)
    }
}
```
Регистрация: `app.UseMiddleware<MyMiddleware>();` или через расширяющий метод (более чистый API):
```csharp
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseMyMiddleware(this IApplicationBuilder builder)
        => builder.UseMiddleware<MyMiddleware>();
}
// использование: app.UseMyMiddleware();
```
**Готовое решение задания ЛР10** — логирующий Middleware:
```csharp
public class ResponseLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ResponseLoggingMiddleware> _logger;
    public ResponseLoggingMiddleware(RequestDelegate next, ILogger<ResponseLoggingMiddleware> logger)
    { _next = next; _logger = logger; }

    public async Task InvokeAsync(HttpContext context)
    {
        await _next(context); // сначала пропускаем запрос дальше по конвейеру
        var statusCode = context.Response.StatusCode;
        if (statusCode < 200 || statusCode >= 300)
            _logger.LogInformation("---> request {Path} returns {StatusCode}", context.Request.Path, statusCode);
    }
}
```
Порядок в конвейере важен: Middleware логирования нужно ставить **после** `UseRouting()`/`MapControllers()` (или использовать `app.Use(...)` — обёртку вокруг всего конвейера), чтобы `context.Response.StatusCode` был уже выставлен обработчиком запроса.

#### Serilog — структурированное логирование

*(В лекциях подробностей мало — только ссылка на github.com/serilog; ниже — необходимый для лабы минимум по актуальному Serilog.AspNetCore.)*

Установка: `dotnet add package Serilog.AspNetCore`. Конфигурация **через `appsettings.json`** (как требует задание):
```json
"Serilog": {
  "MinimumLevel": { "Default": "Information", "Override": { "Microsoft.AspNetCore": "Warning" } },
  "WriteTo": [
    { "Name": "Console" },
    { "Name": "File", "Args": { "path": "Logs/log-.txt", "rollingInterval": "Day" } }
  ]
}
```
В `Program.cs`:
```csharp
builder.Host.UseSerilog((context, configuration) => configuration.ReadFrom.Configuration(context.Configuration));
```
Пакеты для файлового/консольного синка: `Serilog.Sinks.Console`, `Serilog.Sinks.File`, `Serilog.Settings.Configuration`. Инъекция логгера в любой класс — через `ILogger<T>` (встроенная абстракция Microsoft.Extensions.Logging, которую Serilog подменяет «под капотом»):
```csharp
public class MyService(ILogger<MyService> logger)
{
    public void DoWork() => logger.LogInformation("Работа выполнена");
}
```

**Middleware vs фильтры (Filters) — для общего понимания** *(дополнение от себя, важно для аттестации)* **[Атт. №24]**: Middleware работает на уровне всего HTTP-конвейера (видит любой запрос, даже до маршрутизации), а **фильтры контроллеров** (`IActionFilter`, `IExceptionFilter`, `IAuthorizationFilter` и др., атрибуты `[ServiceFilter]`/`[TypeFilter]`) работают только внутри MVC-конвейера, на уровне конкретного действия/контроллера — их проще привязать к бизнес-логике конкретного метода, но они не увидят запросы к статическим файлам или к другим Middleware.

---

## Лабораторная работа №11: Введение в Blazor (Server-Side Rendering)

### Текст задания

**Цель.** Изучение проекта Blazor. Знакомство с компонентами Razor.

**2.1. Исходные данные.** Проект из ЛР10. **2.2. Подготовка проекта.** Добавить в решение новый проект — **Blazor Web App**, имя `XXX.Blazor.SSR` (SSR — server side rendering), режим рендеринга — **на сервере**.

**2.3. Задание 1.** В `Program.cs` найти регистрацию сервиса Blazor и конечных точек:
```csharp
builder.Services.AddRazorComponents().AddInteractiveServerComponents();
// ...
app.MapRazorComponents<App>().AddInteractiveServerRenderMode();
```
В `Components` — корневой компонент `App.razor`, компонент `Routes`. В `Components/Pages` — компоненты-страницы. В `Components/Shared` (Layout) — макет, использование компонента `NavMenu`, выражение `@Body`. Найти использование `NavLink` в `NavMenu` для переключения страниц. Изучить `_Imports.razor`.

**2.4. Задание 2.** Запустить `XXX.Blazor.SSR`, открыть Developer Tools → Network — убедиться, что при переключении страниц **не отправляются HTTP-запросы**. На вкладке Console/Messages — убедиться, что между браузером и сервером открыто соединение **WebSocket**.

**2.5. Задание 3.** На странице `Counter` — поле ввода и кнопка «Установить»: по нажатию счётчику присваивается введённое значение (целое положительное число **от 1 до 10**), с валидацией и выводом сообщения об ошибке. Рекомендации: компонент **`EditForm`**, инициализация в обработчике `OnValidSubmit`; для ввода — **`<InputNumber>`**; правила валидации — вспомогательный класс со свойством `int` и атрибутом `[Range]`; для вывода ошибок — **`<DataAnnotationsValidator/>`** и **`<ValidationSummary/>`**.

**2.6. Задание 4.** Реализовать передачу начального значения счётчика через строку запроса.

**3. Вопросы для самопроверки:** 1. Что такое компоненты Blazor? 2. Как передаются данные между клиентом и сервером при `@rendermode InteractiveServer`? 3. Как передать данные компоненту? 4. Как установить привязку данных в компоненте? 5. Когда возникает `OnParameterSetAsync`? 6. Как обработать событие `click` кнопки компонента? 7. Когда и для чего используется `[StreamRendering(true)]`? 8. Когда выполняется привязка данных `<input>` к свойству в коде компонента? 9. Для чего используется компонент `Router`? 10. Как в компоненте получить значение сегмента маршрута? 11. Для чего `NavigationManager`? 12. Как дочерний компонент передаёт данные родительскому?

### Теория для выполнения ЛР11

#### Что такое Blazor и зачем он нужен

*Определение.* **Blazor** — платформа для создания интерактивного клиентского веб-интерфейса на **.NET/C# вместо JavaScript**: общий код сервер+клиент, рендер в HTML/CSS для широкой поддержки браузеров, интеграция с Docker и т.д. *Где пригодится: ЛР11–ЛР13 целиком построены на Blazor; сравнение SSR/WASM — топ-тема аттестации* **[Атт. №49–№60]**.

**Модели хостинга Blazor (до .NET 7 включительно):**
- **Blazor Server** — компоненты выполняются на сервере, обновления UI идут через **SignalR/WebSocket**-соединение; маленький размер загрузки, быстрый старт, полный доступ к серверным API и инструментам отладки .NET, работает на «тонких» клиентах (без поддержки WebAssembly), но требует постоянного соединения с сервером и не работает офлайн, код на сервере не раскрывается клиенту.
- **Blazor WebAssembly (WASM)** — .NET-рантайм компилируется в **WebAssembly** и выполняется прямо в браузере; после загрузки работает даже при отключении сервера (или вовсе без него — например, при раздаче через CDN); но: больше размер загрузки, дольше первая загрузка, ограничения браузерной песочницы, клиентский код виден и потенциально модифицируем пользователем.

**Режимы рендеринга (начиная с .NET 8, RenderMode)** — Blazor Web App объединяет обе модели под одной крышей:

| Понятие | Значение |
|---|---|
| **Статический рендеринг** | компонент рендерится без интерактивности — HTML отправляется один раз, обработчики .NET-событий не подключены |
| **Интерактивный рендеринг** | компонент обрабатывает .NET-события и привязку — либо на сервере (Interactive Server), либо в браузере (Interactive WebAssembly), либо гибридно (Interactive Auto) |
| **CSR** (Client-Side Rendering) | итоговая HTML-разметка генерируется .NET WebAssembly-рантаймом в браузере; CSR по определению всегда интерактивен |
| **SSR** (Server-Side Rendering) | итоговая HTML-разметка генерируется ASP.NET Core на сервере и отправляется клиенту; бывает статическим или интерактивным |
| **Пререндеринг** | первичная отрисовка контента на сервере без обработчиков событий — ускоряет отклик и помогает **SEO** (Search Engine Optimization — поисковой оптимизации: чтобы поисковые системы могли проиндексировать содержимое страницы); всегда сопровождается финальным рендером на клиенте/сервере; включён по умолчанию для интерактивных компонентов |

*Где пригодится: именно эта классификация объясняет разницу между `XXX.Blazor.SSR` (ЛР11, `AddInteractiveServerComponents`) и `XXX.BlazorWasm` (ЛР12, отдельный WASM-хостинг standalone).* **[Атт. №49, №51]**

#### Структура проекта Blazor Web App (SSR / Interactive Server)

- **`App.razor`** — корневой компонент с HTML-разметкой `<head>`, компонентом `<Routes/>` и тегом `<script src="_framework/blazor.web.js">`.
- **`Routes.razor`** — настраивает маршрутизацию через компонент **`<Router>`**: для интерактивных компонентов перехватывает навигацию браузера и отображает страницу по адресу.
```razor
<Router AppAssembly="typeof(Program).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="routeData" DefaultLayout="typeof(Layout.MainLayout)" />
        <FocusOnNavigate RouteData="routeData" Selector="h1" />
    </Found>
</Router>
```
- **`MainLayout.razor`** (`Components/Layout`) — макет: `@inherits LayoutComponentBase`, содержит `<NavMenu/>` и `@Body` (аналог `@RenderBody()` из MVC).
- **`NavMenu.razor`** — меню, использует компонент **`<NavLink>`** (подсвечивает активный пункт, атрибут `Match="NavLinkMatch.All"`).
- **`_Imports.razor`** — общие директивы `@using` для всех компонентов `.razor` папки (аналог `_ViewImports.cshtml`).
- **`Program.cs`**:
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddRazorComponents().AddInteractiveServerComponents(); // или .AddInteractiveWebAssemblyComponents()
var app = builder.Build();
app.UseHttpsRedirection(); app.UseStaticFiles(); app.UseAntiforgery();
app.MapRazorComponents<App>().AddInteractiveServerRenderMode(); // или .AddInteractiveWebAssemblyRenderMode()
app.Run();
```
*Где пригодится: прямой вопрос ЛР11 задание 2.3, аттестация* **[Атт. №50]**.

#### Компонент EditForm и встроенные Input-компоненты (для задания 2.5)

*Определение.* **`EditForm`** — компонент Blazor для форм: привязка к модели (с поддержкой Data Annotations), автоматически создаёт **`EditContext`**, отслеживающий изменения полей/состояние валидации/ошибки; поддерживает встроенные Input-компоненты. *Где пригодится: прямое требование ЛР11 п.2.5 (валидация счётчика 1..10).* **[Атт. №53]**

```csharp
public class CounterInputModel { [Range(1, 10, ErrorMessage = "Введите число от 1 до 10")] public int Value { get; set; } }
```
```razor
<EditForm Model="@inputModel" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />
    <InputNumber @bind-Value="inputModel.Value" />
    <button type="submit">Установить</button>
</EditForm>
@code {
    private CounterInputModel inputModel = new();
    private int currentCount;
    private void HandleValidSubmit() { currentCount = inputModel.Value; }
}
```
Таблица встроенных Input-компонентов (полезно помнить наизусть):

| Компонент | Рендерится как |
|---|---|
| `InputText` | `<input>` |
| `InputTextArea` | `<textarea>` |
| `InputSelect<TValue>` | `<select>` |
| `InputNumber<TValue>` | `<input type="number">` |
| `InputCheckbox` | `<input type="checkbox">` |
| `InputDate<TValue>` | `<input type="date">` |
| `InputRadioGroup`/`InputRadio` | группа радиокнопок |

`form` vs `EditForm` (частый вопрос): обычный `<form>` требует ручной привязки и валидации; `EditForm` даёт двустороннюю привязку к модели, автоматический `EditContext`, готовые компоненты валидации.

#### Query string → параметр компонента (задание 2.6)

Значение из строки запроса — через параметр маршрута или через `[SupplyParameterFromQuery]`:
```razor
@page "/counter"
@code {
    [SupplyParameterFromQuery] public int? Start { get; set; }
    protected override void OnInitialized() { currentCount = Start ?? 0; }
}
```
или через сегмент маршрута — см. пример `@page "/counter/{start:int?}"` в разделе «Дополнительные темы → Компоненты Blazor».

> **Углублённая теория Blazor** (жизненный цикл компонента, параметры/дочерние компоненты, каскадные параметры, привязка данных `@bind`, встроенные компоненты EditForm/QuickGrid, аутентификация в Blazor, обзор компонентных библиотек) вынесена отдельным блоком в раздел **«Дополнительные темы» → «Blazor: углублённые материалы»** в конце конспекта, т.к. соответствующие лекционные файлы (`17_0`…`17_4`) не содержат отдельного номера лабораторной работы, а раскрывают темы ЛР11–ЛР13 более подробно. Прежде чем сдавать ЛР11, обязательно прочитайте оттуда подраздел **«Жизненный цикл компонента»** и **«Обработка событий»**.

---

## Лабораторная работа №12: Введение в Blazor WebAssembly

### Текст задания

**Цель.** Изучение проекта Blazor. **Задача.** Создание проекта Blazor, использующего аутентификацию удалённого сервера и получающего данные с API.

**4.1. Исходные данные.** Проект из ЛР11. **4.2. Подготовка проекта.** Добавить проект **Blazor WebAssembly Standalone App**, имя `XXX.BlazorWasm`; аутентификация — **«индивидуальные учётные записи»**. В `Program.cs` — регистрация компонента `App`, сервиса `HttpClient`, настройки OIDC (`builder.Services.AddOidcAuthentication`). `appsettings.json` — в `wwwroot`. Изучить `App.razor` (компонент `Router`), `_Imports.razor`, `wwwroot/index.html` (место монтирования `#app`, подключение `_framework/blazor.webassembly.js`), `MainLayout.razor` (`NavMenu`, `@Body`).

**4.3. Подключение к Keycloak.** В Keycloak создать нового **клиента** для WASM-приложения (аналогично ЛР6). В `appsettings.json`/`appsettings.development.json` (в `wwwroot`):
```json
"Keycloak": {
  "Authority": "http://localhost:8080/auth/realms/[Realm]/",
  "ClientId": "[Id клиента]",
  "MetadataUrl": "http://localhost:8080/realms/[Realm]/.well-known/openid-configuration",
  "ResponseType": "id_token token"
}
```
```csharp
builder.Services.AddOidcAuthentication(options => builder.Configuration.Bind("Keycloak", options.ProviderOptions));
```
Проверить: пункт меню Login → страница входа Keycloak → приветственное сообщение при успехе.

**4.4. Задание 1. Вывод списка объектов.** Страница со списком названий объектов (данные — с API), пункт меню. Запрос к API — в `OnInitializedAsync`.

**4.5. Задание 2. Сервис доступа к данным.** В `Services/IDataService`:
```csharp
public interface IDataService
{
    event Action DataLoaded;
    List<Category> Categories { get; set; }
    List<Dish> Dishes { get; set; }
    bool Success { get; set; }
    string ErrorMessage { get; set; }
    int TotalPages { get; set; }
    int CurrentPage { get; set; }
    Category SelectedCategory { get; set; }
    Task GetProductListAsync(int pageNo = 1);
    Task GetCategoryListAsync();
}
```
Реализация `DataService`; адрес API и размер страницы — из `appsettings.json`; зарегистрировать как **scoped**. Базовый адрес `HttpClient` — задать в `Program.cs`. Формирование URL с query string — класс `QueryString`.

**4.6. Задание 3. Авторизация с JWT.** Запретить неавторизованный доступ к методу списка в контроллере API и к странице списка в `XXX.BlazorWasm`. Токен получить из **`IAccessTokenProvider`**:
```csharp
var tokenRequest = await _tokenProvider.RequestAccessToken();
if (tokenRequest.TryGetToken(out var token)) { /* добавить в заголовок Authorization */ }
```

**4.7. Задание 4. Компонент списка объектов.** Вынести список в отдельный компонент `<DishesList />`. Чтобы компонент «увидел» новые данные — в `DataService` вызывать событие `DataLoaded` после получения данных; компонент подписывается на него методом **`StateHasChanged`** (служебный метод Blazor, который говорит фреймворку «перерисуй этот компонент» — нужен, когда состояние меняется не через обычную привязку `@bind`, а откуда-то извне, например по событию сервиса):
```razor
@inject IDataService DataService
@implements IDisposable
@code {
    protected override void OnInitialized() => DataService.DataLoaded += StateHasChanged;
    public void Dispose() => DataService.DataLoaded -= StateHasChanged;
}
```

**4.8. Задание 5. Фильтрация по категории.** Компонент выбора категории — устанавливает `SelectedCategory` в `DataService` и вызывает `GetProductListAsync`. Можно использовать Bootstrap Dropdown (нужны скрипты `bootstrap.bundle.min.js` — добавить на `index.html`).

**4.9. Задание 6. Переключение страниц списка.** Компонент пейджера (аналог Bootstrap Pagination, но `<button>` вместо `<a>`), кнопка текущей страницы выделена; если страница одна — кнопки не выводятся.

**4.10. Задание 7. Взаимодействие между компонентами.** Компонент подробной информации об объекте; в списке — кнопка, по клику ищется объект по `Id` **на странице** (не в компоненте списка) и передаётся в компонент деталей:
```razor
<DishesList DishSelected="OnDishSelected" />
<DishDetails SelectedDish="SelectedDish" />
@code {
    Dish SelectedDish { get; set; }
    void OnDishSelected(int id) => SelectedDish = DataService.Dishes.FirstOrDefault(d => d.Id == id);
}
```

**5. Контрольные вопросы:** 1. В какой вид компилируется клиентский код Blazor WebAssembly? 2. Чем SPA отличается от обычного веб-приложения? 3. Что такое `CascadingParameter`? 4. Как родительский компонент обрабатывает событие дочернего? 5. Для чего `StateHasChanged`? 6. Как используется `<AuthorizeView/>`? 7. Для чего `AuthenticationStateProvider`? 8. Для чего параметр `Task<AuthenticationState>`? 9. Как ограничить доступ неавторизованных пользователей к странице Blazor?

### Теория для выполнения ЛР12

#### Структура Blazor WebAssembly Standalone

`wwwroot/index.html` — точка входа (не `App.razor`!), содержит `<div id="app">` (место монтирования) и `<script src="_framework/blazor.webassembly.js">`. `Program.cs`:
```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
await builder.Build().RunAsync();
```
`App.razor` (WASM-вариант) содержит `<Router>` с секциями `<Found>`/`<NotFound>`.

*Определение.* **SPA (Single Page Application)** — приложение, загружающее одну HTML-страницу один раз, а дальнейшая навигация/обновление UI идёт без перезагрузки страницы (через JS/WASM-роутинг); в отличие от классического веб-приложения, где каждый переход — новый полный HTTP-запрос страницы. *Где пригодится: прямой вопрос ЛР12* **[Атт. №2 общая часть]**.

#### Аутентификация в Blazor WebAssembly (OIDC/Keycloak)

Компонент **`<CascadingAuthenticationState>`** оборачивает `<Router>` и передаёт состояние аутентификации каскадно всем дочерним компонентам:
```razor
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(App).Assembly">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
                <NotAuthorized>You are not authorized!</NotAuthorized>
            </AuthorizeRouteView>
        </Found>
    </Router>
</CascadingAuthenticationState>
```
**`AuthenticationStateProvider`** — абстрактный класс, поставляющий текущее состояние аутентификации; свой провайдер (например, парсящий JWT) переопределяет `GetAuthenticationStateAsync()`:
```csharp
public class CustomAuthStateProvider : AuthenticationStateProvider
{
    public override async Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        var jwt = "eyJ..."; // токен
        var token = new JwtSecurityToken(jwt);
        var identity = new ClaimsIdentity(token.Claims, "jwt");
        var user = new ClaimsPrincipal(identity);
        var state = new AuthenticationState(user);
        NotifyAuthenticationStateChanged(Task.FromResult(state));
        return state;
    }
}
builder.Services.AddScoped<AuthenticationStateProvider, CustomAuthStateProvider>();
builder.Services.AddAuthorizationCore();
```
*Где пригодится: прямой ответ на контрольные вопросы ЛР12 №6, №7, №8* **[Атт. №53–№60]**. В готовом шаблоне `AddOidcAuthentication` (используемом в самой лабе) всё это уже настроено «из коробки» под Keycloak.

Ограничение доступа к странице: `@attribute [Authorize]` (в начале `.razor`-файла) — аналог `[Authorize]` в MVC. Разное содержимое для авторизованных/нет:
```razor
<AuthorizeView>
    <Authorized><span>@context.User.FindFirst("aud")</span></Authorized>
    <NotAuthorized><span>Your are still not authorized</span></NotAuthorized>
</AuthorizeView>
```

**Получение JWT-токена для вызова API из WASM** — сервис **`IAccessTokenProvider`** (регистрируется автоматически шаблоном с OIDC-аутентификацией):
```csharp
var tokenResult = await AccessTokenProvider.RequestAccessToken();
if (tokenResult.TryGetToken(out var token))
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token.Value);
```

#### Взаимодействие компонентов и вызовы API — см. также «Дополнительные темы»

Передача события от дочернего компонента родителю, `EventCallback`, `[CascadingParameter]`, вызовы `HttpClient`/`GetFromJsonAsync` из Blazor Server/WASM — подробно разобраны в разделе **«Дополнительные темы → Привязка данных в Blazor»** и **«…→ Компоненты Blazor»** в конце конспекта (используются в заданиях 4.7, 4.9, 4.10 этой лабы).

---

## Лабораторная работа №13: Введение в SignalR (командный проект)

> **Важно про нумерацию:** в исходном файле лекции (`13_Введение в signalR`) заголовок ошибочно указывает «Лабораторная работа № 12» — это опечатка автора курса. По порядку файлов (после `12_Введение в Blazor WASM`) и по факту содержания (используются знания из ЛР11/ЛР12) это **тринадцатая и последняя из 13 лабораторных работ** курса. В конспекте она идёт как ЛР13.

### Текст задания

**Цель.** Изучение технологии SignalR. **Задача.** Создание проекта Blazor WebAssembly с реализацией взаимодействия клиентов в реальном времени через SignalR. Время — 6 часов (3 занятия).

**3.1. Постановка задачи.** Разработать **онлайн-игру для не менее 2 игроков** согласно индивидуальному заданию. Работа выполняется **командой** (до 5 участников, можно из разных групп потока), с назначенным старшим для координации. Части разработки (клиент/сервер/интерфейс/бизнес-логика) рекомендуется распределить между участниками. **При защите обязательно указать, кто какую часть выполнял.** Студенты, не участвующие в проекте, сдают зачёт с обязательными дополнительными вопросами по SignalR.

**3.2. Обязательные требования к проекту:**
- Количество игроков — не менее 2.
- Бизнес-логика игры (очерёдность хода, подсчёт результатов, формирование игрового поля) — **на сервере**, в отдельном классе.
- Взаимодействие клиент↔сервер — **по технологии SignalR**, использовать **строго типизированный Hub**.
- Предусмотреть возможность **нескольких игр одновременно** (свойство `Group`).
- Клиентская часть — на **Blazor WebAssembly**; бизнес-логика клиента — в отдельном классе; отрисовка игрового поля — в отдельном компоненте Blazor.

**3.3. Варианты индивидуальных заданий** (простые алгоритмы): 1. Нарды 2. Чак-э-лак (кости) 3. Калах 4. Блэк Джек 5. Реверси 6. Перудо (кости) 7. Крестики-нолики 5×5 8. Бак-дайс (кости) 9. Бура (карты) 10. Свинья (кости) 11. Двадцать одно (карты) 12. Тузы/Aces (кости).

**3.4. Более сложные алгоритмы** (по желанию/договорённости): Блеф, Яхта, Зонк/Farkle (кости), Морской бой, Тысяча (карты), Тысяча (кости), Покер на костях.

### Теория для выполнения ЛР13

#### Проблема реального времени поверх HTTP

Классический HTTP-запрос — «клиент спрашивает → сервер отвечает», сервер сам не может ничего «протолкнуть» клиенту без нового запроса. Способы имитировать/обеспечить обновления в реальном времени:

| Технология | Принцип | Минусы |
|---|---|---|
| **Polling** (обычный опрос) | клиент постоянно спрашивает «есть новое сообщение?» | много лишних запросов, задержка до следующего опроса |
| **Long Polling** | сервер «придерживает» ответ, пока не появится событие или не истечёт таймаут | сложнее в реализации, но меньше лишнего трафика |
| **Server-Sent Events (SSE)** | однонаправленный поток событий сервер→клиент через один открытый HTTP-запрос (`Content-Type: text/event-stream`), JS API `EventSource` | только сервер→клиент, HTTP/1.1 совместим |
| **WebSocket** | двунаправленный полнодуплексный канал поверх **TCP**-соединения (Transmission Control Protocol — надёжный транспортный протокол интернета, на котором строится и HTTP) | нужна явная поддержка на обеих сторонах |

```javascript
var ws = new WebSocket("ws://localhost:9998/echo");
ws.onopen = function () { ws.send("Message to send"); };
ws.onmessage = function (evt) { var received_msg = evt.data; };
```
```javascript
var source = new EventSource("/getevents");
source.onmessage = function (event) { alert(event.data); };
```

*Определение.* **SignalR** — open-source библиотека ASP.NET Core, упрощающая добавление веб-функций реального времени: сервер может мгновенно передавать контент клиентам. SignalR — это уровень **абстракции** над транспортом: сам выбирает лучший доступный транспорт (WebSocket → SSE → Long Polling, по убыванию предпочтения), скрывая эту сложность от разработчика. *Где пригодится: обязательная технология ЛР13; топ-тема аттестации* **[Атт. №48]**.

Применение SignalR: приложения, требующие частых обновлений с сервера (игры, соцсети, голосования, аукционы, карты/GPS), панели мониторинга, совместные приложения (доски, групповые собрания), системы уведомлений.

Уровни абстракции: **SignalR → Hubs (высокоуровневый RPC) → Транспорт (WebSockets / Server-Sent Events / Long Polling) → Интернет-протокол**.

#### Hub — центральный API SignalR

*Определение.* **Hub** — высокоуровневый конвейер, реализующий модель **двунаправленного RPC** (Remote Procedure Call): клиент может вызывать методы сервера, и наоборот сервер может вызывать методы клиента. *Где пригодится: буквально «Hub» — ключевое требование ЛР13 («использовать строго типизированный Hub»).*

```csharp
public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message)
        => await Clients.All.SendAsync("ReceiveMessage", user, message);
}
```
**Строго типизированный Hub** (обязателен по заданию) — вместо магических строк-имён методов клиента используется интерфейс:
```csharp
public interface IChatClient { Task ReceiveMessage(string user, string message); }
public class ChatHub : Hub<IChatClient>
{
    public async Task SendMessage(string user, string message)
        => await Clients.All.ReceiveMessage(user, message); // вызов метода интерфейса — компилятор проверит сигнатуру
}
```
Любой **публичный** метод Hub доступен вызову извне — это потенциальная угроза безопасности (не делать публичными служебные методы).

**Адресаты отправки сообщений** (`Clients.*`):

| Свойство | Кому уходит сообщение |
|---|---|
| `Clients.All` | всем подключённым клиентам |
| `Clients.Caller` | только вызывающему клиенту |
| `Clients.Others` | всем, кроме вызывающего |
| `Clients.Connection(connectionId)` | конкретному соединению |
| `Clients.AllExcept(connections)` | всем, кроме перечисленных |
| `Clients.Group(groupName, excludeConnectionIds)` | участникам группы |
| `Clients.User(userName)` | конкретному пользователю (по всем его соединениям) |

**Контекст соединения** — `Context.ConnectionId` (уникальный id соединения), `Context.UserIdentifier` (по умолчанию `ClaimTypes.NameIdentifier` из `ClaimsPrincipal`). События подключения: `OnConnectedAsync()`, `OnDisconnectedAsync(bool stopCalled)`.

**Группы** — обязательное требование ЛР13 (для нескольких одновременных партий):
```csharp
await Groups.AddToGroupAsync(Context.ConnectionId, groupId);
await Clients.Group(groupId).BoardUpdated(board, currentTurn);
```

Регистрация Hub'а в `Program.cs`:
```csharp
builder.Services.AddSignalR();
// ...
app.MapHub<ChatHub>("/chathub");
```

#### Клиент SignalR: JavaScript и Blazor WebAssembly

**JavaScript-клиент**:
```html
<script src="~/lib/signalr/dist/browser/signalr.js"></script>
```
```javascript
var connection = new signalR.HubConnectionBuilder().withUrl("/chathub").build();
connection.on("ReceiveMessage", function (user, message) { /* обработка */ });
connection.start().catch(err => console.error(err.toString()));
connection.invoke("SendMessage", user, message).catch(err => console.error(err.toString()));
```

**Blazor WebAssembly-клиент** (то, что нужно для ЛР13) — пакет `Microsoft.AspNetCore.SignalR.Client`:
```csharp
@using Microsoft.AspNetCore.SignalR.Client
@inject NavigationManager NavigationManager
@implements IDisposable
@code {
    HubConnection connection;
    protected override async Task OnInitializedAsync()
    {
        connection = new HubConnectionBuilder()
            .WithUrl(NavigationManager.ToAbsoluteUri("/ttt"))
            .WithAutomaticReconnect()
            .Build();
        connection.On<GameCell[][], GameCellStyle>("BoardUpdated", (b, d) => BoardUpdated(b, d));
        connection.On<GameOverInfo>("GameOver", i => GameOver(i));
        connection.On<string, GameCellStyle>("Registered", (id, s) => Registered(id, s));
        await connection.StartAsync();
    }
    async Task CellClicked(int x, int y)
    {
        if (!CanMove()) return;
        await connection.InvokeAsync("PlayerTurn", GroupId, x, y);
    }
    void Registered(string id, GameCellStyle style) { GroupId = id; PlayerStyle = style; StateHasChanged(); }
    public async void Dispose() => await connection.DisposeAsync();
}
```
На сервере (пример из лекции — крестики-нолики, `TttHub`):
```csharp
public class TttHub : Hub<TttClient>
{
    static Dictionary<string, GameGroup> groups = new();
    public async Task Register(string name)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupId);
        await Clients.Caller.Registered(groupId, player.PlayerStyle);
    }
    public async Task PlayerTurn(string groupId, int x, int y)
    {
        await Clients.Group(groupId).BoardUpdated(game.Board.Board, game.Board.CurrentTurn);
        if (game.Board.GameComplete) await Clients.Group(groupId).GameOver(game.Board.GetWinner());
    }
}
```
Регистрация сервиса и endpoint'а на сервере (важно для standalone WASM-хостинга — сжатие ответов, fallback на `index.html`):
```csharp
builder.Services.AddResponseCompression(opts =>
    opts.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[] { "application/octet-stream" }));
builder.Services.AddSignalR();
// ...
app.UseResponseCompression();
app.MapRazorPages(); app.MapControllers();
app.MapFallbackToFile("index.html");
app.MapHub<TttHub>("/ttt");
```

**Архитектурный совет для командного проекта (обязательное требование п.3.2):** бизнес-логику игры выносите в **отдельный класс, не зависящий от SignalR** (например `GameEngine`/`TicTacToeBoard`) — Hub должен быть тонким и только вызывать методы этого класса и рассылать результат через `Clients.Group(...)`. Аналогично на клиенте — не мешайте логику игры с кодом компонента Blazor (компонент только отображает состояние и вызывает `connection.InvokeAsync`). Это прямо облегчает описание «кто что делал» на защите и тестирование бизнес-логики (см. также ЛР9 — модульные тесты).

---

# Дополнительные темы (без отдельной лабораторной работы)

Ниже — темы из лекционных файлов `14_GraphQL`, `15_gRPC`, `16_signalR` (расширенная теория) и `17_0`…`17_4` (углублённый Blazor). Ни в одном из этих файлов **не обнаружено** встроенного текста лабораторной работы («Лабораторная работа №…», «Задача работы», «Выполнение работы») — это чисто лекционный материал, поэтому по требованиям задачи он вынесен сюда, а не оформлен как «ЛР14»/«ЛР15» и т.п. Заказчик ожидал именно 13 лабораторных работ — это подтвердилось: расхождений с ожиданием не обнаружено, а GraphQL/gRPC/расширенный Blazor — это темы «сверху» для общего кругозора и подготовки к аттестации (в списке 60 вопросов GraphQL и gRPC не упоминаются вовсе, а Blazor — вопросы №49–60).

## GraphQL

*Определение.* **GraphQL** — язык запросов для API и среда выполнения для обработки этих запросов; разработан Facebook (2012, публично — 2015). Радикально отличается от REST: вместо множества конечных точек с фиксированной структурой ответа — как правило **одна конечная точка**, а структуру возвращаемых данных выбирает **клиент**. *Где пригодится: не является заданием отдельной лабы в этом курсе, но входит в общий кругозор «современных платформ»; полезно на собеседованиях и в реальных проектах с сложными/гибкими запросами данных.*

**Когда выбирать GraphQL:** ограниченная пропускная способность (минимизировать количество запросов/ответов); несколько источников данных, которые нужно объединить в одном адресе; сильно различающиеся запросы клиентов, ожидающие разные ответы.
**Когда выбирать REST:** небольшие приложения с несложными данными; данные/операции, одинаковые для всех клиентов; нет требований к сложным запросам.

**Query** (получение данных) и **Mutation** (изменение данных: создание/обновление/удаление):
```graphql
query {
  students(where: { groupId: { eq: 1 } }) {
    fullName
    birthDay
  }
}
```
```graphql
mutation {
  addStudent(input: { fullName: "Goga" }) {
    student { id fullName birthDay }
  }
}
```
Ответ сервера — JSON вида `{ "data": { ... } }` или `{ "errors": [ { "message": ..., "path": [...] } ] }`.

**Реализация в .NET — ChilliCream Hot Chocolate** (`https://chillicream.com/docs/hotchocolate`):
```csharp
public class Query
{
    private readonly AppDbContext _context;
    public Query(IDbContextFactory<AppDbContext> factory) => _context = factory.CreateDbContext();
    [UseProjection] [UseFiltering] [UseSorting]
    public IQueryable<Dish> Dishes() => _context.Dishes;
}
public record AddDishInput(string Name, string Description, int Calories, int CategoryId);
public record DishPayload(Dish dish);
public class Mutation
{
    public async Task<DishPayload> AddDish(AddDishInput input)
    {
        var dish = new Dish { Name = input.Name, Description = input.Description, Calories = input.Calories, CategoryId = input.CategoryId };
        _context.Dishes.Add(dish);
        await _context.SaveChangesAsync();
        return new DishPayload(dish);
    }
}
// Program.cs:
builder.Services.AddPooledDbContextFactory<AppDbContext>(opt => opt.UseSqlite(connStr));
builder.Services.AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddProjections().AddFiltering().AddSorting();
app.MapGraphQL();
```
`[UseProjection]` — сервер сам выберет из БД только запрошенные клиентом поля; `[UseFiltering]`/`[UseSorting]` — автоматически поддерживают фильтрацию (`where`) и сортировку (`order`) без ручного кода.

Библиотеки .NET-клиентов: **ZeroQL**, **GraphQLinq**.
```csharp
var client = new MenuClient(httpClient);
var filter = new DishFilterInput { Id = new IntOperationFilterInput { Gt = 2 } };
var response = await client.Query(o => o.Dishes(where: filter, null, d => new { d.Id, d.Name }));
```

## gRPC

*Определение.* **gRPC** — независимая от языка высокопроизводительная платформа **RPC** (Remote Procedure Call), разработана Google (2015). Транспорт — **HTTP/2**; язык описания интерфейса (IDL) — **Protocol Buffers (protobuf)**. Подход «контракт вперёд» (contract-first): сначала пишется `.proto`-файл, из него генерируется код для сервера/клиента на любом поддерживаемом языке.

**Файл `.proto`** содержит: определение службы (`service`) и сообщения (`message`), передаваемые между клиентом и сервером:
```proto
syntax = "proto3";
option csharp_namespace = "GrpcServer";
service Menu {
  rpc GetDishInfo (DishInfoRequest) returns (DishResponse);
  rpc GetDishesList (DishListRequest) returns (stream DishResponse); // серверный потоковый вызов
}
message DishInfoRequest { int32 Id = 1; }
message DishListRequest {}
message DishResponse { int32 Id=1; string Name=2; string Description=3; int32 Calories=4; string Image=5; int32 CategoryId=6; }
```

**Сервер** — пакет `Grpc.AspNetCore`, `.proto` подключается как `<Protobuf Include="Protos\dishes.proto" GrpcServices="Server"/>`, после компиляции появляются базовые классы:
```csharp
public class MenuService : Menu.MenuBase
{
    public override async Task<DishResponse> GetDishInfo(DishInfoRequest request, ServerCallContext context)
    {
        var dish = await _context.Dishes.FindAsync(request.Id);
        return new DishResponse { Id = dish.Id, Name = dish.Name, Description = dish.Description, Calories = dish.Calories, CategoryId = dish.CategoryId };
    }
    public override async Task GetDishesList(DishListRequest request, IServerStreamWriter<DishResponse> responseStream, ServerCallContext context)
    {
        foreach (var dish in _context.Dishes)
            await responseStream.WriteAsync(new DishResponse { Id = dish.Id, Name = dish.Name });
    }
}
// Program.cs:
builder.Services.AddGrpc();
app.MapGrpcService<MenuService>();
```
`appsettings.json`: `"Kestrel": { "EndpointDefaults": { "Protocols": "Http2" } }`.

**Клиент** — пакеты `Grpc.Net.Client`, `Google.Protobuf`, `Grpc.Tools`, тот же `.proto` с `GrpcService="Client"`:
```csharp
var channel = GrpcChannel.ForAddress("http://localhost:5085");
var client = new Menu.MenuClient(channel);
var response = client.GetDishesList(new DishListRequest());
while (await response.ResponseStream.MoveNext())
    Console.WriteLine($"{response.ResponseStream.Current.Name}");
```

**gRPC и браузер:** службу gRPC **нельзя вызвать напрямую из браузера** (использует специфику HTTP/2, недоступную обычному браузерному API) — решения: **gRPC-Web** (спец. клиент + протокол, совместимый с HTTP/1.1) или **перекодирование gRPC-JSON** (с .NET 7+, служба выглядит как обычный REST/JSON API).

**Сравнение gRPC и HTTP API с JSON (REST)** — таблица из лекции, важна для сравнительных вопросов:

| Свойство | gRPC | HTTP API с JSON (REST) |
|---|---|---|
| Контракт | обязателен (`.proto`) | необязателен (OpenAPI) |
| Протокол | HTTP/2 | HTTP (любая версия) |
| Payload | Protobuf (малый, бинарный) | JSON (больше, но человекочитаемый) |
| Строгость | строгая спецификация | нестрогая, допустимы любые HTTP-соглашения |
| Потоковые операторы | клиент, сервер, оба направления | клиент, сервер |
| Поддержка браузеров | нет (нужен gRPC-Web) | да |
| Безопасность | транспортная (TLS) | транспортная (TLS) |
| Генерация клиентского кода | да, «из коробки» | OpenAPI + сторонние инструменты |

Применение gRPC: микросервисы, взаимодействие точка-точка в реальном времени, разноязыковые среды, среды с ограниченными сетевыми ресурсами, межпроцессное взаимодействие (IPC). Недостатки: ограниченная поддержка браузера, payload не читается человеком напрямую (нужны спец. инструменты и знание `.proto`).

## REST vs GraphQL vs gRPC — сводная таблица для сравнения (пригодится на аттестации)

| | REST | GraphQL | gRPC |
|---|---|---|---|
| Транспорт | HTTP/1.1+ | HTTP (обычно POST на 1 endpoint) | HTTP/2 |
| Формат данных | JSON (обычно) | JSON | Protobuf (бинарный) |
| Число конечных точек | много (по ресурсам) | как правило одна | по числу RPC-методов в `.proto` |
| Гибкость запроса клиента | фиксированная структура ответа | клиент сам выбирает поля | фиксированная схема сообщений |
| Контракт | необязателен (можно описать OpenAPI) | схема GraphQL обязательна | `.proto` обязателен |
| Потоковая передача | нет (только через SignalR/SSE отдельно) | нет «из коробки» (есть subscriptions) | да (клиент/сервер/двунаправленный стрим) |
| Поддержка браузера | полная | полная | нет напрямую (нужен gRPC-Web) |
| Где используется в курсе | ЛР4, ЛР6, ЛР8, ЛР12 (основной API) | доп. тема, не в лабах | доп. тема, не в лабах |
| Типичный сценарий | публичные web API, CRUD | сложные/агрегированные запросы, мобильные клиенты с ограниченным трафиком | межсервисное взаимодействие, микросервисы, высокая производительность |

## SignalR — расширенная теория (в дополнение к разделу ЛР13)

*(В файле `16_signalR.txt` материал полностью совпадает по содержанию с тем, что уже разобрано в разделе ЛР13 — Hubs, группы, JS/Blazor-клиенты. Уникальное дополнение — сравнение технологий реального времени, уже приведено в разделе ЛР13 в виде таблицы Polling/Long Polling/SSE/WebSocket.)*

## Blazor: углублённые материалы (файлы 17_0–17_4)

### Жизненный цикл компонента Blazor

*(Слайды лекции по этой теме почти не содержат текста — только заголовки и диаграммы; поэтому раздел ниже дополнен актуальными сведениями по документации ASP.NET Core, необходимыми для полноценной сдачи ЛР11–ЛР13.)*

При создании и рендеринге компонента Blazor последовательно вызываются следующие методы (можно переопределять):

| Метод | Когда вызывается | Типичное использование |
|---|---|---|
| `SetParametersAsync` | первым, при установке параметров компонента | редко переопределяется вручную |
| `OnInitialized` / `OnInitializedAsync` | один раз, при создании компонента (после установки параметров) | первичная загрузка данных, подписка на события сервиса |
| `OnParametersSet` / `OnParametersSetAsync` | после `OnInitialized`, а также при **каждом** обновлении параметров от родителя | реакция на изменение входных параметров |
| `ShouldRender` | перед каждым рендером (кроме первого) | оптимизация — вернуть `false`, чтобы пропустить лишний рендер |
| `OnAfterRender` / `OnAfterRenderAsync(firstRender)` | после того как компонент отрисован в DOM | работа с JS-интеропом, установка фокуса, инициализация JS-виджетов (проверять `firstRender`) |
| `Dispose` / `DisposeAsync` (`IDisposable`/`IAsyncDisposable`) | при удалении компонента из дерева | отписка от событий (как в ЛР12, `DataService.DataLoaded -= StateHasChanged`), закрытие `HubConnection` (как в ЛР13) |

*Где пригодится: `OnInitializedAsync` — где грузить данные при открытии страницы (ЛР11 задание 3, ЛР12 задание 1); `OnParametersSet` — прямой вопрос контрольных вопросов ЛР11 (№5: «когда возникает OnParameterSetAsync»); `Dispose` — обязателен при подписке на события/SignalR, иначе — утечка памяти.* **[Атт. №49–№60]**

**`[StreamRendering(true)]`** (атрибут страницы) — включает потоковый рендеринг: страница сначала отдаётся с индикатором загрузки (`@if (forecasts == null) { <p>Loading...</p> }`), а когда асинхронные данные готовы — обновляется без блокировки первого ответа сервера. *Где пригодится: прямой вопрос контрольных ЛР11 №7.*
```razor
@page "/weather"
@attribute [StreamRendering]
@if (forecasts == null) { <p><em>Loading...</em></p> } else { /* таблица */ }
@code {
    protected override async Task OnInitializedAsync() { await Task.Delay(3000); forecasts = await LoadAsync(); }
}
```

### Параметры компонента, ChildContent, ссылки на компоненты, generic-компоненты

**Параметр** — свойство с атрибутом `[Parameter]`, значение передаётся из родителя как атрибут тега:
```razor
<!-- Дочерний CounterDisplay.razor -->
<p role="status">Current count: @currentCount</p>
@code {
    [Parameter] public int StartValue { get; set; }
    int currentCount;
    protected override void OnParametersSet() => currentCount = StartValue;
}
<!-- Родитель -->
<CounterDisplay StartValue="Start"/>
```
Параметр из сегмента маршрута:
```razor
@page "/counter/{start:int?}"
@code { [Parameter] public int Start { get; set; } = 0; }
```
**`ChildContent`** (`RenderFragment`) — содержимое между открывающим и закрывающим тегом компонента:
```razor
<div class="card"><div class="card-body">@ChildContent</div></div>
@code { [Parameter] public RenderFragment? ChildContent { get; set; } }
```
**Ссылка на компонент** (`@ref`) — вызов публичного метода дочернего компонента из родителя напрямую (минуя параметры):
```razor
<CounterDisplay StartValue="Start" @ref="@counterDisplay"/>
<button @onclick="IncrementCount">Click me</button>
@code {
    CounterDisplay counterDisplay { get; set; }
    private void IncrementCount() => counterDisplay.IncrementCount(); // публичный метод дочернего компонента
}
```
**Generic-компоненты** — `@typeparam TItem` (или `@typeparam TEntity where TEntity : class`); при использовании нужно явно указать тип (`TExample="string"`), либо компилятор выведет его сам по переданным данным.

**Каскадные параметры (`CascadingParameter`)** — способ «прокинуть» значение сразу многим потомкам без передачи через каждый уровень параметров вручную. *Где пригодится: прямой вопрос контрольных ЛР12 №3.*
```razor
<!-- через компонент CascadingValue -->
<CascadingValue Value="GameStatus">
    <CounterDisplay />
</CascadingValue>
<!-- получение в потомке -->
@code { [CascadingParameter] public GameStatus? gameStatus { get; set; } }
```
Или через DI (регистрация каскадного значения глобально для всего приложения, .NET 8+):
```csharp
builder.Services.AddCascadingValue(sp => new GameStatus());
builder.Services.AddCascadingValue("AiGamer", sp => new GameStatus { Gamer2 = "Computer" });
// получение по имени:
[CascadingParameter(Name = "AiGamer")] public GameStatus? aiGameStatus { get; set; }
```

### Встроенные компоненты Blazor (расширение раздела ЛР11)

**`EditForm` vs `<form>`** (сравнение из лекции):

| | `<form>` | `EditForm` |
|---|---|---|
| Привязка к модели | нет автоматической | да, двусторонняя (`Model`/`EditContext`) |
| Отслеживание изменений/валидации | нет | да, через `EditContext` |
| Встроенные Input-компоненты | нет | да (`InputText`, `InputSelect`, `ValidationSummary`...) |
| Data Annotations | нужно проверять вручную | поддерживаются через `<DataAnnotationsValidator/>` |

Пример с `EditContext` напрямую (более гибкий контроль над валидацией):
```razor
<EditForm EditContext="editContext" FormName="ExampleForm" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator /> <ValidationSummary />
    <InputText id="name" @bind-Value="exampleModel.Name" />
</EditForm>
@code {
    EditContext? editContext;
    [SupplyParameterFromForm] private ExampleModel? exampleModel { get; set; }
    protected override void OnInitialized() { exampleModel ??= new() { Name = "User" }; editContext = new(exampleModel); }
}
```
Радиокнопки:
```razor
<InputRadioGroup Name="Radio" TValue="int" @bind-Value="Radio">
    <InputRadio Value="1">Radio1</InputRadio>
    <InputRadio Value="2">Radio2</InputRadio>
</InputRadioGroup>
```
**Секции (`SectionOutlet`/`SectionContent`)** — позволяют странице вставлять контент в именованное место макета (аналог `@RenderSection` из MVC, но динамически, во время выполнения):
```razor
<!-- в макете -->
<SectionOutlet SectionName="top-bar" />
<!-- на странице -->
<SectionContent SectionName="top-bar"><button @onclick="IncrementCount">Click me</button></SectionContent>
```
**`QuickGrid`** — официальный компонент-таблица от Microsoft для быстрого вывода данных с сортировкой/постраничностью без написания HTML-таблицы вручную (пакет `Microsoft.AspNetCore.Components.QuickGrid`) — альтернатива ручной вёрстке `<table>` из ЛР3/ЛР12.

### Привязка данных (`@bind`) и обработка событий

**Базовая привязка** — атрибут `@bind`:
```razor
<input type="text" @bind="TextToInput"/>
@code { private string TextToInput { get; set; } }
```
По умолчанию значение обновляется по событию `onchange`; для обновления «по мере ввода» — `@bind:event="oninput"`. Для атрибута, отличного от `value`, — `@bind-{ATTRIBUTE}`. **`@bind:after`** — выполнить метод сразу после обновления привязанного значения (упрощает код по сравнению с ручной подпиской на `onclick`+вызов метода):
```razor
<input @bind="@searchSummary" @bind:event="oninput" @bind:after="Search" />
```

**Привязка параметров родитель↔дочерний** — соглашение `Property` + `PropertyChanged` (аналог `@bind-Value`/`ValueChanged` в стандартных компонентах):
```razor
<!-- Дочерний -->
@code {
    [Parameter] public int Age { get; set; }
    [Parameter] public EventCallback<int> AgeChanged { get; set; }
}
<!-- Родитель -->
<ChildComponent @bind-Age="ParentAge" />
```
От дочернего к родительскому (пароль, например):
```razor
<input @oninput="OnPasswordChanged" type="password" value="@Password" />
@code {
    [Parameter] public string Password { get; set; }
    [Parameter] public EventCallback<string> PasswordChanged { get; set; }
    private Task OnPasswordChanged(ChangeEventArgs e) { Password = e.Value.ToString(); return PasswordChanged.InvokeAsync(Password); }
}
```
`@bind:get`/`@bind:set` — полный контроль над чтением/записью привязанного значения (гибче, чем стандартный `@bind-Value`).

**Обработка событий** — атрибут `@on{EVENT}` (например `@onclick`):
```razor
<button @onclick="DoSomethingHandler">Do something</button>
@code { private void DoSomethingHandler(MouseEventArgs e) { ... } } // или async Task
```
Передача дополнительных параметров в обработчик — через лямбду:
```razor
@for (var i = 1; i < 4; i++)
{
    var buttonNumber = i;
    <button @onclick="@(e => DoSomething(e, buttonNumber))">Button @i</button>
}
```
Таблица типов аргументов события (важно помнить хотя бы категории): `MouseEventArgs` (`onclick`, `ondblclick`, `onmousedown`...), `KeyboardEventArgs` (`onkeydown`, `onkeyup`...), `ChangeEventArgs` (`onchange`, `oninput`), `FocusEventArgs` (`onfocus`, `onblur`), `ClipboardEventArgs`, `DragEventArgs`, `TouchEventArgs`, `PointerEventArgs`, `WheelEventArgs`, `ProgressEventArgs`, обобщённый `EventArgs`.

**`EventCallback`** — механизм передачи события из дочернего компонента родителю (типичный сценарий — клик в дочернем компоненте должен запустить метод родителя):
```razor
<!-- дочерний DishesList -->
@foreach (var dish in Dishes)
{
    <button @onclick="@(e => Selected(e, dish.DishId))">@dish.DishName</button>
}
@code {
    [Parameter] public EventCallback<int> SelectedIdChanged { get; set; }
    private void Selected(MouseEventArgs e, int id) => SelectedIdChanged.InvokeAsync(id);
}
<!-- родитель -->
<DishesList @bind-Dishes="dishes" SelectedIdChanged="ShowDetails"></DishesList>
@code { private async Task ShowDetails(int id) { ... } }
```
*Где пригодится: прямая реализация задания ЛР12 п.4.10 (взаимодействие между компонентами).* **[Атт. №57, №58]**

**Бизнес-логика в отдельном сервисе** (а не в компоненте) — сервис инжектируется через `@inject`, компонент подписывается на событие изменения:
```csharp
public class CalcService
{
    int _result;
    public int Result { get => _result; set { if (_result == value) return; _result = value; ResultChanged?.Invoke(this, null); } }
    public void Increment() => Result++;
    public event EventHandler? ResultChanged;
}
builder.Services.AddScoped<CalcService>();
```
```razor
@inject CalcService Calc
<p>Current count: @Calc.Result</p>
@code { protected override void OnInitialized() => Calc.ResultChanged += (o, e) => StateHasChanged(); }
```
*Где пригодится: ровно такой паттерн используется в `IDataService`/`DataService` ЛР12 (событие `DataLoaded`).*

**Запросы к API-сервисам из Blazor** — отличаются между хостинг-моделями:
- **Blazor Server** — `HttpClient` создаётся через `IHttpClientFactory` (у сервера нет «единого» предварительно настроенного `HttpClient` под конкретный API):
```csharp
builder.Services.AddHttpClient();
@inject IHttpClientFactory clientFactory
var client = clientFactory.CreateClient();
var response = await client.GetAsync(apiBaseAddress);
```
- **Blazor WebAssembly** — `HttpClient` предварительно настроен на `BaseAddress` приложения (регистрируется в `Program.cs`, см. раздел ЛР12), обычно просто:
```razor
@inject HttpClient client
dishes = await client.GetFromJsonAsync<IEnumerable<ListViewModel>>(apiBaseAddress);
```

### Библиотеки компонентов Blazor (обзор)

Готовые наборы UI-компонентов для Blazor (не входят в лабораторные работы, но полезны для реальных проектов и общего кругозора):

| Библиотека | Ссылка |
|---|---|
| **Blazorise** | https://blazorise.com/ |
| **MudBlazor** | https://mudblazor.com/ |
| **Radzen Blazor** | https://blazor.radzen.com/ |
| **Microsoft FluentUI Blazor** | https://www.fluentui-blazor.net/ |

---

# Сводные таблицы для быстрого повторения

**IActionResult — типы результатов действий контроллера**

| Результат | Возвращает |
|---|---|
| `Ok()` / `Ok(value)` | 200, опционально тело |
| `Created(...)`/`CreatedAtAction(...)`/`CreatedAtRoute(...)` | 201 + заголовок `Location` |
| `BadRequest()` | 400 |
| `Unauthorized()` | 401 |
| `NotFound()` | 404 |
| `View()`/`View(model)` | HTML-представление |
| `PartialView()` | частичное HTML-представление |
| `Json(value)` | JSON |
| `File(...)`/`PhysicalFile(...)`/`VirtualFile(...)` | файл |
| `Redirect(url)` / `RedirectToAction(...)` | 302/301 + `Location` |

**ViewData vs ViewBag vs TempData vs Model** — см. таблицу в разделе ЛР2.

**Жизненные циклы DI-сервисов (Transient/Scoped/Singleton)** — см. таблицу в разделе ЛР3.

**Механизмы сохранения состояния (Query string/Hidden fields/TempData/Cookie/Session/Cache)** — см. таблицу в разделе ЛР8.

**Blazor Server vs Blazor WebAssembly** — см. раздел ЛР11 «Модели хостинга».

**REST vs GraphQL vs gRPC** — см. раздел «Дополнительные темы».

**Cookie vs JWT аутентификация** — см. раздел ЛР6.

---

# Вопросы к аттестации (полный список, 60 вопросов)

Все вопросы разобраны в соответствующих разделах конспекта выше (см. пометки **[Атт. №N]**). Ниже — полный список для финального повторения.

1. Принцип работы Веб-приложения. Основные отличия работы веб-приложения и настольного приложения. *(→ ЛР1)*
2. Протокол HTTP. Доступ к элементам Http-запроса из кода. *(→ ЛР1)*
3. Язык разметки HTML. Назначение, синтаксис. *(→ ЛР1)*
4. Стили, CSS. Применение стилей и CSS на странице. *(→ ЛР1)*
5. Формы HTML, синтаксис, назначение. Методы Get и Post. *(→ ЛР1)*
6. Структура каталогов приложения ASP.NET Core. Правила размещения файлов в проекте. Правила формирования имён файлов. *(→ ЛР1)*
7. Файлы `appsettings.json`, `_ViewStart.cshtml` и `_ViewImports.cshtml`. Назначение, использование. *(→ ЛР1, ЛР2)*
8. Класс Startup. Назначение. Методы Configure и ConfigureServices. *(→ ЛР1; в актуальных версиях — `Program.cs` с `WebApplicationBuilder`/`WebApplication`, см. раздел ЛР1)*
9. Шаблон проектирования MVC. Реализация MVC в ASP.NET Core. *(→ ЛР1)*
10. Представление ASP.NET. Язык Razor. *(→ ЛР2)*
11. Частичные представления. Методы для вызова частичных представлений. *(→ ЛР2)*
12. Страницы-макеты. Объявление секций на странице-макете. Использование страницы-макета в представлении. *(→ ЛР1, ЛР2)*
13. Компоненты представлений. Использование компонента в представлении. *(→ ЛР2)*
14. Отображение данных в представлении. Свойства ViewBag и ViewData. Строго типизированные представления. *(→ ЛР2)*
15. Вспомогательные (расширяющие) методы Html для генерации разметки. *(→ ЛР2)*
16. Вспомогательные (расширяющие) методы Html для передачи управления и генерации частичных представлений. *(→ ЛР2)*
17. Вспомогательные (расширяющие) методы Url. Метод Url.Content и свойство Server.MapPath. *(→ ЛР2, ЛР7)*
18. Тэг-хелперы. Встроенные тэг-хелперы для формы. *(→ ЛР2)*
19. Тэг-хелперы. Встроенные тэг-хелперы для передачи управления. *(→ ЛР2)*
20. Шаблон проектирования «Внедрение зависимостей» (Dependency Injection). IoC-контейнер. *(→ ЛР3)*
21. Реализация шаблона DI в ASP.NET Core. *(→ ЛР3)*
22. Контроллер ASP.NET. Назначение. Типы данных, возвращаемые методами контроллера. Вспомогательные методы класса Controller для возврата значения из метода контроллера. *(→ ЛР3)*
23. Передача данных методам контроллера. Механизм привязки данных. Передача данных от контроллера представлению. *(→ ЛР3)*
24. Фильтры, применяемые в контроллерах. *(→ ЛР10, дополнение)*
25. Аннотация и валидация данных. Проверка результата валидации. Вывод сообщений об ошибках. *(→ ЛР3)*
26. Механизм маршрутизации ASP.NET Core. Свойства маршрута. Регистрация маршрутов. *(→ ЛР8)*
27. Механизм маршрутизации ASP.NET Core. Маршрутизация с помощью атрибутов. *(→ ЛР8)*
28. Генерирование URL с помощью вспомогательных методов Html и Url. Класс LinkGenerator. Чтение значений сегментов маршрута. *(→ ЛР7, ЛР8)*
29. Передача файлов от клиента на сервер. Метод формы для передачи файла. Элемент формы для передачи файла. *(→ ЛР5)*
30. Передача файлов от сервера к клиенту. Класс FileResult. Вспомогательный метод File. *(→ ЛР5)*
31. Работа со статическими файлами. *(→ ЛР1, ЛР5)*
32. Технология Ajax. Назначение, принцип работы. Формирование Ajax-запросов с помощью JQuery. *(→ ЛР7)*
33. JSON. Создание JSON-запросов на веб-странице. Формирование ответов контроллера в формате JSON. *(→ ЛР7)*
34. Модульное тестирование в ASP.NET Core. Основные принципы модульного тестирования. Структура теста — шаблон AAA. *(→ ЛР9)*
35. Модульное тестирование в ASP.NET. Библиотека Moq. Методы класса Assert. Тестирование контроллера. *(→ ЛР9)*
36. Система аутентификации ASP.NET. Назначение. Принцип работы. *(→ ЛР6)*
37. Система аутентификации ASP.NET Identity. Класс IdentityUser. *(→ ЛР6)*
38. Основные методы классов UserManager, RoleManager и SignInManager. Проверка результата аутентификации и получение данных о текущем пользователе. *(→ ЛР6)*
39. Атрибуты ограничения доступа к контроллеру или методу контроллера. *(→ ЛР6)*
40. REST API контроллеры. Принцип обработки запросов. *(→ ЛР4)*
41. REST API контроллеры. Типы данных, возвращаемых методами API контроллера. *(→ ЛР4)*
42. Компоненты Middleware. Регистрация компонентов. *(→ ЛР1, ЛР10)*
43. Сессии в ASP.NET Core. *(→ ЛР8)*
44. Области (Areas). Назначение. Использование областей с контроллерами и страницами. *(→ ЛР5)*
45. Razor Pages. Обработка запросов при обращении к страницам. *(→ ЛР5)*
46. Razor Pages. Привязка данных между страницей и кодом модели. *(→ ЛР5)*
47. Razor Pages. Маршрутизация страниц. *(→ ЛР5)*
48. SignalR. Принцип работы. *(→ ЛР13)*
49. Фреймворк Blazor. Принцип работы Blazor SSR. *(→ ЛР11)*
50. Фреймворк Blazor. Структура проекта Blazor SSR. *(→ ЛР11)*
51. Фреймворк Blazor. Принцип работы Blazor WebAssembly. *(→ ЛР12)*
52. Фреймворк Blazor. Структура проекта Blazor WebAssembly. *(→ ЛР12)*
53. Фреймворк Blazor. Компоненты Razor. *(→ ЛР11, доп. материалы)*
54. Фреймворк Blazor. Маршрутизация страниц. *(→ ЛР11, ЛР12)*
55. Фреймворк Blazor. Привязка данных к параметрам маршрута. *(→ ЛР11, доп. материалы)*
56. Фреймворк Blazor. Привязка данных внутри компонента. *(→ доп. материалы)*
57. Фреймворк Blazor. Привязка данных от родительского компонента к дочернему. *(→ доп. материалы)*
58. Фреймворк Blazor. Привязка данных от дочернего компонента к родительскому. *(→ доп. материалы, ЛР12)*
59. Фреймворк Blazor. Обработка событий. *(→ доп. материалы)*
60. Фреймворк Blazor. Генерация ссылок в разметке и в коде. Отправка Http-запросов. *(→ ЛР12, доп. материалы)*


