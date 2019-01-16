Использование DiagnosticSource в .NET Core: теория / Блог компании OZON: life in tech

DiagnosticSource — это простой, но весьма полезный набор API (доступен в NuGet пакете [System.Diagnostics.DiagnosticSource](https://www.nuget.org/packages/System.Diagnostics.DiagnosticSource/)), который, с одной стороны, позволяет различным библиотекам отправлять именованные события о своей работе, а с другой — позволяет приложениям подписываться на эти события и обрабатывать их.

Каждое такое событие содержит дополнительную информацию (payload), а поскольку обработка событий происходит в том же процессе, что и отправка, эта информация может содержать практически любые объекты без необходимости сериализации/десереализации.

DiagnosticSource уже используется в AspNetCore, EntityFrameworkCore, HttpClient и SqlClient, что фактически даёт разработчикам возможность перехватывать входящие/исходящие http запросы, запросы к базам данных, получать доступ к таким объектам, как `HttpContext`, `DbConnection`, `DbCommand`, `HttpRequestMessage` и многим другим и даже изменять эти объекты при необходимости.

Я решил разделить свой рассказ про DiagnosticSource на две статьи. В этой статье мы на простом примере разберем принцип работы механизма, а в следующей я расскажу о существующих в .NET событиях, которые можно обрабатывать с его помощью и покажу несколько примеров его использования в OZON.ru.

## Пример

Чтобы лучше понять, как работает DiagnosticSource, рассмотрим небольшой пример c перехватом запросов к базе данных. Представим, что у нас есть простое консольное приложение, которое делает запрос в базу данных и выводит полученный результат в консоль.

  

    public static class Program
    {
        public const string ConnectionString =
            @"Data Source=localhost;Initial Catalog=master;User ID=sa;Password=Password12!;";
    
        public static async Task Main()
        {
            var answer = await GetAnswerAsync();
            Console.WriteLine(answer);
        }
    
        public static async Task<int> GetAnswerAsync()
        {
            using (var connection = new SqlConnection(ConnectionString))
            {
                
                return await connection.QuerySingleAsync<int>("SELECT 42;");
            }
        }
    }

Для простоты я поднял SQL Server в docker контейнере.

  

**docker run**

    docker run --rm --detach --name mssql-server \
        --publish 1433:1433 \
        --env ACCEPT_EULA=Y \
        --env SA_PASSWORD=Password12! \
        mcr.microsoft.com/mssql/server:2017-latest

Теперь представим, что у нас есть задача: нужно измерить время выполнения всех запросов в базу данных с помощью `Stopwatch` и вывести в консоль пары "Запрос" — "Время выполнения".

Конечно, можно просто обернуть вызов `QuerySingleAsync` кодом, который создаст и запустит экземпляр `Stopwarch`, остановит его после выполнения запроса и выведет результат, но тут возникает сразу несколько сложностей:

  

*   Что если в приложении не один запрос, а значительно больше?
*   Что если код, который выполняет запрос уже скомпилирован, подключен к приложению в виде NuGet пакета, и у нас нет возможности изменить его?
*   Что если запрос в базу данных делается не через Dapper, а например через EntityFramework, и у нас нет доступа ни к объекту `DbCommand`, ни к сгенерированному тексту запроса, который реально будет выполняться?

Попробуем решить эту задачу с использованием DiagnosticSource.

  

### Использование NuGet пакета System.Diagnostics.DiagnosticSource

Первое, что нужно сделать после подключения NuGet пакета [System.Diagnostics.DiagnosticSource](https://www.nuget.org/packages/System.Diagnostics.DiagnosticSource/) — создать класс, который будет обрабатывать интересующие нас события:

  

    public sealed class ExampleDiagnosticObserver
    { }

Для того, чтобы начать обрабатывать события, нужно создать экземпляр данного класса и зарегистрировать его наблюдателем в статическом объекте `DiagnosticListener.AllListeners` (находится в пространстве имён `System.Diagnostics`). Сделаем это в самом начале функции `Main`:

  

    public static async Task Main()
    {
        var observer = new ExampleDiagnosticObserver();
        IDisposable subscription = DiagnosticListener.AllListeners.Subscribe(observer);
    
        var answer = await GetAnswerAsync();
        Console.WriteLine(answer);
    }

При этом компилятор справедливо скажет нам, что класс `ExampleDiagnosticObserver` должен реализовывать интерфейс `IObserver<DiagnosticListener>`. Давайте реализуем его:

  

    public sealed class ExampleDiagnosticObserver : IObserver<DiagnosticListener>
    {
        void IObserver<DiagnosticListener>.OnNext(DiagnosticListener diagnosticListener)
        {
            Console.WriteLine(diagnosticListener.Name);
        }
    
        void IObserver<DiagnosticListener>.OnError(Exception error)
        { }
    
        void IObserver<DiagnosticListener>.OnCompleted()
        { }
    }

Если мы сейчас запустим этот код, то увидим, что в консоль будет выведено следующее:

  

    SqlClientDiagnosticListener
    SqlClientDiagnosticListener
    42

Это означает, что где-то в .NET зарегестрировано два объекта типа `DiagnosticListener` с названием `"SqlClientDiagnosticListener"` которые сработали при выполнении этого кода.

  

Метод `IObserver<DiagnosticListener>.OnNext` будет вызываться **один раз при первом использовании** для каждого экземпляра `DiagnosticListener` который создан в приложении (обычно они создаются как статические свойства). Сейчас мы просто вывели в консоль название экземпляров `DiagnosticListener`, но на практике в этом методе необходимо проверить это название и, если нам интересно обрабатывать события от этого экземпляра, подписаться на него с использованием метода `Subscribe`.

Также хочу отметить, что при вызове `DiagnosticListener.AllListeners.Subscribe` мы в качестве результата получим объект `subscription`, который реализует интерфейс `IDisposable`. Вызов метода `Dispose` у этого объекта приведёт к отписке, которую надо реализовать в методе `IObserver<DiagnosticListener>.OnCompleted`.

Давайте реализуем`IObserver<DiagnosticListener>` ещё раз:

  

    public sealed class ExampleDiagnosticObserver : IObserver<DiagnosticListener>
    {
        private readonly List<IDisposable> _subscriptions = new List<IDisposable>();
    
        void IObserver<DiagnosticListener>.OnNext(DiagnosticListener diagnosticListener)
        {
            if (diagnosticListener.Name == "SqlClientDiagnosticListener")
            {
                var subscription = diagnosticListener.Subscribe(this);
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

Теперь компилятор скажет нам, что наш класс `ExampleDiagnosticObserver` должен так же реализовывать интерфейс `IObserver<KeyValuePair<string, object>>`. Здесь нам необходимо реализовать метод `IObserver<KeyValuePair<string, object>>.OnNext`, который в качесте параметра принимает `KeyValuePair<string, object>`, где ключём является название события, а значением анонимный объект (обычно) с произвольными параметрами, которые мы можем использовать по своему усмотрению. Давайте добавим реализацию этого интерфекса:

  

    public sealed class ExampleDiagnosticObserver :
        IObserver<DiagnosticListener>,
        IObserver<KeyValuePair<string, object>>
    {
        
        
    
        void IObserver<KeyValuePair<string, object>>.OnNext(KeyValuePair<string, object> pair)
        {
            Write(pair.Key, pair.Value);
        }
    
        void IObserver<KeyValuePair<string, object>>.OnError(Exception error)
        { }
    
        void IObserver<KeyValuePair<string, object>>.OnCompleted()
        { }
    
        private void Write(string name, object value)
        {
            Console.WriteLine(name);
            Console.WriteLine(value);
            Console.WriteLine();
        }
    }

Если мы теперь запустим полученный код, то в консоль будет выведено примерно следующее:

  

    System.Data.SqlClient.WriteConnectionOpenBefore
    { OperationId = 3da1b5d4-9ce1-4f28-b1ff-6a5bfc9d64b8, Operation = OpenAsync, Connection = System.Data.SqlClient.SqlConnection, Timestamp = 26978341062 }
    
    System.Data.SqlClient.WriteConnectionOpenAfter
    { OperationId = 3da1b5d4-9ce1-4f28-b1ff-6a5bfc9d64b8, Operation = OpenAsync, ConnectionId = 84bd0095-9831-456b-8ebc-cb9dc2017368, Connection = System.Data.SqlClient.SqlConnection,
    Statistics = System.Data.SqlClient.SqlStatistics+StatisticsDictionary, Timestamp = 26978631500 }
    
    System.Data.SqlClient.WriteCommandBefore
    { OperationId = 5c6d300c-bc49-4f80-9211-693fa1e2497c, Operation = ExecuteReaderAsync, ConnectionId = 84bd0095-9831-456b-8ebc-cb9dc2017368, Command = System.Data.SqlClient.SqlComman
    d }
    
    System.Data.SqlClient.WriteCommandAfter
    { OperationId = 5c6d300c-bc49-4f80-9211-693fa1e2497c, Operation = ExecuteReaderAsync, ConnectionId = 84bd0095-9831-456b-8ebc-cb9dc2017368, Command = System.Data.SqlClient.SqlComman
    d, Statistics = System.Data.SqlClient.SqlStatistics+StatisticsDictionary, Timestamp = 26978709490 }
    
    System.Data.SqlClient.WriteConnectionCloseBefore
    { OperationId = 3f6bfd8f-e5f6-48b7-82c7-41aeab881142, Operation = Close, ConnectionId = 84bd0095-9831-456b-8ebc-cb9dc2017368, Connection = System.Data.SqlClient.SqlConnection, Stat
    istics = System.Data.SqlClient.SqlStatistics+StatisticsDictionary, Timestamp = 26978760625 }
    
    System.Data.SqlClient.WriteConnectionCloseAfter
    { OperationId = 3f6bfd8f-e5f6-48b7-82c7-41aeab881142, Operation = Close, ConnectionId = 84bd0095-9831-456b-8ebc-cb9dc2017368, Connection = System.Data.SqlClient.SqlConnection, Stat
    istics = System.Data.SqlClient.SqlStatistics+StatisticsDictionary, Timestamp = 26978772888 }
    
    42

Всего мы увидим шесть событий. Два из них выполняются до и после открытия подключения к базе данных, два — до и после выполнения команды и ещё два — до и после закрытия подключения к базе данных.

Каждое событие содержит набор параметров, таких как `OperationId`, `ConnectionId`, `Connection`, `Command`, которые обычно передаются как свойства анонимного объекта. Получить типизированные значения этих свойств можно, например, с помощью reflection. (На практике использование reflection может быть не очень желательно. Мы для получения параметров событий используем DynamicMethod.)

Теперь у нас всё готово, чтобы решить изначальную задачу — измерить время выполнения всех запросов в базу данных и вывести его в консоль вместе с исходным запросом.

Изменим реализацию метода `Write` следующим образом:

  

    public sealed class ExampleDiagnosticObserver :
        IObserver<DiagnosticListener>,
        IObserver<KeyValuePair<string, object>>
    {
        
        
    
        
        
    
        private readonly AsyncLocal<Stopwatch> _stopwatch = new AsyncLocal<Stopwatch>();
    
        private void Write(string name, object value)
        {
            switch (name)
            {
                case "System.Data.SqlClient.WriteCommandBefore":
                {
                    
                    _stopwatch.Value = Stopwatch.StartNew();
                    break;
                }
    
                case "System.Data.SqlClient.WriteCommandAfter":
                {
                    
                    var stopwatch = _stopwatch.Value;
                    stopwatch.Stop();
    
                    var command = GetProperty<SqlCommand>(value, "Command");
    
                    Console.WriteLine($"CommandText: {command.CommandText}");
                    Console.WriteLine($"Elapsed: {stopwatch.Elapsed}");
                    Console.WriteLine();
    
                    break;
                }
            }
        }
    
        private static T GetProperty<T>(object value, string name)
        {
            return (T) value.GetType()
                .GetProperty(name)
                .GetValue(value);
        }
    }

Здесь мы перехватываем события начала и окончания запроса в базу данных. Перед выполнением запроса мы создаём и запускаем секундомер, сохраняя его в переменной типа `AsyncLocal<Stopwatch>`, чтобы позже получить его обратно. После выполнения запроса мы получаем ранее запущенный секундомер, останавливаем его, получаем выполненную команду из параметра `value` через reflection и выводим полученный результат в консоль.

Если мы теперь запустим полученный код, то в консоль будет выведено примерно следующее:

  

    CommandText: SELECT 42;
    Elapsed: 00:00:00.0341357
    
    42

Казалось бы, что мы уже решили нашу задачу, но осталась одна маленькая деталь. Дело в том, что когда мы подписываемся на события `DiagnosticListener` мы начинаем получать от него даже те события, которые нам не интересны, а поскольку при отправке каждого события создаётся анонимный объект с параметрами, это может создавать лишнюю нагрузку на GC.

Чтобы избежать этой ситуации и сообщить, какие именно события от `DiagnosticListener` мы собираемся обрабатывать, мы можем при подписке указать специальный делегат типа `Predicate<string>` который в качестве параметра принимает название события и возвращает `true`, если это событие должно быть обработано.

Немного изменим метод `IObserver<DiagnosticListener>.OnNext` в нашем классе:

  

    void IObserver<DiagnosticListener>.OnNext(DiagnosticListener diagnosticListener)
    {
        if (diagnosticListener.Name == "SqlClientDiagnosticListener")
        {
            var subscription = diagnosticListener.Subscribe(this, IsEnabled);
            _subscriptions.Add(subscription);
        }
    }
    
    private bool IsEnabled(string name)
    {
        return name == "System.Data.SqlClient.WriteCommandBefore"
            || name == "System.Data.SqlClient.WriteCommandAfter";
    }

Теперь наш метод `Write` будет вызываться только для событий `"System.Data.SqlClient.WriteCommandBefore"` и `"System.Data.SqlClient.WriteCommandAfter"`.

  

### Использование NuGet пакета Microsoft.Extensions.DiagnosticAdapter

Поскольку параметры события, которые мы получаем от `DiagnosticListener`, обычно передаются в виде анонимного объекта, извлекать их через reflection может быть слишком затратно. К счастью, существует NuGet пакет [Microsoft.Extensions.DiagnosticAdapter](https://www.nuget.org/packages/Microsoft.Extensions.DiagnosticAdapter/), который может сделать это за нас, используя runtime кодогенерацию из пространства имён `System.Reflection.Emit`.

Для того, чтобы использовать этот пакет при подписке на события от экземпляра `DiagnosticListener` вместо метода `Subscribe` необходимо использовать метод-расширение `SubscribeWithAdapter`. Реализовывать интерфейс `IObserver<KeyValuePair<string, object>>` в этом случае больше не требуется. Вместо этого для каждого события, которое мы хотим обрабатывать нам необходимо оъявить отдельный метод, пометив его атрибутом `DiagnosticNameAttribute` (из пространства имён `Microsoft.Extensions.DiagnosticAdapter`). Параметрами этих методов будут параметры обрабатываемого события.

Если мы перепишем наш класс `ExampleDiagnosticObserver` с использованием данного NuGet пакета, то получим следующий код:

  

    public sealed class ExampleDiagnosticObserver : IObserver<DiagnosticListener>
    {
        private readonly List<IDisposable> _subscriptions = new List<IDisposable>();
    
        void IObserver<DiagnosticListener>.OnNext(DiagnosticListener diagnosticListener)
        {
            if (diagnosticListener.Name == "SqlClientDiagnosticListener")
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
    
        private readonly AsyncLocal<Stopwatch> _stopwatch = new AsyncLocal<Stopwatch>();
    
        [DiagnosticName("System.Data.SqlClient.WriteCommandBefore")]
        public void OnCommandBefore()
        {
            _stopwatch.Value = Stopwatch.StartNew();
        }
    
        [DiagnosticName("System.Data.SqlClient.WriteCommandAfter")]
        public void OnCommandAfter(DbCommand command)
        {
            var stopwatch = _stopwatch.Value;
            stopwatch.Stop();
    
            Console.WriteLine($"CommandText: {command.CommandText}");
            Console.WriteLine($"Elapsed: {stopwatch.Elapsed}");
            Console.WriteLine();
        }
    }

Таким образом мы теперь можем измерить время выполнения всех запросов в базу данных из нашего приложения, практически не изменяя при этом код самого приложения.

  

## Создание собственных экземпляров DiagnosticListener

При использовании DiagnosticSource на практике вы в большинстве случаев будете подписываться на уже существующие события. Создавать собственные экземпляры `DiagnosticListener` и отправлять собственные события вам скорее всего не придётся (только если вы не разрабатываете какую-нибудь библиотеку), поэтому я не буду долго останавливаться на этом раздере.

Для создания собственного экземпляра `DiagnosticListener` вам необходимо будет объявить его как статическую переменную где-то в коде:

  

    private static readonly DiagnosticSource _myDiagnosticSource =
        new DiagnosticListener("MyLibraty");

После этого для отправки события можно использовать конструкцию вида:

  

    if (_myDiagnosticSource.IsEnabled("MyEvent"))
        _myDiagnosticSource.Write("MyEvent", new {  });

Более подробную информацию о создании собственных экземпляров `DiagnosticListener` можно найти в [DiagnosticSource User's Guide](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md), где подробно описаны Best Practices по использованию DiagnosticSource.

  

## Заключение

Пример, который мы рассмотрели, безусловно, очень абстрактный и вряд ли пригодится в реальном проекте. Но он отлично показывает, как этот механизм можно использовать для мониторинга и диагностики ваших приложений.

В следующей статье я приведу список известных мне событий, которые могут быть обработаны через DiagnosticSource, и покажу несколько практических примеров его использования.