Популярный open source — часть первая: 3 инструмента для работы с данными / Блог компании ИТ-ГРАД

Мы решили подготовить серию дайджестов с обзорами наиболее популярных open source проектов. В список попали самые обсуждаемые на Hacker News решения с открытым исходным кодом. Тема первой подборки — _инструменты и сервисы для работы с логами и базами данных_.

[![](../../_resources/f613c047800f42bf967b8d538f301937.jpeg)](https://habr.com/ru/company/it-grad/blog/436962/)

_/ фото [AKT.UZ](https://www.flickr.com/photos/165518082@N02/45188906785/) PD_

Мы поговорим о таких решениях, как **FoundationDB**, **LogDevice** и **Queryparser**. В прошлом году они активно обсуждались на Hacker News. Интерес был вызван тем, что к их разработке оказались причастны крупные ИТ-компании — Apple, Uber и Facebook. Это означает, что все три инструмента подходят для работы с масштабной и высоконагруженной ИТ-инфраструктурой.

* * *

## [FoundationDB](https://github.com/apple/foundationdb)

* * *

FoundationDB — это мультимодельная СУБД, которая относится к типу [NoSQL](https://ru.wikipedia.org/wiki/NoSQL). Её представили в 2012 году три инженера из компании Visual Sciences, [работавшие](https://janicemandel.com/2014/12/16/nick-lavezzo-decision/) над платформой визуализации данных (сегодня она является частью Adobe Analytics).

В отличие от других подобных систем, операции в FoundationDB соответствуют принципам [ACID](https://ru.wikipedia.org/wiki/ACID): атомарности, согласованности, изолированности и долговечности данных. СУБД, соблюдающие эту модель, считаются наиболее надежными и предсказуемыми, но в NoSQL некоторыми принципами ACID зачастую жертвуют ради большей производительности.

Еще одним плюсом FoundationDB является мощный низкоуровневый интерфейс. С его помощью любые системы могут использовать СУБД для распределённого хранения данных. Например, на базе FoundationDB можно строить фронтэнды для более крупных универсальных СУБД.

> Благодаря этим характеристикам FoundationDB быстро стала популярной. Её [внедрили](https://abdullin.com/foundationdb-is-back/) несколько облачных сервисов: сервис мониторинга Wavefront (сейчас часть VMware) и системы хранения данных Snowflake и SkuVault. На популярность FoundationDB повлияло и то, что с начала существования исходный код проекта был открытым.

Всё изменилось в 2015 году, когда компанию [приобрела](https://www.businessinsider.com/why-apple-bought-foundationdb-2015-3) Apple. ИТ-гигант закрыл доступ к коду FoundationDB и начал использовать СУБД в собственных онлайн-сервисах. Это решение доставило некоторые проблемы для разработчиков, которые [задействовали](https://news.ycombinator.com/item?id=9260769) FoundationDB в своих проектах. Но в апреле 2018 года в Apple приняли решение вернуть СУБД «под крыло» open source. Это принесло пользу не только ИТ-сообществу, но и самой Apple. За две недели к проекту [проявили интерес](https://www.foundationdb.org/blog/foundationdb-community-highlights-two-weeks-in/) более семи тысяч разработчиков, а на [тематическом форуме](https://forums.foundationdb.org/) открыли сотню новых тредов.

ИТ-гигант решил и дальше придерживаться стратегии «открытости». В ноябре 2018 года был [представлен](https://www.foundationdb.org/blog/announcing-document-layer/) новый компонент СУБД — Document Layer — он позволяет создавать хранилища документов. В будущем планируется разработка дополнительных инструментов. И внести свой вклад в создание продукта может любой желающий. Узнать как это сделать можно в официальном репозитории на GitHub — там [есть подробная инструкция](https://github.com/apple/foundationdb/blob/master/CONTRIBUTING.md#contributing-to-foundationdb).

* * *

## [LogDevice](https://github.com/facebookincubator/LogDevice)

* * *

LogDevice — система распределенного хранения логов, которую создали в Facebook. Она оптимизирована для записи последовательно поступающих данных: любая информация в системе сохраняется не как отдельный файл, а как часть определенного «потока записей». Это позволяет точно определить порядок поступления данных.

Изначально проект использовался для внутренних задач Facebook, но в сентябре 2018 года компания открыла его исходный код. До этого момента LogDevice был не так известен в ИТ-сообществе, но некоторые из читателей Hacker News уже заинтересовались инструментом. Например, [отметили](https://news.ycombinator.com/item?id=17975281) её потенциал в системах хранения данных для машинного обучения.

Но есть мнение, что популярность решение Facebook будет приобретать медленно. На рынке существует большое количество аналогичных инструментов (например, Apache Kafka). И они уже обладают большим числом интеграций, а LogDevice только предстоит ими обзавестись. К слову, сейчас разработчики инструмента [трудятся](https://logdevice.io/blog/2018/09/12/open-sourcing-announcement.html) над внедрением интеграции LogDevice с системой оркестровки контейнеров Kubernetes.

К участию приглашаются все желающие — требования к коду описаны [в отдельном документе репозитория на GitHub](https://github.com/facebookincubator/LogDevice/blob/master/CONTRIBUTING.md).

![](../../_resources/3b1bea7cf0584297bdd44626ca2becd3.jpeg)

_/ фото [Alexander Day](https://www.flickr.com/photos/photatojonez/25957663330/) [CC BY](https://creativecommons.org/licenses/by/2.0/)_

* * *

## [Queryparser](https://github.com/uber/queryparser)

* * *

Queryparser — это система парсинга для трёх SQL-диалектов: Vertica, Hive и Presto. Как и LogDevice, Queryparser изначально создавался для внутренних задач крупной ИТ-компании — на этот раз проект зародился в Uber.

В 2015 году инженеры компании решили обновить систему наименований объектов в базах данных и заменить названия в формате целых чисел на номера по стандарту [UUID](https://ru.wikipedia.org/wiki/UUID). Чтобы переписать все идентификаторы, инженерам нужно было выявить все ссылки в таблицах. Это оказалось сложной задачей: в Uber хранились десятки тысяч таблиц с данными, которые принадлежали разным отделам компании. Чтобы установить связи между несколькими базами данных, разработчики и создали Queryparser.

Инструмент успешно справился с задачей, но инженеры обнаружили для него и другие возможные применения. Например, автоматический мониторинг изменений в базах данных. Queryparser сохраняет все запросы об объединении потоков данных или создании новых и оповещает об этом пользователей БД, на которых эти изменения влияют.

Также Queryparser помог Uber собрать статистику о SQL-запросах и оптимизировать хранилище: редко используемые таблицы удалили, а базы, которые часто ссылались друг на друга, объединили.

> Исходный код Queryparser открыт с самого начала существования проекта, но только в 2018 году Uber [выпустила](https://eng.uber.com/queryparser/) подробную статью об инструменте. Её можно рассматривать как руководство по работе с системой. А в [репозитории](https://github.com/uber/queryparser) можно найти инструкцию по установке и указания для тех, кто хочет поучаствовать в развитии Queryparser.

В будущем Uber [планирует](https://github.com/uber/queryparser/blob/master/FUTURE.md) развивать решение и дальше. Например, добавлять поддержку новых диалектов SQL: PostgreSQL, MySQL и SQLite. Также среди задач компании — добавить проверку типов данных в запросах и перевод запросов с одного диалекта на другой.

* * *

_В следующий раз мы продолжим рассказ о популярных open source проектах 2018 года. Поговорим про [открытые решения для управления облаком](https://habr.com/ru/company/it-grad/blog/438032/) и инструменты для разработчиков._

* * *

Пара постов из Первого блога о корпоративном IaaS:

*   [Почему корпоративные заказчики используют ВМ, а не контейнеры](https://iaas-blog.it-grad.ru/tendencii/pochemu-korporativnye-zakazchiki-ispolzuyut-virtualnye-mashiny-a-ne-kontejnery/)
*   [Облачные сервисы: опыт использования IaaS российскими компаниями](https://iaas-blog.it-grad.ru/tendencii/oblachnye-servisy-opyt-ispolzovaniya-iaas-rossijskimi-kompaniyami/)

О чем мы пишем в Telegram-канале:

*   [Что управляет облаком: инструменты автоматизации от VMware](https://t.me/iaasblog/188)
*   [Блокчейн в работе облачного провайдера — 3 сферы применения](https://t.me/iaasblog/186)