Синтетические символы и модули (WinDbg/DbgEng)

В этой публикации речь пойдет о синтетических модулях и символах отладочного движка Windows (debugger engine). То есть о сущностях, которые можно искусственно добавить в отладчик для _раскраски_ адресов памяти.

![](../_resources/ad79c47acd1143b5831f0549c044acc1.png)

## Синтетические модули

Вообще [модуль](https://docs.microsoft.com/windows-hardware/drivers/debugger/modules) в рамках отладочного движка (debugger engine) это некоторая непрерывная область виртуальной памяти: базовый адрес модуля (адрес первого байта спроецированного файла) и его размер. Для каждого из типов отладки (живая отладка/анализ дампа, пользовательских режим/режим ядра и т.д.) отладчик имеет свой механизм чтения и поддержания в актуальном состоянии списка загруженных модулей. Самый простой способ принудительной актуализации списка загруженных модулей — [команда .reload с параметром /s](https://docs.microsoft.com/windows-hardware/drivers/debugger/-reload--reload-module-).

Если говорить, например, про живую отладку процесса пользовательского режима, то список загруженных модулей строится по [трем основным спискам загрузчика](https://github.com/processhacker/processhacker/blob/5f1db39c17910841ac93263fec8df9549bdf3a27/phnt/include/ntpsapi.h#L68): список очередности загрузки модулей (InLoadOrderModuleList), список модулей в памяти (InMemoryOrderModuleList) и список очередности инициализации (InInitializationOrderModuleList). С одной стороны нам ([почти](https://habr.com/ru/post/358650/)) ни что не мешает взять произвольные данные (из PE файла на диске, например) и собственноручно разметить их в памяти на исполнение. А с другой стороны давно известны техники удаления загруженной штатными средствами DLL из упомянутых выше трех списков загрузчика (скрытие модуля).

В обоих случаях, при отладке такой ситуации, будет полезным возможность пометить область скрытого модуля. Для этого подойдут синтетические модули. В качестве практического эксперимента можно просто заставить WinDbg выбросить загруженный модуль из своего внутреннего списка все той же командой [.reload, но у же с параметром /u](https://docs.microsoft.com/windows-hardware/drivers/debugger/-reload--reload-module-).

Далее, в качетве приводимых листингов, я буду использовать обычную отладку процесса пользовательского режима (notepad.exe). Для начала запомним базовый адрес модуля ntdll (0x07ffa8a160000) и вычислим его размер (0x01e1000):

  

    0:007> lm m ntdll
    Browse full module list
    start             end                 module name
    00007ffa`8a160000 00007ffa`8a341000   ntdll
    0:007> ? 00007ffa`8a341000-00007ffa`8a160000
    Evaluate expression: 1970176 = 00000000`001e1000

Рассмотрим [листиг](https://docs.microsoft.com/windows-hardware/drivers/debugger/uf--unassemble-function-) простой функции RtlFreeSid из модуля ntdll, которая вызывает функцию RtlFreeHeap из этого модуля, попутно запомнив адреса символов RtlFreeSid и RtlFreeHeap (0x7ffa8a1cbc20 и 0x7ffa8a176df0):

  

    0:007> uf ntdll!RtlFreeSid
    ntdll!RtlFreeSid:
    00007ffa`8a1cbc20 4053            push    rbx
    00007ffa`8a1cbc22 4883ec20        sub     rsp,20h
    00007ffa`8a1cbc26 488bd9          mov     rbx,rcx
    00007ffa`8a1cbc29 33d2            xor     edx,edx
    00007ffa`8a1cbc2b 65488b0c2560000000 mov   rcx,qword ptr gs:[60h]
    00007ffa`8a1cbc34 4c8bc3          mov     r8,rbx
    00007ffa`8a1cbc37 488b4930        mov     rcx,qword ptr [rcx+30h]
    00007ffa`8a1cbc3b e8b0b1faff      call    ntdll!RtlFreeHeap (00007ffa`8a176df0)
    00007ffa`8a1cbc40 33c9            xor     ecx,ecx
    00007ffa`8a1cbc42 85c0            test    eax,eax
    00007ffa`8a1cbc44 480f45d9        cmovne  rbx,rcx
    00007ffa`8a1cbc48 488bc3          mov     rax,rbx
    00007ffa`8a1cbc4b 4883c420        add     rsp,20h
    00007ffa`8a1cbc4f 5b              pop     rbx
    00007ffa`8a1cbc50 c3              ret

Так же, по приведенному листингу, нетрудно высчитать размер функции ntdll!RtlFreeSid:

  

    0:007> ? 00007ffa`8a1cbc51 - 0x07ffa8a1cbc20
    Evaluate expression: 49 = 00000000`00000031

А размер функции ntdll!RtlFreeHeap для нашего эксперимента не важен, поэтому для простоты можно принять его равным одному байту.

Теперь имитируем скрытие модуля ntdll:

  

    0:007> .reload /u ntdll
    Unloaded ntdll

Поле этого работа с адресом функции ntdll!RtlFreeSid в отладчике уже не так информативна (да и обратиться к началу функции уже можно только по виртуальному адресу):

  

    0:007> uf 00007ffa`8a1cbc20
    00007ffa`8a1cbc20 4053            push    rbx
    00007ffa`8a1cbc22 4883ec20        sub     rsp,20h
    00007ffa`8a1cbc26 488bd9          mov     rbx,rcx
    00007ffa`8a1cbc29 33d2            xor     edx,edx
    00007ffa`8a1cbc2b 65488b0c2560000000 mov   rcx,qword ptr gs:[60h]
    00007ffa`8a1cbc34 4c8bc3          mov     r8,rbx
    00007ffa`8a1cbc37 488b4930        mov     rcx,qword ptr [rcx+30h]
    00007ffa`8a1cbc3b e8b0b1faff      call    00007ffa`8a176df0
    00007ffa`8a1cbc40 33c9            xor     ecx,ecx
    00007ffa`8a1cbc42 85c0            test    eax,eax
    00007ffa`8a1cbc44 480f45d9        cmovne  rbx,rcx
    00007ffa`8a1cbc48 488bc3          mov     rax,rbx
    00007ffa`8a1cbc4b 4883c420        add     rsp,20h
    00007ffa`8a1cbc4f 5b              pop     rbx
    00007ffa`8a1cbc50 c3              ret

Для того, что бы добавить синтетический модуль нужно вызвать программный интерфейс [IDebugSymbols3::AddSyntheticModule](https://docs.microsoft.com/windows-hardware/drivers/ddi/content/dbgeng/nf-dbgeng-idebugsymbols3-addsyntheticmodule). Встроенных команд, которые бы позволяли совершить этот вызов, просто нет (насколько мне известно). Следовательно, нам нужно расширение отладчика, а тут на помощь придет [pykd](https://githomelab.ru/pykd/pykd) ![](../_resources/0115086a49ef4dbd98ba092314d093f8.png). В самом свежем (на момент написания статьи) релизе [0.3.4.3](https://githomelab.ru/pykd/pykd/wikis/0.3.4.3#synthetic-modules) как раз были добавлены функции для управления синтетическими модулями: **addSyntheticModule**(...) (которая нужна в нашем случае) и **removeSyntheticModule**(...).

По запомненному ранее базовому адресу и размеру модуля добавляем в отладчик информацию о _скрытом_ модуле ntdll (в случае ошибки будет брошено исключение, поэтому молчаливое исполнение — признак успешности, печать предупреждений можно игнорировать):

  

    0:007> !py
    Python 2.7.15 (v2.7.15:ca079a3ea3, Apr 30 2018, 16:30:26) [MSC v.1500 64 bit (AMD64)] on win32
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>> addSyntheticModule(0x07ffa8a160000, 0x01e1000, 'ntdll')
    *** WARNING: Unable to verify timestamp for ntdll
    >>> exit()

Теперь листинг дизассемблера стал немного информативнее:

  

    0:007> uf 00007ffa`8a1cbc20
    ntdll+0x6bc20:
    00007ffa`8a1cbc20 4053            push    rbx
    00007ffa`8a1cbc22 4883ec20        sub     rsp,20h
    00007ffa`8a1cbc26 488bd9          mov     rbx,rcx
    00007ffa`8a1cbc29 33d2            xor     edx,edx
    00007ffa`8a1cbc2b 65488b0c2560000000 mov   rcx,qword ptr gs:[60h]
    00007ffa`8a1cbc34 4c8bc3          mov     r8,rbx
    00007ffa`8a1cbc37 488b4930        mov     rcx,qword ptr [rcx+30h]
    00007ffa`8a1cbc3b e8b0b1faff      call    ntdll+0x16df0 (00007ffa`8a176df0)
    00007ffa`8a1cbc40 33c9            xor     ecx,ecx
    00007ffa`8a1cbc42 85c0            test    eax,eax
    00007ffa`8a1cbc44 480f45d9        cmovne  rbx,rcx
    00007ffa`8a1cbc48 488bc3          mov     rax,rbx
    00007ffa`8a1cbc4b 4883c420        add     rsp,20h
    00007ffa`8a1cbc4f 5b              pop     rbx
    00007ffa`8a1cbc50 c3              ret

А точнее мы видим, что функция внутри модуля ntdll вызывает другую функцию внутри этого же модуля.

  

## Синтетические символы

Листинг, после добавления синтетического модуля, стал более информативным, но все еще не так красноречив, как изначальный. В нем не хватает [символов](https://docs.microsoft.com/windows-hardware/drivers/debugger/symbols) (в нашем случае — имен функций RtlFreeSid и RtlFreeHeap). Что бы это исправить снова требуется вызов, но уже другого программного интерфейса — [IDebugSymbols3::AddSyntheticSymbol](https://docs.microsoft.com/windows-hardware/drivers/ddi/content/dbgeng/nf-dbgeng-idebugsymbols3-addsyntheticsymbol). И снова [pykd](https://githomelab.ru/pykd/pykd) ![](../_resources/0115086a49ef4dbd98ba092314d093f8.png) готов нам в этом помочь:

  

    0:007> !py
    Python 2.7.15 (v2.7.15:ca079a3ea3, Apr 30 2018, 16:30:26) [MSC v.1500 64 bit (AMD64)] on win32
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>> addSyntheticSymbol(0x07ffa8a1cbc20, 0x31, 'RtlFreeSid')
    <pykd.syntheticSymbol object at 0x0000014D47699518>
    >>> addSyntheticSymbol(0x07ffa8a176df0, 1, 'RtlFreeHeap')
    <pykd.syntheticSymbol object at 0x0000014D476994A8>
    >>> exit()

Результат очень похож на тот, что был при использовании символов из pdb-файла ntdll:

  

    0:007> uf 00007ffa`8a1cbc20
    ntdll!RtlFreeSid:
    00007ffa`8a1cbc20 4053            push    rbx
    00007ffa`8a1cbc22 4883ec20        sub     rsp,20h
    00007ffa`8a1cbc26 488bd9          mov     rbx,rcx
    00007ffa`8a1cbc29 33d2            xor     edx,edx
    00007ffa`8a1cbc2b 65488b0c2560000000 mov   rcx,qword ptr gs:[60h]
    00007ffa`8a1cbc34 4c8bc3          mov     r8,rbx
    00007ffa`8a1cbc37 488b4930        mov     rcx,qword ptr [rcx+30h]
    00007ffa`8a1cbc3b e8b0b1faff      call    ntdll!RtlFreeHeap (00007ffa`8a176df0)
    00007ffa`8a1cbc40 33c9            xor     ecx,ecx
    00007ffa`8a1cbc42 85c0            test    eax,eax
    00007ffa`8a1cbc44 480f45d9        cmovne  rbx,rcx
    00007ffa`8a1cbc48 488bc3          mov     rax,rbx
    00007ffa`8a1cbc4b 4883c420        add     rsp,20h
    00007ffa`8a1cbc4f 5b              pop     rbx
    00007ffa`8a1cbc50 c3              ret

  

## Особенности синтетических сущностей

Стоит отметить, что создание синтетических символов разрешено в любых существующих модулях, а не только в синтетических. То есть можно добавлять символы как к модулям [с доступными pdb-файлами](https://docs.microsoft.com/windows/desktop/dxtecharts/debugging-with-symbols), так и к модулям для которых у нас нет файлов с отладочной информацией. Но вот символы вне модулей создавать нельзя. То есть, что бы пометить произвольный адрес памяти вне границ существующих модулей, нужно в начале создать объемлющий синтетический модуль, а затем уже (если это необходимо, ведь имени модуля может быть вполне достаточно для _раскраски_) создать в нем синтетический символ.

Так же не лишним будет упомянуть, что синтетические модули и символы можно удалить _неосторожным_ движением руки:

  

*   перезагрузка отладочных символов модуля удаляет все синтетические символы этого модуля;
*   перезагрузка всех модулей приводит к удалению всех синтетических модулей.

  

## [!synexts](https://github.com/kITerE/synexts)

Хотя использование **pykd** удобно в подавляющем большинстве случаев (особенно учитывая наличие к нему [bootstarpper'а](https://githomelab.ru/pykd/pykd-ext)), иногда можно попасть в ситуацию, когда для использования pykd придется потратить значительные силы. Например, мне единоразово потребовалось отлаживать с host'овой машины, на которой работала Windows XP. Как выяснилось, уже достаточно давно pykd не поддерживает XP, а мне нужны были синтетические символы и модули. Мне показалось, что для этой задачи проще собрать отдельное небольшое расширение, которое будет решать узкий круг необходимых задач, чем восстановить полноценную поддержку XP для pykd. В результате был создан отдельный проект [!synexts](https://github.com/kITerE/synexts).

Это простое расширение, которое имеет два экспорта, доступных пользователю:

  

*   !synexts.addsymbol — создает синтетический символ в любом существующем модуле;
*   !synexts.addmodule — создает синтетический модуль во внутреннем списке отладочного движка.

Для демонстрации снова имитируем скрытие модуля ntdll:

  

    0:007> .reload /u ntdll
    Unloaded ntdll
    0:007> uf 00007ffa`8a1cbc20
    00007ffa`8a1cbc20 4053            push    rbx
    00007ffa`8a1cbc22 4883ec20        sub     rsp,20h
    00007ffa`8a1cbc26 488bd9          mov     rbx,rcx
    00007ffa`8a1cbc29 33d2            xor     edx,edx
    00007ffa`8a1cbc2b 65488b0c2560000000 mov   rcx,qword ptr gs:[60h]
    00007ffa`8a1cbc34 4c8bc3          mov     r8,rbx
    00007ffa`8a1cbc37 488b4930        mov     rcx,qword ptr [rcx+30h]
    00007ffa`8a1cbc3b e8b0b1faff      call    00007ffa`8a176df0
    00007ffa`8a1cbc40 33c9            xor     ecx,ecx
    00007ffa`8a1cbc42 85c0            test    eax,eax
    00007ffa`8a1cbc44 480f45d9        cmovne  rbx,rcx
    00007ffa`8a1cbc48 488bc3          mov     rax,rbx
    00007ffa`8a1cbc4b 4883c420        add     rsp,20h
    00007ffa`8a1cbc4f 5b              pop     rbx
    00007ffa`8a1cbc50 c3              ret

Снова восстанавливаем знания о модуле и символах в виде синтетических сущностей, но уже с использованием расширения !synexts:

  

    0:007> !synexts.addmodule ntdll C:\Windows\System32\ntdll.dll 00007ffa`8a160000 0x01e1000
    *** WARNING: Unable to verify timestamp for C:\Windows\System32\ntdll.dll
    Synthetic module successfully added
    0:007> !synexts.addsymbol RtlFreeSid 00007ffa`8a1cbc20 31
    Synthetic symbol successfully added
    0:007> !synexts.addsymbol RtlFreeHeap 00007ffa`8a176df0 1
    Synthetic symbol successfully added

Предполагается, что [скомпилированную библиотеку synexts.dll](https://github.com/kITerE/synexts/releases) (соответствующей используемому WinDbg разрядности) вы уже скопировали в директорию winext отладчика:

  

    0:007> .chain
    Extension DLL search Path:
        C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\WINXP;C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\winext;C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\winext\arcade;C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\pri;C:\Program Files (x86)\Windows Kits\10\Debuggers\x64;<…>
    Extension DLL chain:
        pykd.pyd: image 0.3.4.3, API 1.0.0, built Thu Jan 10 19:56:25 2019
            [path: C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\winext\pykd.pyd]
        synexts: API 1.0.0, built Fri Jan 18 17:38:17 2019
            [path: C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\winext\synexts.dll]
    <…>

И снова видим результат добавления синтетических символов в синтетический модуль:

  

    0:007> uf 00007ffa`8a1cbc20
    ntdll!RtlFreeSid:
    00007ffa`8a1cbc20 4053            push    rbx
    00007ffa`8a1cbc22 4883ec20        sub     rsp,20h
    00007ffa`8a1cbc26 488bd9          mov     rbx,rcx
    00007ffa`8a1cbc29 33d2            xor     edx,edx
    00007ffa`8a1cbc2b 65488b0c2560000000 mov   rcx,qword ptr gs:[60h]
    00007ffa`8a1cbc34 4c8bc3          mov     r8,rbx
    00007ffa`8a1cbc37 488b4930        mov     rcx,qword ptr [rcx+30h]
    00007ffa`8a1cbc3b e8b0b1faff      call    ntdll!RtlFreeHeap (00007ffa`8a176df0)
    00007ffa`8a1cbc40 33c9            xor     ecx,ecx
    00007ffa`8a1cbc42 85c0            test    eax,eax
    00007ffa`8a1cbc44 480f45d9        cmovne  rbx,rcx
    00007ffa`8a1cbc48 488bc3          mov     rax,rbx
    00007ffa`8a1cbc4b 4883c420        add     rsp,20h
    00007ffa`8a1cbc4f 5b              pop     rbx
    00007ffa`8a1cbc50 c3              ret