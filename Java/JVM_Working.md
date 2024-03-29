# Как работает JVM
*Конспект статьи ["Как работает виртуальная машина Java — взгляд изнутри"](https://tproger.ru/blogs/jvm-insides/)*

## Структура class-файла

Файл начинается с числа: 0xCAFEBABE - с его помощью система понимает, что перед ней class-файл.

```
CA FE BA BE 00 00 00 34 00 26 OA 00 06 00 18 09 00 19
00 1A 07 00 1B 08 00 1C 0A 00 1D 00 1E 07 00 1F 01 00
```

Следующие 4 байта содержат старший и младший номера версий Java. В примере  major версия 0x34 — это Java SE 8 (для Java SE 11 — 0x37). 
С 9 байта - пул констант, представляющий из себя массив переменной длины. Перед массивом находится переменная, указывающая на его длину. Во всём class-файле константы указываются целочисленным индексом их положения в массиве (начальная константа имеет индекс 1). 
Каждый элемент пула констант начинается с однобайтового тега, определяющего его тип (например, если тег указывает, что константа является строкой, JVM получает значение тега 1 и обрабатывает следующее за тегом число как длину массива байт, которые необходимо считать, чтобы получить нужную нам строку полностью). 

| Тип константы  | Значение тега |
| ------------- | ------------- |
| CONSTANT_Class|   7|
| CONSTANT_Fieldref|    9|
| CONSTANT_Methodref|   10|
| CONSTANT_InterfaceMethodref|1 1|
| CONSTANT_String|  8|
| CONSTANT_Integer| 3|
| CONSTANT_Float|   4|
| CONSTANT_Long|    5|
| CONSTANT_Double|  6|
| CONSTANT_NameAndType| 12|
| CONSTANT_Utf8|    1|
| CONSTANT_MethodHandle|    15|
| CONSTANT_MethodType|  16|
| CONSTANT_InvokeDynamic|   18|

Следующие 2 байта — флаги доступа (определяют, это класс или интерфейс, общедоступный или абстрактный, является ли финальным). 
Следующие 4 байта - массив констант с именами класса и его родителя. За определением класса и его родителя идет число — размер массива интерфейсов, и сам массив. 

| Имя флага| Код| Определение
| ------------- | ------------- | --- 
| ACC_PUBLIC| 0x0001| Объявлен публичным
| ACC_FINAL| 0x0010| Объявлен финальным
| ACC_SUPER| 0x0020| Специальный флаг (введён в версии Java 1.1 для совместимости при вызове  методов родителя
| ACC_INTERFACE| 0x0200| Объявлен интерфейсом
| ACC_ABSTRACT| 0x0400| Объявлен абстрактным
| ACC_SYNTHETIC| 0x1000| Зарезервированное определение
| ACC_ANNOTATION| 0x2000| Объявлен аннотацией
| ACC_ENUM| 0x4000| Объявлен перечислением

Подобную структуру имеет и следующий блок — Fields — блок начинается с двухбайтового параметра количества полей в этом классе/интерфейсе. За ним идет массив структур переменной длины (каждая структура содержит информацию об одном поле: имя поля, тип, значение, если это, например, финальная переменная). Здесь отображаются поля, объявленые классом или интерфейсом, определённым в файле - поля от родительских классов и имплементированных интерфейсов задаются в своих class-файлах.
Далее идут методы. В массиве переменной длины содержатся структуры, куда входит полное описание сигнатуры метода: модификаторы доступа, имя метода, его атрибуты, представляющие из себя новую структуру.

## Загрузка классов
Существуют специальные классы-загрузчики:
1. Bootstrap — базовый загрузчик, загружает платформенные классы; является родителем всех остальных классов и частью платформы.
2. Extension ClassLoader — загрузчик расширений; загружает классы расширений, которые по умолчанию находятся в каталоге jre/lib/ext.
3. AppClassLoader — системный загрузчик классов из classpath;  загружает классы из каталогов и jar-файлов, указанных переменной среды CLASSPATH, системным свойством java.class.path или параметром командной строки -classpath.
4. Собственный загрузчик.
Главный класс приложения всегда загружается системным загрузчиком, остальные — различными пользовательскими загрузчиками. NB! — имя загрузчика создает уникальное пространство имен (в программе может существовать несколько классов с одним и тем же полным именем, если они обрабатывались разными загрузчиками, поэтому каждый загрузчик делегирует свои полномочия родителю — перед поиском класса для загрузки попытается узнать, не был ли загружен нужный класс раньше). 

После загрузки класса начинается этап линковки, состоящий из этапов:
1. Верификация байт-кода (статический анализ кода, выполняется один раз для класса - система проверяет, нет ли ошибок в байт-коде: н-р, корректность инструкций, переполнение стека и совместимость типов переменных).
2. Выделение памяти под статические поля и их инициализация.
3. Разрешение символьных ссылок — JVM подставляет ссылки на другие классы, методы и поля (в большинстве случаев это происходит лениво - при первом обращении к классу).
JVM получает один поток байтовых кодов (последовательность инструкций для виртуальной машины) для каждого метода в классе. Байт-код метода выполняется при вызове этого метода. Каждая инструкция состоит из однобайтового кода операции, за которым может следовать несколько операндов. Код операции указывает действие, которое нужно предпринять (на данный момент в Java более 200 операций). Все коды операций занимают только 1 байт, их максимальное число не может превысить 256 штук.
В основе работы JVM находится стек. Пример умножения двух чисел (байт-код и Java):

```
0: iconst_1 // взять число 1, положить в стек                       int a = 1;
1: istore_1 // сохранить это число в переменную 1 стека метода      int b = 5;
2: iconst_5 // взять число 5, положить в стек                       int c = a * b
3: istore_2 // сохранить его в переменную 2 стека метода| 
4: iload_1 // положить в стек переменную 1
5: iload_2 // положить в стек переменную 2
6: imul // достать из стека два числа, умножить их и положить в стек
7: istore_3 // взять из стека число и сохранить его в переменную 3
```

Коды операций сами по себе указывают тип и значение (н-р, iconst_1 указывает JVM на целочисленное значение, равное единице). Такие байт-коды определены для самых часто используемых констант (эти инструкции занимают 1 байт). Всего JVM поддерживает семь примитивных типов данных: byte, short, int, long, float, double и char.
Если бы мы хотели положить в переменную а другое значение, например 11112, то нам пришлось бы использовать инструкцию sipush:

```
0: sipush 11112
3: istore_1
```

Данные операции выполняются в так называемом фрейме стека метода - у каждого метода есть некоторая своя часть в общем стеке. В главном потоке исполнения программы создаются множество подстеков на каждый вызов метода.

|THE STACK|
|---|
|stack frame|
|stack frame|
|stack frame|
|stack frame|
|stack frame|

В каждом стек-фрейме хранится массив локальных переменных, который позволяет сохранять и доставать локальные переменные. NB! - если бы переменных в методе было много, использовался бы код операции сохранения значения вместе с указанием позиции переменной в массиве.
Для получения нужного значения из пула констант класса используются инструкции lcd и lcd_w (lcd может ссылаться только на константы с индексами от 1 до 255, т.к. размер её операнда составляет 1 байт; lcd_w имеет 2-байтовый индекс).

## Вызовы методов
Java предоставляет два основных вида методов: методы экземпляра и методы класса. Методы экземпляра используют динамическое (позднее) связывание, тогда как методы класса используют статическое (раннее) связывание.
Виртуальная машина Java вызывает метод класса на основании типа ссылки на объект, известный во время компиляции. С другой стороны, при вызыве метод экземпляра, машина выбирает метод на основе фактического класса объекта, известный только во время выполнения. Поэтому для вызова методов используются разные инструкции: invokevirtual и invokestatic. Данные функции ссылаются на запись в пуле констант в виде полного пути к необходимой функции. Виртуальная машина снимает нужное количество переменных со стека и передает в метод.
Возвращаемое методом значение кладётся на стек. Типы возвращаемых значений методов указаны ниже:

| Операция| Описание
|---|---
| ireturn| Помещает на стек значение типа int
| lreturn| Помещает на стек значение long,
| freturn| Помещает на стек значение float
| dreturn| Помещает на стек значение double
| areturn| Помещает на стек значение object
| return| Не изменяет стек

## Циклы
```
int i = 1000;           0: sipush        1000
while (i < 9999) {      3: istore_1
   i += 10;             4: iload_1
}                       5: sipush        9999
                        8: if_icmpge     17
                        11: iinc          1, 10
                        14: goto          4
                        17: return
```

Каждый раз на стеке оказывается два числа, которые сравниваются, и если i > 9999, происходит выход из цикла. При этом для создания цикла используется конструкция на основе goto, которая запрещена в самом языке Java, хотя ключевое слово и зарезервировано.

## Заключение
Изначально байт-код интерпретируется в большинстве JVM, но как только система замечает, что некоторый код используется очень часто, она подключает встроенный компилятор, который компилирует байт-коды в машинный код, тем самым значительно ускоряя работу приложения.