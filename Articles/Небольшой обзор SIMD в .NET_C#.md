Небольшой обзор SIMD в .NET/C#

Вашему вниманию предлагается небольшой обзор возможностей векторизации алгоритмов в .NET Framework и .NETCORE. Цель статьи познакомить с этими приёмами тех, кто их вообще не знал и показать, что .NET не сильно отстаёт от "настоящих, компилируемых" языков для нативной  
разработки.

Я только начинаю изучать приёмы векторизации, так что если кто из сообщества укажет мне на явные косяк, или предложит улучшенные версии описанных ниже алгоритмов, то буду дико рад.

  

## Немного истории

В .NET SIMD впервые появился в 2015 году с выходом .NET Framework 4.6. Тогда были добавлены типы Matrix3x2, Matrix4x4, Plane, Quaternion, Vector2, Vector3 и Vector4, которые позволили прозводить векторизированные вычисления. Позже был добавлен тип Vector&lt;T&gt;, который предоставил больше возможностей для векторизации алгоритмов. Но многие программисты всё равно были недовольны, т.к. вышеописанные типы ограничивали поток мыслей программиста и не давали возможности использовать полную мощь SIMD-инструкций современных процессоров. Уже в наше время, в .NET Core 3.0 Preview появилось пространство имён System.Runtime.Intrinsics, которое предоставляет гораздо большую свободу в выборе инструкций. Чтобы получить наилучшие результаты в скорости надо использовать RyuJit и нужно либо собирать под x64, либо отключить Prefer 32-bit и собирать под AnyCPU. Все бенчмарки я запускал на компьютере с процессором Intel Core i7-6700 CPU 3.40GHz (Skylake).

  

## Суммируем элементы массива

Я решил начать с классической задачи, которую часто пишут первой, когда речь заходит про векторизацию. Это задача нахождения суммы элементов массива. Напишем четыре реализации этой задачи, будем суммировать элементы массива Array:

Самая очевидная

  

    public int Naive() {
        int result = 0;
        foreach (int i in Array) {
            result += i;
        }
        return result;
    }

С использованием LINQ

  

    public long LINQ() => Array.Aggregate<int, long>(0, (current, i) => current + i);

С использованием векторов из System.Numerics:

  

    public int Vectors() {
        int vectorSize = Vector<int>.Count;
        var accVector = Vector<int>.Zero;
        int i;
        var array = Array;
        for (i = 0; i < array.Length - vectorSize; i += vectorSize) {
            var v = new Vector<int>(array, i);
            accVector = Vector.Add(accVector, v);
        }
        int result = Vector.Dot(accVector, Vector<int>.One);
        for (; i < array.Length; i++) {
            result += array[i];
        }
        return result;
    }

С использованием кода из пространства System.Runtime.Intrinsics:

  

    public unsafe int Intrinsics() {
        int vectorSize = 256 / 8 / 4;
        var accVector = Vector256<int>.Zero;
        int i;
        var array = Array;
        fixed (int* ptr = array) {
            for (i = 0; i < array.Length - vectorSize; i += vectorSize) {
                var v = Avx2.LoadVector256(ptr + i);
                accVector = Avx2.Add(accVector, v);
            }
        }
        int result = 0;
        var temp = stackalloc int[vectorSize];
        Avx2.Store(temp, accVector);
        for (int j = 0; j < vectorSize; j++) {
            result += temp[j];
        }   
        for (; i < array.Length; i++) {
            result += array[i];
        }
        return result;
    }

Я запустил бенчмарк на эти 4 метода у себя на компьютере и получил такой результат:

  

| Method | ItemsCount | Median |
| --- | --- | --- |
| Naive | 10  | 75.12 ns |
| LINQ | 10  | 1 186.85 ns |
| Vectors | 10  | 60.09 ns |
| Intrinsics | 10  | 255.40 ns |
| Naive | 100 | 360.56 ns |
| LINQ | 100 | 2 719.24 ns |
| Vectors | 100 | 60.09 ns |
| Intrinsics | 100 | 345.54 ns |
| Naive | 1000 | 1 847.88 ns |
| LINQ | 1000 | 12 033.78 ns |
| Vectors | 1000 | 240.38 ns |
| Intrinsics | 1000 | 630.98 ns |
| Naive | 10000 | 18 403.72 ns |
| LINQ | 10000 | 102 489.96 ns |
| Vectors | 10000 | 7 316.42 ns |
| Intrinsics | 10000 | 3 365.25 ns |
| Naive | 100000 | 176 630.67 ns |
| LINQ | 100000 | 975 998.24 ns |
| Vectors | 100000 | 78 828.03 ns |
| Intrinsics | 100000 | 41 269.41 ns |

Видно, что решения с Vectors и Intrinsics очень сильно выигрывают в скорости по сравнению с очевидным решением и с LINQ. Теперь надо разобраться что происходит в этих двух методах.

Рассмотрим подробнее метод Vectors:

  

**Vectors**

    public int Vectors() {
        int vectorSize = Vector<int>.Count;
        var accVector = Vector<int>.Zero;
        int i;
        var array = Array;
        for (i = 0; i < array.Length - vectorSize; i += vectorSize) {
            var v = new Vector<int>(array, i);
            accVector = Vector.Add(accVector, v);
        }
        int result = Vector.Dot(accVector, Vector<int>.One);
        for (; i < array.Length; i++) {
            result += array[i];
        }
        return result;
    }

  

*   int vectorSize = Vector&lt;int&gt;.Count; — это сколько 4х байтовых чисел мы можем поместить в вектор. Если используется аппаратное ускорение, то эта величина показывает сколько 4х-байтовых чисел можно поместить в один SIMD регистр. По сути она показывает над сколькими элементами данного типа можно проводить операции параллельно;
*   accVector — вектор, в котором будет накапливаться результат функции;  
    var v = new Vector&lt;int&gt;(array, i); — загружаются данные в новый вектор v, из array, начиная с индекса i. Загрузится ровно vectorSize данных.
*   accVector = Vector.Add(accVector, v); — складываются два вектора.  
    Например в Array хранится 8 чисел: {0, 1, 2, 3, 4, 5, 6, 7} и vectorSize == 4, тогда:  
    В первой итерации цикла accVector = {0, 0, 0, 0}, v = {0, 1, 2, 3}, после сложения в accVector будет: {0, 0, 0, 0} + {0, 1, 2, 3} = {0, 1, 2, 3}.  
    Во второй итерации v = {4, 5, 6, 7} и после сложения accVector = {0, 1, 2, 3} + {4, 5, 6, 7} = {4, 6, 8, 10}.
*   Остаётся только как-то получить сумму всех элементов вектора, для этого можно применить скалярное умножение на вектор, заполненный единицами: int result = Vector.Dot(accVector, Vector&lt;int&gt;.One);  
    Тогда получится: {4, 6, 8, 10} _{1, 1, 1, 1} = 4_ 1 + 6 _1 + 8_ 1 + 10 * 1 = 28.
*   В конце, если требуется, то досуммируются числа, которые не помещаются в последний вектор.

Если взглянуть в код метода Intrinsics:

  

**Intrinsics**

    public unsafe int Intrinsics() {
        int vectorSize = 256 / 8 / 4;
        var accVector = Vector256<int>.Zero;
        int i;
        var array = Array;
        fixed (int* ptr = array) {
            for (i = 0; i < array.Length - vectorSize; i += vectorSize) {
                var v = Avx2.LoadVector256(ptr + i);
                accVector = Avx2.Add(accVector, v);
            }
        }
        int result = 0;
        var temp = stackalloc int[vectorSize];
        Avx2.Store(temp, accVector);
        for (int j = 0; j < vectorSize; j++) {
            result += temp[j];
        }   
        for (; i < array.Length; i++) {
            result += array[i];
        }
        return result;
    }

То можно увидеть, что он очень похож на Vectors за некоторым исключением:

  

*   vectorSize задан константой. Так происходит потому что в этом методе явно используются Avx2 инструкции, которые оперируют с 256 битными регистрами. В реальном приложении должна быть проверка на то поддерживает ли текущий процессор Avx2 инструкции и если не поддерживает, вызывать другой код. Выглядит это примерно так:  
    
        if (Avx2.IsSupported) {
        DoThingsForAvx2();
        }
        else if (Avx.IsSupported) {
        DoThingsForAvx();
        }
        ...
        else if (Sse2.IsSupported) {
        DoThingsForSse2();
        }
        ...
    
*   var accVector = Vector256&lt;int&gt;.Zero; accVector объявлен как 256 битный вектор, заполненный нулями.
*   fixed (int* ptr = Array) — в ptr заносится указатель на array.
*   Дальше такие же операции как и в Vectors: загрузка данных в вектор и сложение двух векторов.
*   Для суммирования элементов вектора был применён такой способ:  
    *   создаётся массив на стэке: var temp = stackalloc int\[vectorSize\];
    *   загружается вектор в этот массив: Avx2.Store(temp, accVector);
    *   в цикле суммируются элементы массива.
*   потом досуммируются элементы массива, которые не помещаются в последний вектор

  

## Сравниваем два массива

Надо сравнить два массива байт. Собственно это та задача, из-за которой я начал изучать SIMD в .NET. Напишем опять несколько методов для бенчмарка, будем сравнивать два массива: ArrayA и ArrayB:

Самое очевидное решение:

  

    public bool Naive() {
        for (int i = 0; i < ArrayA.Length; i++) {
            if (ArrayA[i] != ArrayB[i]) return false;
        }
        return true;
    }

Решение через LINQ:

  

    public bool LINQ() => ArrayA.SequenceEqual(ArrayB);

Решение через функцию MemCmp:

  

    [DllImport("msvcrt.dll", CallingConvention = CallingConvention.Cdecl)]
    static extern int memcmp(byte[] b1, byte[] b2, long count);
    
    public bool MemCmp() => memcmp(ArrayA, ArrayB, ArrayA.Length) == 0;

С использованием векторов из System.Numerics:

  

    public bool Vectors() {
        int vectorSize = Vector<byte>.Count;
        int i = 0;
        for (; i < ArrayA.Length - vectorSize; i += vectorSize) {
            var va = new Vector<byte>(ArrayA, i);
            var vb = new Vector<byte>(ArrayB, i);
            if (!Vector.EqualsAll(va, vb)) {
                return false;
            }
        }
        for (; i < ArrayA.Length; i++) {
            if (ArrayA[i] != ArrayB[i])
                return false;
        }
        return true;
    }

С использованием Intrinsics:

  

    public unsafe bool Intrinsics() {
        int vectorSize = 256 / 8;
        int i = 0;
        const int equalsMask = unchecked((int) (0b1111_1111_1111_1111_1111_1111_1111_1111));
        fixed (byte* ptrA = ArrayA)
        fixed (byte* ptrB = ArrayB) {
            for (; i < ArrayA.Length - vectorSize; i += vectorSize) {
                var va = Avx2.LoadVector256(ptrA + i);
                var vb = Avx2.LoadVector256(ptrB + i);
                var areEqual = Avx2.CompareEqual(va, vb);
                if (Avx2.MoveMask(areEqual) != equalsMask) {
                    return false;
                }
            }
            for (; i < ArrayA.Length; i++) {
                if (ArrayA[i] != ArrayB[i])
                    return false;
            }
            return true;
        }
    }

Результат бэнчмарка на моём компьютере:

  

| Method | ItemsCount | Median |
| --- | --- | --- |
| Naive | 10000 | 66 719.1 ns |
| LINQ | 10000 | 71 211.1 ns |
| Vectors | 10000 | 3 695.8 ns |
| MemCmp | 10000 | 600.9 ns |
| Intrinsics | 10000 | 1 607.5 ns |
| Naive | 100000 | 588 633.7 ns |
| LINQ | 100000 | 651 191.3 ns |
| Vectors | 100000 | 34 659.1 ns |
| MemCmp | 100000 | 5 513.6 ns |
| Intrinsics | 100000 | 12 078.9 ns |
| Naive | 1000000 | 5 637 293.1 ns |
| LINQ | 1000000 | 6 622 666.0 ns |
| Vectors | 1000000 | 777 974.2 ns |
| MemCmp | 1000000 | 361 704.5 ns |
| Intrinsics | 1000000 | 434 252.7 ns |

Весь код этих методов, думаю, понятен, за исключением двух строк в Intrinsics:

  

    var areEqual = Avx2.CompareEqual(va, vb);
    if (Avx2.MoveMask(areEqual) != equalsMask) {
        return false;
    }

В первой два вектора сравниваются на равенство и результат сохраняется в вектор areEqual, у которого в элемент на конкретной позиции все биты выставляются в 1, если соответствующие элементы в va и vb равны. Получается, что если векторы из байт va и vb полностью равны, то в areEquals все элементы должны равняться 255 (11111111b). Т.к. Avx2.CompareEqual является обёрткой над \_mm256\_cmpeq_epi8, то [на сайте Intel](https://software.intel.com/sites/landingpage/IntrinsicsGuide/#expand=406,3832,784&text=_mm256_cmpeq_epi8) можно посмотреть псевдокод этой операции:  
Метод MoveMask из вектора делает 32-х битное число. Значениями битов являются старшие биты каждого из 32 однобайтовых элементов вектора. Псевдокод можно посмотреть [здесь](https://software.intel.com/sites/landingpage/IntrinsicsGuide/#expand=406,3832,784,3832&text=_mm256_movemask_epi8).

Таким образом, если какие-то байты в va и vb не совпадают, то в areEqual соответствующие байты будут равны 0, следовательно старшие биты этих байт тоже будут равны 0, а значит и соответствующие биты в ответе Avx2.MoveMask тоже будут равны 0 и сравнение с equalsMask не пройдёт.

Разберём небольшой пример, приняв что длина вектора 8 байт (чтобы писать было меньше):

  

*   Пускай va = {100, 10, 20, 30, 100, 40, 50, 100}, а vb = {100, 20, 10, 30, 100, 40, 80, 90};
*   Тогда areEqual будет равен {255, 0, 0, 255, 255, 255, 0, 0};
*   Метод MoveMask вернёт 10011100b, который нужно будет сравнивать с маской 11111111b, т.к. эти маски неравны, то выходит, что и векторы va и vb неравны.

  

## Подсчитываем сколько раз элемент встречается в коллекции

Иногда требуется посчитать сколько раз конкретный элемент встречается в коллекции, например, интов, этот алгоритм тоже можно ускорить. Напишем несколько методов для сравнения, будем искать в массиве Array элемент Item.

Самый очевидный:

  

    public int Naive() {
        int result = 0;
        foreach (int i in Array) {
            if (i == Item) {
                result++;
            }
        }
        return result;
    }

с использованием LINQ:

  

    public int LINQ() => Array.Count(i => i == Item);

с использованием векторов из System.Numerics.Vectors:

  

    public int Vectors() {
        var mask = new Vector<int>(Item);
        int vectorSize = Vector<int>.Count;
        var accResult = new Vector<int>();
        int i;
        var array = Array;
        for (i = 0; i < array.Length - vectorSize; i += vectorSize) {
            var v = new Vector<int>(array, i);
            var areEqual = Vector.Equals(v, mask);
            accResult = Vector.Subtract(accResult, areEqual);
        }
        int result = 0;
        for (; i < array.Length; i++) {
            if (array[i] == Item) {
                result++;
            }
        }
        result += Vector.Dot(accResult, Vector<int>.One);
        return result;
    }

С использованием Intrinsics:

  

    public unsafe int Intrinsics() {
        int vectorSize = 256 / 8 / 4;
        //var mask = Avx2.SetAllVector256(Item);
        //var mask = Avx2.SetVector256(Item, Item, Item, Item, Item, Item, Item, Item);
        var temp = stackalloc int[vectorSize];
        for (int j = 0; j < vectorSize; j++) {
            temp[j] = Item;
        }
        var mask = Avx2.LoadVector256(temp);
        var accVector = Vector256<int>.Zero;
        int i;
        var array = Array;
        fixed (int* ptr = array) {
            for (i = 0; i < array.Length - vectorSize; i += vectorSize) {
                var v = Avx2.LoadVector256(ptr + i);
                var areEqual = Avx2.CompareEqual(v, mask);
                accVector = Avx2.Subtract(accVector, areEqual);
            }
        }
        int result = 0;
        Avx2.Store(temp, accVector);
        for(int j = 0; j < vectorSize; j++) {
            result += temp[j];
        }
        for(; i < array.Length; i++) {
            if (array[i] == Item) {
                result++;
            }
        }
        return result;
    }

Результат бенчмарка на моём компьютере:

  

| Method | ItemsCount | Median |
| --- | --- | --- |
| Naive | 1000 | 2 824.41 ns |
| LINQ | 1000 | 12 138.95 ns |
| Vectors | 1000 | 961.50 ns |
| Intrinsics | 1000 | 691.08 ns |
| Naive | 10000 | 27 072.25 ns |
| LINQ | 10000 | 113 967.87 ns |
| Vectors | 10000 | 7 571.82 ns |
| Intrinsics | 10000 | 4 296.71 ns |
| Naive | 100000 | 361 028.46 ns |
| LINQ | 100000 | 1 091 994.28 ns |
| Vectors | 100000 | 82 839.29 ns |
| Intrinsics | 100000 | 40 307.91 ns |
| Naive | 1000000 | 1 634 175.46 ns |
| LINQ | 1000000 | 6 194 257.38 ns |
| Vectors | 1000000 | 583 901.29 ns |
| Intrinsics | 1000000 | 413 520.38 ns |

Методы Vectors и Intrinsics полностью совпадают в логике, отличия только в реализации конкретных операций. Идея в целом такая:

  

*   создаётся вектор mask, в котором в каждом элементе храниться искомое число;
*   Загружается в вектор v часть массива и сравнивается с маской, тогда в равных элементах в areEqual будут выставлены все биты, т.к. areEqual — вектор из интов, то, если выставить все биты одного элемента, получим -1 в этом элементе ((int)(1111\_1111\_1111\_1111\_1111\_1111\_1111_1111b) == -1);
*   вычитается из accVector вектор areEqual и тогда в accVector будет сумма того, сколько раз элемент item встретился во всех векторах v для каждой позиции (минус на минут дают плюс).

Весь код из статьи можно найти на [GitHub](https://github.com/tdkkdt/SIMDArticle)

  

## Заключение

Я рассмотрел лишь очень малую часть возможностей, которые предоставляет .NET для векторизации вычислений. За полным и актуальным список доступных интринсиков в .NETCORE под x86 можно обратиться к [исходному коду](https://github.com/dotnet/corefx/tree/master/src/Common/src/CoreLib/System/Runtime/Intrinsics/X86). Удобно, что там в C# файлах в summary каждого интринсика есть его же название из мира C, что упрощает и понимание назначения этого интринсика, и перевод уже существующих С++/С алгоритмов на .NET. Документация по System.Numerics.Vector доступна на [msdn](https://docs.microsoft.com/en-us/dotnet/api/system.numerics.vector).

На мой вгляд, у .NET есть большое преимущество перед C++, т.к. JIT компиляция происходит уже на клиентской машине, то компилятор может оптимизировать код под конкретный клиентский процессор, предоставляя максимальную производительность. При этом программист для написания быстрого кода может оставаться в рамках одного языка и технологий.