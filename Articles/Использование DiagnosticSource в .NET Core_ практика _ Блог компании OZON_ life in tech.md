Использование DiagnosticSource в .NET Core: практика / Блог компании OZON: life in tech

В [предыдущей статье](https://habr.com/ru/company/ozontech/blog/435896/) я рассказал про механизм DiagnosticSource и на простом примере показал, как с его помощью можно перехватывать запросы в базу данных через классы `SqlConnection` и `SqlCommand` и измерять время их выполнения.

В настоящее время DiagnosticSource уже используется в AspNetCore, EntityFrameworkCore, HttpClient и SqlClient — каждый из них отправляет собственные события, которые могут быть перехвачены и обработаны.

В этой статье я хочу рассмотреть несколько примеров того, как можно на практике использовать DiagnosticSource в приложениях ASP.NET Core.

  

*   CorrelationID и пробрасывание заголовков между сервисами
*   Сбор метрик и трассировок
*   Логирование

Кроме того, в этой статье я решил собрать список событий, которые доступны для обработки и могут быть использованы в ваших приложениях, а также рассказать о некоторых подводных камнях, с которыми вы можете столкнуться, если решите использовать механизм DiagnosticSource в своём проекте.

## Существующие события

Прежде чем перейти к рассмотрению примеров, нам необходимо понять, какие компоненты отправляют события через DiagnosticSource, и как эти события называются. К сожалению, нигде в документации полный список событий не описан, и найти его можно разве что в исходниках на GitHub.

Поэтому самый простой способ понять, какие события существуют — создать класс, реализующий интерфейсы `IObserver<DiagnosticListener>` и `IObserver<KeyValuePair<string, object>>`, подписаться в нём на любые экземпляры `DiagnosticListener` и посмотреть, какие события будут перехватываться в приложении. Таким же способом можно определить параметры, передаваемые с каждым событием.

Чтобы упростить вам задачу, я уже собрал некоторые наиболее полезные события (это далеко не полный список) для четырёх компонентов:

  

**Microsoft.AspNetCore**

События компонента `Microsoft.AspNetCore` позволяют перехватывать события жизненного цикла обработки http запроса в ASP.NET Core.

  

*   Microsoft.AspNetCore.Hosting.HttpRequestIn.Start
*   Microsoft.AspNetCore.Hosting.HttpRequestIn.Stop

Эти события происходят самом начале и самом конце обработки http запроса.

  

*   Microsoft.AspNetCore.Diagnostics.UnhandledException

Происходят при необработанных исключениях. Это единственное место, где можно обработать исключения для данного компонента.

  

*   Microsoft.AspNetCore.Mvc.BeforeAction
*   Microsoft.AspNetCore.Mvc.AfterAction

Происходят до и после обработки http запроса в middleware, которые добавляются при использовании `UseMvc`. Фактически все следующие события происходят между этими двумя.

  

*   Microsoft.AspNetCore.Mvc.BeforeOnAuthorization
*   Microsoft.AspNetCore.Mvc.AfterOnAuthorization

Происходят до и после авторизации.

  

*   Microsoft.AspNetCore.Mvc.BeforeActionMethod
*   Microsoft.AspNetCore.Mvc.AfterActionMethod

Происходят до и после выполнения метода контроллера.

  

*   Microsoft.AspNetCore.Mvc.BeforeActionResult
*   Microsoft.AspNetCore.Mvc.AfterActionResult

Происходят до и после вызова `ExecuteResultAsync` у экземпляра `IActionResult`, который был возвращён из метода контроллера. Сюда, например, может входить сериализация результата в json.

  

*   Microsoft.AspNetCore.Mvc.BeforeHandlerMethod
*   Microsoft.AspNetCore.Mvc.AfterHandlerMethod

Используются в ASP.NET Pages. Происходят до и после выполнения метода модели страницы.

  

*   Microsoft.AspNetCore.Mvc.BeforeView
*   Microsoft.AspNetCore.Mvc.AfterView

Происходят до и после рендеринга представления.

  

**Microsoft.EntityFrameworkCore**

События компонента `Microsoft.EntityFrameworkCore` позволяют перехватывать события обращения к базе данных через EntityFrameworkCore.

  

*   Microsoft.EntityFrameworkCore.Infrastructure.ContextInitialized
*   Microsoft.EntityFrameworkCore.Infrastructure.ContextDisposed

Происходят до и после использования экземпляра `DbContext`

  

*   Microsoft.EntityFrameworkCore.Database.Connection.ConnectionOpening
*   Microsoft.EntityFrameworkCore.Database.Connection.ConnectionOpened
*   Microsoft.EntityFrameworkCore.Database.Connection.ConnectionError

Происходят до и после открытия подключения к базе данных. Если подключение было успешно открыто, происходит событие `ConnectionOpened`. Если при открытии подключения возникла ошибка, происходит событие `ConnectionError`.

  

*   Microsoft.EntityFrameworkCore.Database.Connection.ConnectionClosing
*   Microsoft.EntityFrameworkCore.Database.Connection.ConnectionClosed
*   Microsoft.EntityFrameworkCore.Database.Connection.ConnectionError

Аналогично происходят до и после закрытия подключения к базе данных.

  

*   Microsoft.EntityFrameworkCore.Database.Command.CommandExecuting
*   Microsoft.EntityFrameworkCore.Database.Command.CommandExecuted
*   Microsoft.EntityFrameworkCore.Database.Command.CommandError

Аналогично происходят до и после выполнения запроса к базе данных.

  

*   Microsoft.EntityFrameworkCore.Database.Command.DataReaderDisposing

Происходит после завершения чтения из экземпляра `DbDataReader`.

  

**SqlClientDiagnosticListener**

События компонента `SqlClientDiagnosticListener` позволяют перехватывать события обращения к базе данных SQL Server через соответствующий ADO.NET провайдер.

  

*   System.Data.SqlClient.WriteConnectionOpenBefore
*   System.Data.SqlClient.WriteConnectionOpenAfter
*   System.Data.SqlClient.WriteConnectionOpenError

Происходят до и после открытия подключения к базе данных. Если подключение было успешно открыто, происходит событие `WriteConnectionOpenAfter`. Если при открытии подключения возникла ошибка, происходит событие `WriteConnectionOpenError`.

  

*   System.Data.SqlClient.WriteConnectionCloseBefore
*   System.Data.SqlClient.WriteConnectionCloseAfter
*   System.Data.SqlClient.WriteConnectionCloseError

Аналогично происходят до и после закрытия подключения к базе данных.

  

*   System.Data.SqlClient.WriteCommandBefore
*   System.Data.SqlClient.WriteCommandAfter
*   System.Data.SqlClient.WriteCommandError

Аналогично происходят до и после выполнения запроса к базе данных.

  

**HttpHandlerDiagnosticListener**

События компонента `HttpHandlerDiagnosticListener` позволяют перехватывать исходящие http запросы, например, при использовании класса `HttpClient`.

  

*   System.Net.Http.HttpRequestOut.Start
*   System.Net.Http.HttpRequestOut.Stop

Происходят до и после исходящего http запроса.

  

*   System.Net.Http.Exception

Происходит, если при исходящем http запроса возникла ошибка.

Кстати, существует даже [DiagnosticSource User's Guide](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md), в котором описаны рекомендации и соглашения по именованию событий для DiagnosticSource.

Как можно легко догадаться, Microsoft не следует этим рекомендациям и делает всё наоборот =) (Ладно, я преувеличиваю. Просто DiagnosticSource начал использоваться в компонентах .NET Core раньше, чем появился DiagnosticSource User's Guide)

  

## Общий код

Предполагается, что все примеры, которые я буду рассматривать ниже, будут использоваться в приложении ASP.NET Core (хотя это не обязательно), и будут использовать базовый класс `DiagnosticObserverBase` для подписки на события от DiagnosticSource и их обработки.

Этот класс основан на классе `ExampleDiagnosticObserver` из моей [предыдущей статьи](https://habr.com/ru/company/ozontech/blog/435896/), где можно найти описание его работы. Для подписки и обработки событий этот класс будет использовать метод `SubscribeWithAdapter` из NuGet пакета [Microsoft.Extensions.DiagnosticAdapter](https://www.nuget.org/packages/Microsoft.Extensions.DiagnosticAdapter/).

  

    public abstract class DiagnosticObserverBase : IObserver<DiagnosticListener>
    {
        private readonly List<IDisposable> _subscriptions = new List<IDisposable>();
    
        protected abstract bool IsMatch(string name);
    
        void IObserver<DiagnosticListener>.OnNext(DiagnosticListener diagnosticListener)
        {
            if (IsMatch(diagnosticListener.Name))
            {
                var subscription = diagnosticListener.SubscribeWithAdapter(this);
                _subscriptions.Add(subscription);
            }
        }
    
        void IObserver<DiagnosticListener>.OnError(Exception error)
        { }
    
        void IObserver<DiagnosticListener>.OnCompleted()
        {
            _subscriptions.ForEach(x => x.Dispose());
            _subscriptions.Clear();
        }
    }

Чтобы подписаться на события от определённых компонентов, необходимо создать новый класс, унаследовать его от `DiagnosticObserverBase`, переопределить метод `IsMatch`, чтобы он возвращал `true` для тех компонентов, на которые мы хотим подписаться, добавить методы для обработки событий и пометить их атрибутами `DiagnosticNameAttribute`, где указать название обрабатываемого события. Например:

  

    public sealed class SomeDiagnosticObserver : DiagnosticObserverBase
    {
        protected override bool IsMatch(string name)
        {
            return name == "SomeComponent";
        }
    
        [DiagnosticName("SomeEvent")]
        public void OnSomeEvent()
        {
            
        }
    }

Для того, чтобы зарегистрировать в DI контейнере обработчики, основанные на классе `DiagnosticObserverBase`, мы воспользуемся методом-расширением `AddDiagnosticObserver`, которое будет использовать в методе `ConfigureServices` в файле Startup.cs:

  

    public static class DiagnosticServiceCollectionExtensions
    {
        public static void AddDiagnosticObserver<TDiagnosticObserver>(
            this IServiceCollection services)
            where TDiagnosticObserver : DiagnosticObserverBase
        {
            services.TryAddEnumerable(ServiceDescriptor
                .Transient<DiagnosticObserverBase, TDiagnosticObserver>());
        }
    }

А для того, чтобы подписаться на события от DiagnosticSource, добавим в метод `Configure` следующие строки:

  

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        var diagnosticObservers = app
            .ApplicationServices.GetServices<DiagnosticObserverBase>();
        foreach (var diagnosticObserver in diagnosticObservers)
        {
            DiagnosticListener.AllListeners.Subscribe(diagnosticObserver);
        }
    
        
    
        app.UseMvc();
    }

Возможно, это не лучший способ регистрации и на практике мы обычно используем для таких целей интерфейс `IHostedService`, но наших примеров этого вполне будет достаточно.

  

## Некоторые подводные камни

Если вы решите использовать DiagnosticSource в своих проектах, то можете столкнуться с некоторыми неочевидными моментами, на которых я хотел бы немного остановиться.

  

### Иногда могут потребоваться фиктивные обработчики для несуществующих событий

Обычно, если какой-то компонент отправляет события о свой работе, код отправки события выглядит следующим образом:

  

    if (_diagnosticSource.IsEnabled("SomeEvent"))
        _diagnosticSource.Write("SomeEvent", new {  });

Это позволяет не создавать объект с параметрами, если событие никто не собирается обрабатывать, и немного сэкономить на сборке мусора.

Однако, в некоторых случаях существуют парные события с суффиксами `.Start` и `.Stop`, оба из которых должны либо срабатывать, либо нет. Код отправки таких событий может выглядеть примерно следующим образом:

  

     
    var someEventIsEnabled = _diagnosticSource.IsEnabled("SomeEvent");
    
    if (someEventIsEnabled && _diagnosticSource.IsEnabled("SomeEvent.Start"))
        _diagnosticSource.Write("SomeEvent.Start", new {  });
    
    
    
    if (someEventIsEnabled && _diagnosticSource.IsEnabled("SomeEvent.Stop"))
        _diagnosticSource.Write("SomeEvent.Stop", new {  });

Поэтому для подписки на события `SomeEvent.Start` и `SomeEvent.Stop` необходимо также добавить фиктивный обработчик для события `SomeEvent`, который никогда не будет вызываться, но будет проверяться его наличие.

  

### Некоторые события парные, а некоторые тройные

Некоторые события являются парными, например `System.Net.Http.HttpRequestOut.Start` и `System.Net.Http.HttpRequestOut.Stop`. Это означает, что событие с суффиксом `.Start` будет вызвано до начала какой-то операции, а событие с суффиксом `.Stop` — в конце. При этом последнее событие будет гарантированно вызвано (если есть соответствующие обработчики) вне зависимости от того, закончилась ли операция ошибкой или нет.

Однако, некоторые события являются тройными, например `System.Data.SqlClient.WriteCommandBefore`, `System.Data.SqlClient.WriteCommandAfter` и `System.Data.SqlClient.WriteCommandError`, где последнее событие зависит от результата опреации. В этом случае, если операция была завершена успешно, будет вызвано только событие `System.Data.SqlClient.WriteCommandAfter`, а если во время операции возникла ошибка — только событие `System.Data.SqlClient.WriteCommandError`.

Это надо учитывать, если вы, например, используете события для измерения времени операций. Например, если вы запускаете секундомер в начале операции, то его остановку нужно делать в двух местах, чтобы не потрерять данные.

  

## Примеры использования DiagnosticSource

Теперь у нас всё готово для того, чтобы рассмотреть, как механизм DiagnosticSource можно применить на практике в реальных приложениях.

  

### CorrelationID и пробрасывание заголовков между сервисами

В мире микросервисов часто можно встретить термин CorrelationID. Это некоторый идентификатор, который генерируется при каждом обращении к любому сервису и дальше передаётся от сервиса к сервису через http заголовки. Этот идентификатор обычно пишется в логи, позволяя тем самым связать сообщения от нескольких сервисов, полученных в рамках выполнения одной транзакции.

Для ASP.NET Core существует NuGet пакет [CorrelationId](https://github.com/stevejgordon/CorrelationId), однако он требует того, чтобы разработчики вручную добавляли соответствующий заголовок во все исходящие запросы, поэтому не очень удобен в использовании.

Реализуем CorrelationId через DiagnosticSource. Для начала, добавим класс `CorrelationId` который будет отвечать за хранение нашего идентификатора:

  

    public static class CorrelationId
    {
        private static readonly AsyncLocal<Guid?> _current = new AsyncLocal<Guid?>();
    
        public static Guid Current
        {
            get
            {
                var value = _current.Value;
                if (value == null)
                    throw new InvalidOperationException("CorrelationId isn't assigned.");
    
                return value.Value;
            }
    
            set { _current.Value = value; }
        }
    }

Этот класс использует экземпляр типа [AsyncLocal&lt;T&gt;](https://docs.microsoft.com/ru-ru/dotnet/api/system.threading.asynclocal-1?view=netcore-2.2) для хранения текущего значения CorrelationId, которое будет уникально для каждого запроса, но при этом будет корректно передаваться от одного потока из ThreadPool к другому при работе с асинхронным кодом.

Следующим шагом добавим обработчик событий от DiagnosticSource, который будет перехватывать входящие и исходящие http запросы. Во входящих запросах мы будем проверять наличие заголовка `X-Correlation-ID` и, если его нет будем генерировать новый идентификатор через `Guid.NewGuid()`. В исходящих запросах мы будем просто добавлять заголовок, используя `CorrelationId.Current`.

  

    public sealed class CorrelationIdHandler : DiagnosticObserverBase
    {
        protected override bool IsMatch(string name)
        {
            return name == "Microsoft.AspNetCore"
                || name == "HttpHandlerDiagnosticListener";
        }
    
        
    
        [DiagnosticName("Microsoft.AspNetCore.Hosting.HttpRequestIn")]
        public void OnHttpRequestIn()
        { }
    
        [DiagnosticName("Microsoft.AspNetCore.Hosting.HttpRequestIn.Start")]
        public void OnHttpRequestInStart(HttpContext httpContext)
        {
            
            var headers = httpContext.Request.Headers;
            if (headers.TryGetValue("X-Correlation-ID", out var header))
            {
                if (Guid.TryParse(header, out var correlationId))
                {
                    CorrelationId.Current = correlationId;
                    return;
                }
            }
    
            
            CorrelationId.Current = Guid.NewGuid();
        }
    
        
    
        [DiagnosticName("System.Net.Http.HttpRequestOut")]
        public void OnHttpRequestOut()
        { }
    
        [DiagnosticName("System.Net.Http.HttpRequestOut.Start")]
        public void OnHttpRequestOutStart(HttpRequestMessage request)
        {
            
            var correlationId = CorrelationId.Current.ToString();
            request.Headers.Add("X-Correlation-ID", correlationId);
        }
    }

В этом классе в методе `IsMatch` мы сообщаем, что хотим обрабатывать события от компонентов `Microsoft.AspNetCore` (отвечает за входящие http запросы) и `HttpHandlerDiagnosticListener` (отвечает за исходящие http запросы). Непосредственная обработка заголовков происходит в методах `OnHttpRequestInStart` и `OnHttpRequestOutStart`.

Кроме этого нам пришлось добавить два фиктивных метода `OnHttpRequestIn` и `OnHttpRequestOut`. Они не будут вызываться при обработке, но используются для определения того, нужно ли вызывать пару обработчиков `Start` и `Stop`. Без них эти события не будут вызваны.

Осталось только зарегистрировать наш обработчик в файле Startup.cs:

  

    services.AddDiagnosticObserver<CorrelationIdHandler>();

На практике бывает также полезно пробрасывать не один, а несколько заголовков с определённым префиксом (например "X-Api-"), реализуя тем самым так называемый Context Propagation. Этот механизм позволяет задать значение с определённым ключом в одном сервисе и прочитать в другом, не передавая это значение явно через тело запроса. Подобный механизм можно легко реализовать на базе описанного выше класса `CorrelationIdHandler`.

  

### Сбор метрик и трассировок

Метрики и трассировки являются важной частью любого приложения. Метрики позволяют настраивать мониторинг и дашборды для приложений, а трассировки — находить в них узкие места.

Мы в OZON.ru для сбора метрик используем [Prometheus](https://prometheus.io/) и, для ASP.NET Core сервисов, NuGet пакет [Prometheus.Client.AspNetCore](https://github.com/PrometheusClientNet/Prometheus.Client.AspNetCore).

Для сбора трассировок мы используем [OpenTracing](https://opentracing.io/) и [Jaeger](https://www.jaegertracing.io/). (При желании вы можете посмотреть [мой доклад](https://www.youtube.com/watch?v=-siZv8TZoMg) "Использование OpenTracing в .NET" с DotNetMsk Meetut #30)

Однако многие разработчики не очень стремятся покрывать свои приложения метриками и трассировками, поскольку зачастую это требует написания дополнительного однообразного кода и не очень согласуется с "бизнес-задачами".

К счастью, большинство компонентов, отдающих события через DiagnosticSource, отдают парные события, первое из которых обозначает начало определённой операции, а второе — её завершение. Это позволяет, например, сначала запустить секундомер, а после остановить его и отдать определённую метрику.

Например, если мы заходим собирать метрику по времени выполнения всех действий контроллеров, можем использовать следующий класс:

  

    public sealed class AspNetCoreMetricsHandler : DiagnosticObserverBase
    {
        private readonly Histogram requestDurationSeconds;
    
        public MetricsHandler(MetricFactory metricFactory)
        {
            
            
            requestDurationSeconds = metricFactory.CreateHistogram(
                "request_duration_seconds", "",
                labelNames: new[] {"action_name"});
        }
    
        protected override bool IsMatch(string name)
        {
            return name == "Microsoft.AspNetCore";
        }
    
        [DiagnosticName("Microsoft.AspNetCore.Hosting.HttpRequestIn")]
        public void OnHttpRequestIn()
        { }
    
        [DiagnosticName("Microsoft.AspNetCore.Hosting.HttpRequestIn.Start")]
        public void OnHttpRequestInStart(HttpContext httpContext)
        {
            
            httpContext.Items["Stopwatch"] = Stopwatch.StartNew();
        }
    
        [DiagnosticName("Microsoft.AspNetCore.Mvc.BeforeAction")]
        public void OnBeforeAction(HttpContext httpContext, ActionDescriptor actionDescriptor)
        {
            
            
            httpContext.Items["ActionName"] = actionDescriptor.DisplayName;
        }
    
        [DiagnosticName("Microsoft.AspNetCore.Hosting.HttpRequestIn.Stop")]
        public void OnHttpRequestInStop(HttpContext httpContext)
        {
            
            
    
            if (!httpContext.Items.TryGetValue("Stopwatch", out object stopwatch))
                return;
    
            if (!httpContext.Items.TryGetValue("ActionName", out object actionName))
                actionName = "Unknown";
    
            var duration = ((Stopwatch) stopwatch).Elapsed.TotalSeconds;
    
            requestDurationSeconds
                .WithLabels(actionName.ToString())
                .Observe(duration);
        }
    }

Здесь мы в конструкторе объявляем метрику типа "Гистограмма" из NuGet пакета [Prometheus.Client](https://github.com/PrometheusClientNet/Prometheus.Client). Этой метрике мы добавляем метку "action_name", которая позволит отличать метрики, собранные в разных действиях контроллеров.

В начале обработки событий (метод `OnHttpRequestInStart`) мы запускаем секундомер, чтобы измерить время выполнения запроса. Также мы запоминаем название обрабатываемого действия (метод `OnBeforeAction`). И наконец после обработки запроса (метод `OnHttpRequestInStop`) снова получаем все данные из коллекции `httpContext.Items` и записываем их в метрику.

Осталось только зарегистрировать наш обработчик и экземпляр `MetricFactory` в файле Startup.cs:

  

    services.AddSingleton(Prometheus.Client.Metrics.DefaultFactory);
    services.AddDiagnosticObserver<AspNetCoreMetricsHandler>();

Похожая техника может использоваться и при сборе трассировок с использованием NuGet пакета [OpenTracing](https://github.com/opentracing/opentracing-csharp).

  

### Логирование

Ещё одно очень полезное применение DiagnosticSource — логирование исключений. Но может возникнуть вопрос: "Зачем это нужно?". Ведь можно просто обернуть свой код в блок try-catch или вообще настроить один глобальный обработчик для всех необработанных исключений.

Дело в том, что обработка исключений через события DiagnosticSource происходит на очень раннем этапе, когда ещё доступны различные объекты, которые могут помочь нам понять причину исключения. (Стоит отметить, что DiagnosticSource позволяет только обработать исключение, но не позволит предотвратить его дальнейшее распространение)

Предположим, что мы хотим централизовано обрабатывать все исключения при обращении к базе данных, при этом записывая в лог текст запроса и его параметры. Используя DiagnosticSource мы можем сделать это следующим образом:

  

    public sealed class SqlClientLoggingHandler : DiagnosticObserverBase
    {
        private readonly ILogger<SqlClientLoggingHandler> _logger;
    
        public SqlClientLoggingHandler(ILogger<SqlClientLoggingHandler> logger)
        {
            _logger = logger;
        }
    
        protected override bool IsMatch(string name)
        {
            return name == "SqlClientDiagnosticListener";
        }
    
        [DiagnosticName("System.Data.SqlClient.WriteCommandError")]
        public void OnCommandError(DbCommand command, Exception exception)
        {
            var sb = new StringBuilder();
            sb.AppendLine("Command: " + command.CommandText);
    
            if (command.Parameters.Count > 0)
            {
                sb.AppendLine("Parameters: ");
    
                foreach (DbParameter parameter in command.Parameters)
                {
                    sb.AppendLine($"\t{parameter.ParameterName}: {parameter.Value}");
                }
            }
    
            _logger.LogError(exception, sb.ToString());
        }
    }

В методе `IsMatch` мы указываем, что хотим обрабатывать события от компонента `SqlClientDiagnosticListener`, а в методе `OnCommandError` формируем сообщения с телом запроса и параметрами и записываем его в лог.

Зарегистрировать наш обработчик исключений в файле Startup.cs можно следующим образом:

  

    services.AddDiagnosticObserver<SqlClientLoggingHandler>();

  

## Заключение

В этой статье я рассмотрел практические примеры использования DiagnosticSource. Возможно, достаточно поверхностно, но этого достаточно для общего понимания того, что представляет собой DiagnosticSource, какие возможности он предоставляет и как его его можно использовать в приложениях на ASP.NET Core.

Мы в OZON.ru активно используем этот механизм в наших сервисах для решения похожих задач. Это позволяет, буквально подключив один NuGet пакет и написав одну строчку кода, получить автоматический сбор метрик с приложений, мониторинг и многое другое.

Я надеюсь, что эта и [предыдущая](https://habr.com/ru/company/ozontech/blog/435896/) мои статьи будут полезными, если вы заходите использовать DiagnosticSource в своих проектах.