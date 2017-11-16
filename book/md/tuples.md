# Кортежи

Кортежи (tuples) предназначены для объединения нескольких значений, которые 
будут использоваться как один объект. Они реализованы с помошью шаблона
**Tuple** из модуля **std.typecons**.

**Tuple** использует **AliasSeq** из модуля **std.meta** для некоторых 
операций.

В этой главе рассматриваются только наиболее распространенные операции с 
кортежами. Для получения дополнительной информации о кортежах и шаблонах 
смотрите [Philippe Sigaud's D Templates: A Tutorial](https://github.com/PhilippeSigaud/D-templates-tutorial).

## Tuple и tuple()

Кортежи обычно создаются с помощью удобной функции **tuple()**:

```D
import std.stdio;
import std.typecons;

void main() {
    auto t = tuple(42, "hello");
    writeln(t);
}
```

Вызов **tuple()** создает объект, состоящий из значений **int** 42 и
**string** **hello**. Программа выводит тип объекта кортежа и его члены:

    Tuple!(int, string)(42, "hello")

Тип кортежа, приведенный выше, является эквивалентом следующего 
псевдоопределения **struct** и, вероятно, был реализован точно так же:

```D
// ЭквивалентTuple!(int, string)
struct __Tuple_int_string {
    int __member_0;
    string __member_1;
}
```

К элементам кортежа обычно обращаются по индексу. Этот синтаксис предполагает, 
что кортежи можно рассматривать как массивы, состоящие из разных типов 
элементов:

```D
    writeln(t[0]);
    writeln(t[1]);
```

Вывод:

    42
    hello

## Свойства элементов

Есть возможность обращаться к элементам кортежа через свойства, если он был 
построен непосредственно через шаблон **Tuple** вместо функции **tuple()**. 
Тип и имя каждого элемента определяются двумя последовательными параметрами
шаблона:

```D
    auto t = Tuple!(int, "number",
                    string, "message")(42, "hello");
```

Это определение позволяет обращаться к элементам через свойства **.number** и
**.message**:

```D
    writeln("by index 0 : ", t[0]);
    writeln("by .number : ", t.number);
    writeln("by index 1 : ", t[1]);
    writeln("by .message: ", t.message);
```

Вывод:

    by index 0 : 42
    by .number : 42
    by index 1 : hello
    by .message: hello


## Расширение элементов в список значений

Члены кортежа могут быть развернуты в список значений, который может быть
использован как список аргументов в вызове функции. Развернуть элементы можно,
либо используя свойство **.expand**, либо через срез:

```D
import std.stdio;
import std.typecons;

void foo(int i, string s, double d, char c) {
    // ...
}

void bar(int i, double d, char c) {
    // ...
}

void main() {
    auto t = tuple(1, "2", 3.3, '4');

    // Две следующих строки эквивалентны foo(1, "2", 3.3, '4'):
    foo(t.expand);
    foo(t[]);

    // эквивалентно bar(1, 3.3, '4'):
    bar(t[0], t[$-2..$]);
}
```

В этом примере кортеж состоит из четырех значений **int**, **string**,
**double** и **char**. Поскольку они соответствуют списку парамеров **foo()**,
их расширение можно использовать, как агргументы **foo()**. Для вызова
**bar()** аргументы собираются из первого и двух последних элементов кортежа.

Пока члены кортежа совместимы, как элементы одного и того же массива, их можно
использовать, как литеральные значения массива:

```D
import std.stdio;
import std.typecons;

void main() {
    auto t = tuple(1, 2, 3);
    auto a = [ t.expand, t[] ];
    writeln(a);
}
```

Литерал массива выше инициализируется посредством двойного расширение кортежа:

    [1, 2, 3, 1, 2, 3]

## foreach времени компиляции

Благодаря расширяемости значений кортеж также можно использовать в инструкции 
**foreach**:

```D
    auto t = tuple(42, "hello", 1.5);

    foreach (i, member; t) {
        writefln("%s: %s", i, member);
    }
```

Вывод:

    0: 42
    1: hello
    2: 1.5

В этом примере может возникнуть впечатление, что цикл **foreach** запускается
во время выполнения программы, но это не так. **foreach** при работе с
элементами кортежа _разворачивает_ тело цикла для каждого элемента.
Приведенный выше код эквивалентен следующему:

```D
    {
        enum size_t i = 0;
        int member = t[i];
        writefln("%s: %s", i, member);
    }
    {
        enum size_t i = 1;
        string member = t[i];
        writefln("%s: %s", i, member);
    }
    {
        enum size_t i = 2;
        double member = t[i];
        writefln("%s: %s", i, member);
    }
```

Разворачивание цикла происходит по причине того, что если элементы кортежа
имеют различный тип, тело **foreach** должно компилироваться по разному для
каждого типа.

## Возврат нескольких значений из функций

Кортежи можно использовать в качестве простого решения проблемы возврата
функцией единственного значения. Рассмотрим в качестве примера
**std.algorythm.findSplit()**. **findSplit()** выполняет поиск диапазона
внутри другого диапазона и возвращает результат, состоящий из трех частей:
часть до найденного диапазона, сам найденный диапазон, и часть после
найденного:

```D
import std.algorithm;

// ...

    auto entireRange = "hello";
    auto searched = "ll";

    auto result = findSplit(entireRange, searched);

    writeln("before: ", result[0]);
    writeln("found : ", result[1]);
    writeln("after : ", result[2]);
```

Вывод:

    before: he
    found : ll
    after : o

В качестве альтернативы для возврата нескольких значений можно было бы 
использовать объект **struct**:

```D
struct Result {
    // ...
}

Result foo() {
    // ...
}
```


## AliasSeq

**AliasSeq** определяется в модуле **std.meta**. Он представляет концепцию,
обычную для компилятора, но недоступную программисту в качестве объекта:
разделенный запятыми список значений, типов и символов (т.е аргументы шаблона
**alias**). Вот три примера таких списков:

- Список аргументов функции

- Список аргументов шаблона

- Список литералов массива

В следующих трёх строках три примера этих списков в том же порядке:

```D
    foo(1, "hello", 2.5);         // аргументы функции
    auto o = Bar!(char, long)();  // аргументы шаблона
    auto a = [ 1, 2, 3, 4 ];      // литеральные элементы массива
```

**Tuple** использует **AliasSeq** при расширении своих элементов.

Название **AliasSeq** происходит от "alias sequence". **AliasSeq** может
содержать типы, значения и символы. (**AliasSeq** и **std.meta** ранее
назывались **TypeTuple** и **std.typetuple**, соответственно.)

Эта глава включает примеры **AliasSeq**, состоящих только их типов или только 
из значений. Примеры использования с ними обоими будут рассмотрены в следующей 
главе, как и использование **AliasSeq** с вариативными шаблонами, с которыми
он особенно полезен.

## AliasSeq, состоящий из значений

