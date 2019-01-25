Приложение для iOS и Android на Kotlin + Flutter UI

[volkodav01](https://habr.com/ru/users/volkodav01/ "Автор публикации") сегодня в 09:00

*   Tutorial

![image](../_resources/e2d50085761948c095e61178e33e383b.png)

## Вступление

Всем привет. Какое-то время назад, я решил делать свой проект для Android и iOS одновременно. Естественно, встал вопрос о выборе технологий. Пару недель присматривался к популярным стекам и выбрал Kotlin/Native. Поскольку я являюсь Android-разработчиком, то с Kotlin знаком давно, а со Swift особого опыта не было и хотелось получить большую часть кода общего для обеих платформ. Следовательно, сразу встал вопрос, а как писать UI для iOS. Быстрый взгляд на рынок подсказал, что есть Flutter, который позволяет писать UI для двух платформ одновременно. Собственно, так и началась эта история.  
В статье описан опыт сборки Flutter в качестве UI и Kotlin для основной логики. **Важно:** под катом много картинок и инструкция, как собрать проект  

### Оглавление

  

*   [Создание общей библиотеки на Kotlin](https://habr.com/ru/post/437176/#Part1)
*   [Создание Android-приложения](https://habr.com/ru/post/437176/#Part2)
*   [Создаём iOS-проект](https://habr.com/ru/post/437176/#Part3)
*   [Добавление Flutter в Android-приложение](https://habr.com/ru/post/437176/#Part4)
*   [Добавление Flutter в iOS-проект](https://habr.com/ru/post/437176/#Part5)
*   [Заключение](https://habr.com/ru/post/437176/#Conclusion)

## Часть 1

  

### Создание общей библиотеки на Kotlin

  

1.  Выбираем Kotlin Mobile Shared library ![image](../_resources/54a571410deb4355a2d0d07d46e9de98.png)
2.  Далее ![image](../_resources/df02c21b502542d7bdc62b91ffa19c24.png)
3.  Указываем нашу рабочую папку, здесь я сделал отдельную папку под проект. Поскольку у меня будет 4 разных проекта и их удобнее держать в одном месте ![image](../_resources/0122980229ff4fff9a114a9a5b340c8d.png)
4.  Осталось указать в `local.properties` путь к `sdk.dir` и проект начинает собираться, у меня это путь `/Users/vlad/Library/Android/sdk` ![image](../_resources/2687318cfd0a4e8eaf57a97c02293f19.png)
5.  Структура проекта, изменяем имена пакетов с `` `sample на` habr.example`` ![image](../_resources/9e28a72a62344cbc8a55540edb6ff3a7.png)
6.  Пора приступать к публикации, вызываем `wrapper`. После этого у нас в проекте появится файлик `.gradlew` и можно будет с ним работать из терминала ![image](../_resources/8b328c1b03df49bfa5f68135eb10680b.png)
7.  Запускаем из терминала `./gradlew publishToMavenLocal` ![image](../_resources/728e4a5f9b2d428d82bc439c1e93b6b6.png)
8.  После этого в локальном maven-репозитории у нас появится 4 папки, в которых будут лежать наши библиотеки ![image](../_resources/15b22f8e1f1542a3a9d2b47484bac5c2.png)

## Часть 2

  

### Создание Android-приложения

Создаём android проект

  

1.  На момент написания статьи проект генерируется с «битой» зависимостью, поэтому убираем в конце `jre7`, получится `koltin-stdlib`, после этого проект начинает собираться![image](../_resources/927c7cf5fae7441a8ee07455657195db.png)
2.  Открываем `build.gradle` и в секцию `repositories` добавляем `mavenLocal`. **Важно!** Секция `repositories` должна быть та, что внутри `allprojects`, а не в `buildScript`![image](../_resources/bd00d5d6f8b545ebb47a57b3e4e450da.png)
3.  Теперь мы можем добавить нашу библиотеку как зависимость  
    `implementation 'habr.example:commonLibrary-jvm:0.0.1'`![image](../_resources/4dc1138fb57946ffa72705723fbb5794.png)
4.  Открываем `activity_main.xml` и указываем в `TextView` id `main_activity_text`![image](../_resources/e00b703f70c744cda3c028798f36ccc0.png)
5.  В `MainActivity.kt` просто устанавливаем текст в этом `TextView`![image](../_resources/4a28b455182c4239b2821fa42ddd944a.png)
6.  Отлично, к этому моменту у нас есть Android приложение, которое умеет использовать функцию `hello()` из нашей библиотеки
    
    **hello() на эмуляторе**
    
    ![image](../_resources/7d53faba908f42ddbde8a1fcc7be5546.png)
    

## Часть 3

  

### Создаём iOS-проект

  

1.  Выбираем Single View App![image](../_resources/39f98bccc1cb443da12efd34cb25d57f.png)
2.  Заполняем основную информацию и выбираем папку. Здесь важно выбирать папку, корневую для наших остальных проектов, поскольку в ней Xcode создаст подпапку с название проекта![image](../_resources/ea2d9c24549144d9b46bfb7cd66f19cb.png)![image](../_resources/cf9ac2da23e04e8db08d3232ad92de84.png)
3.  Первым делом добавляем `CocoaPods`. Для этого выполняем `pod init` в папке проекта, закрываем проект в `Xcode` и выполняем `pod install`. Видим, что установка завершилась успешно![image](../_resources/78776cf19ed841e1aabd3cf117ef6cbc.png)
4.  **Важно!** На сайте `CocoaPods` не рекомендуется добавлять папку `/Pods` в `.gitignore`, но я всё равно это сделал. Так как после добавления `flutter`, мы каждый `build` будем реконфигурировать зависимости. Пока что, мне это решение нравится больше, чем засорять `.git`
5.  Открываем проект через файлик `Awesome App.xcworkspace`![image](../_resources/43d348fb83c641b6902db30f2ba1e6cb.png)
6.  Открываем терминал, в нём переходим в папку нашей `commonLibrary` и запускаем `./gradlew linkDebugFrameworkIos`. После этого в нашей папке `build` появляется `iOSFramework`![image](../_resources/d12fb577ec2d4766acde20471c074e9c.png)
7.  Выбираем Target  
    ![image](../_resources/e7133afb78d2447185666f450d19391f.png)
8.  И для этой Target добавляем бинарник![image](../_resources/e3b3df18ec294344a9516bfe2bda705e.png)
9.  Выбираем Add Other![image](../_resources/8747329ea4b447b9b600e95907983b5c.png)
10. Указываем путь к `framework`, который получили на шаге 6 (`commonLibrary.framework`)![image](../_resources/afd066a466da408c981e16b5d05960aa.png)![image](../_resources/cea18a170a4c47b89ddc73a40b0cd897.png)
11. Теперь в проекте у нас должен отображаться этот `framework`![image](../_resources/580979ba2b9f4e1fb62d929174c72dae.png)
12. Идём в `Build Settings` и отключаем `Enable Bitcode`![image](../_resources/dc0e6f96cd074ea9b1df7915bbea2657.png)
13. Теперь нужно указать где именно искать наш `framework`, открываем `Framework Search Path`![image](../_resources/b961fbc312044a84b68bd714b4bb7c42.png)
14. Указываем путь `"${PODS_ROOT}/../../commonLibrary"`. Обязательно выбираем `recursive`. Конечно, можно обойтись и без этого, если более точно сконфигурировать путь. Но, поскольку это только начало проекта, сейчас нам важно убедиться в том, что вся эта связка заработает. А поменять пути мы можем и потом![image](../_resources/bb166822f7f64a38855dc214a884164d.png)
15. Нужно сделать так, чтобы при каждом `build` в `Xcode` собирался наш `framework` при помощи `gradle`. Открываем `Build Phases`![image](../_resources/9141f453322544d3b105d74051d026ae.png)
16. Добавляем новую `Script Phase`![image](../_resources/aee52969d33c43d5a6bf3107254e8d2b.png)
17. Добавляем код скрипта.  
    
        cd "${PODS_ROOT}/../../commonLibrary"
        echo $(pwd)
        ./gradlew linkIos
    
      
    Здесь мы просто переходим в папку проекта нашей библиотеки и запускаем `./gradlew linkIos`. Вызов `echo $(pwd)` нужен только для того, чтобы показать в консоли, в какую конкретно папку мы попали![image](../_resources/b32c5c7671ac47848f9295f96bb0bf42.png)
18. Сдвигаем нашу `build phase` в самый верх, сразу после `target dependencies`![image](../_resources/698a52f484f647779702685ce696a918.png)
19. Теперь открываем `ViewController` и добавляем вызов нашей функции из библиотеки![image](../_resources/1bf75cdec18b49eabbcff07b2f1b07ce.png)
20. Запускаем наш проект и видим![image](../_resources/61ef1ebfa63c4a41801dc56ce9eac8a9.png)

Отлично, это значит, что мы правильно подключили Kotlin Library к iOS-проекту  
Осталось всего ничего — добавить flutter, как framework для написания UI, в наши приложения и можно начинать разрабатывать продукт

## Часть 4

  

### Добавление Flutter в Android-приложение

Тут мне очень помогла статья на [github](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)

1.  Переходим в root-папку, где лежат все наши проекты и делаем `flutter create -t module flutter_ui`  
    ![image](../_resources/bf550d78f8d24f45a6f9322760f2a7cf.png)  
    
2.  Открываем `settings.gradle` и включаем наш flutter-модуль, как подпроект  
    ![image](../_resources/1b3f5a91d1894eee9bf95d6739af985b.png)  
    
3.  Открываем файл build.gradle и добавляем в зависимости наш проект  
    ![image](../_resources/e53ce98c41d64b6480b6cc0a05041c79.png)  
    
4.  Изменяем `MainActivity.kt` на `FlutterActivity`  
    ![image](../_resources/39d38271ab684a4c83df8f57d7dd850d.png)  
    
5.  Добавляем `App.kt`, в котором будем инициализировать `Flutter` при старте приложения  
    ![image](../_resources/80744d80a0ce4edeb1d046bee7800792.png)  
    
6.  Изменяем манифест и говорим, что теперь у нас есть класс для `Application`  
    ![image](../_resources/5cbb4517f5c442c4befa2a8d5f6df821.png)  
    
7.  Обязательно добавляем `java8`, без этого flutter не запустится  
    ![image](../_resources/b1bc4ed093b74c8da0a6890b2f4f0fd8.png)  
    
8.  Видим UI и в логах Hello from JVM, а это значит, что мы собрали вместе UI на Flutter и основную библиотеку на Kotlin/Native  
    [![](../_resources/3fba92b10a144b4aa67c52d5bfa519e5.png)](https://habrastorage.org/webt/xv/mt/8e/xvmt8etrk8q3lw3ijutfzxmjipg.png)  
    ![image](../_resources/e25611ed8f4543b6bb462429893faa9a.png)  
    
9.  Добавим в `MainActivity.kt` метод, который мы будем вызывать из Flutter. Здесь по событию из Flutter мы возвращаем наш `hello()` из kotlin-library  
    ![image](../_resources/cf05e6abfd604ffe946d0ed9e0052c25.png)  
    
10. И добавляем в `main.dart` код, который будет вызывать метод в `iOS/Android-части` приложения  
    ![image](../_resources/e3ae259254ca415cb1d84076a6c92b1b.png)  
    
11. Получаем  
    [![](../_resources/3811fcd5dc414438a7a2861bdb4c3aec.png)](https://habrastorage.org/webt/uo/wy/xe/uowyxed9b6evriyfis7igoengy8.png)  
    

## Часть 5

  

### Добавление Flutter в iOS-приложение

  

1.  Обновляем наш `Podfile`  
    
        
            flutter_application_path = File.expand_path("../flutter_ui", File.dirname(path))
            eval(File.read(
            File.join(
            flutter_application_path,
            '.ios',
            'Flutter',
            'podhelper.rb')),
            binding)
            
    
      
    ![image](../_resources/e3b6c3951acb4810bfb97d92338b1588.png)
    
    ![image](../_resources/b10f824eec7a4cc6bade698c52fe4b22.png)
    
2.  **Важно**. Добавьте `$(inherited)` в первую строчку `framework search paths`. Обязательно проверьте что у вас `framework search paths` не пустые![image](../_resources/19985194776a463cb9f8405be8f47d9d.png)Когда вы меняете зависимости в `some/path/my_flutter/pubspec.yaml`, вам нужно запустить `flutter packages get` из `some/path/my_flutter`, чтобы обновить зависимости в `podhelper.rb`. После этого нужно запустить `pod install` из `some/path/MyApp`
3.  Добавим ещё 1 `Build Phase`, только уже для Flutter. **Выше того, что мы добавляли в части 3** `Script phase`  
    
        
            "$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" build
            "$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" embed
            
    
      
    ![image](../_resources/65a3e9153b6d4bb78ebd1bdaaf4ec478.png)
4.  Заменим наш `AppDelegate` на `FlutterAppDelegate`![image](../_resources/a8463963550444058b1faa08b8d50d47.png)
5.  Обновим `ViewController`![image](../_resources/41690c77180b4f11b6117f01a8de7b9d.png)
6.  Обернем наш `ViewController` в `NavigatorController`![image](../_resources/8f4b7899649c4fe6b574d52a0fc088ab.png)[![](../_resources/a12cac48d07343e2b1ca818ae4e82a28.png)](https://habrastorage.org/webt/p0/8m/rs/p08mrsx01f11yi74t-efmdcvgg0.png)
7.  Теперь приложение запускается. Но, пока что, у нас нет связи между библиотекой и flutter [![](../_resources/46cda56ef7c8448fa5437d9613b6422f.png)](https://habrastorage.org/webt/uj/nl/zi/ujnlzi8tmnfcnw0uistebrfhlzu.png)
8.  Добавим эту связь с помощью `FlutterMethodChannel`![image](../_resources/f1ae417aacee4f04b49f3826fff5d3d2.png)  
    
9.  Отлично, теперь и iOS приложение использует `flutter` для `UI` и `kotlin` для основной логики  
    [![](../_resources/2a449f5631234ab6bfc73dc5222055ac.png)](https://habrastorage.org/webt/ak/xt/f6/akxtf6fve4sqyycnnbfwwstdleo.png)  
    

## Заключение

Что важно здесь сказать: я не претендую на то, что вы узнали что-то новое или уникальное. Я просто поделился свои опытом, поскольку для того, чтобы заставить это всё работать вместе, потратил около 4х рабочих дней. И не смог найти примеров кода проекта, который использует одновременно и Kotlin/Native и Flutter

Итоговые проекты

1.  [группа проектов](https://gitlab.com/habr-example-flutter-and-kotlin)
2.  [flutter-ui](https://gitlab.com/habr-example-flutter-and-kotlin/flutter-ui)
3.  [ios](https://gitlab.com/habr-example-flutter-and-kotlin/ios)
4.  [android](https://gitlab.com/habr-example-flutter-and-kotlin/android)
5.  [common-library](https://gitlab.com/habr-example-flutter-and-kotlin/common-library)

Список ссылок, которые мне помогли, но не сразу

1.  [Сам flutter](https://flutter.io/)
2.  Связь между нативным кодом и UI [flutter platform-channels](https://flutter.io/docs/development/platform-integration/platform-channels)
3.  Добавить flutter в существующее приложение [github](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)
4.  Kotlin Native [native-overview](https://kotlinlang.org/docs/reference/native-overview.html)

Теги:

*   [Flutter](https://habr.com/ru/search/?q=%5BFlutter%5D&target_type=posts)
*   [Kotlin](https://habr.com/ru/search/?q=%5BKotlin%5D&target_type=posts)
*   [Разработка мобильных приложений](https://habr.com/ru/search/?q=%5B%D0%A0%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0%20%D0%BC%D0%BE%D0%B1%D0%B8%D0%BB%D1%8C%D0%BD%D1%8B%D1%85%20%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B9%5D&target_type=posts)
*   [Разработка под Android](https://habr.com/ru/search/?q=%5B%D0%A0%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0%20%D0%BF%D0%BE%D0%B4%20Android%5D&target_type=posts)
*   [Разработка под iOSK](https://habr.com/ru/search/?q=%5B%D0%A0%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0%20%D0%BF%D0%BE%D0%B4%20iOSK%5D&target_type=posts)