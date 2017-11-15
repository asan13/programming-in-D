# Object

Классы, не наследующие от какого-либо класса явно, автоматически наследуют
от класса **Object**.

Из этого определения следует, что в любой иерархии наследования самый верхний
класс наследует от **Object**:

```D
// ": Object" is not written; it is automatic
class MusicalInstrument :̶ ̶O̶b̶j̶e̶c̶t̶ {
    // ...
}

// Inherits Object indirectly
class StringInstrument : MusicalInstrument {
    // ...
}
```

Поскольку самый верхний класс наследует от **Object**, каждый класс косвенно
также наследует от **Object**. В этом смысле, любой класс "является"
**Object**.

Любой класс наследует следующие функции-члены **Object**:

- **toString**: Представление объекта в виде **string**.

- **opEquals**: Сравнение на равенство с другим объектом.

- **opCmp**: Порядок сортировки при сравнении с другим объектом.

- **toHash**: хеш-значение для ассоциативного массива.

Последние три функции акцентированы на значениях объектов. Также они позволяют
использовать класс в качестве ключа ассоциативного массива.

Поскольку эти функции наследуются, для их переопределения в подклассах нужно
использовать ключевое слово **override**.

_**Примечание:**_ **Object** также определяет другие члены. В эту главу войдут
только эти четыре функции-члены.


## typeid и TypeInfo

**Object** определяется в [модуле **object**](http://dlang.org/phobos/object.html) (
который не является частью пакета **std**). В этом модуле также определяется
**TypeInfo**, класс, предоставляющий информацию о типах.  Каждый тип имеет
отдельный объект **TypeInfo**, доступ к которому предоставляет _выражение_
**typeid**. Как мы увидим ниже, класс TypeInfo может использоваться для 
определения того, являются ли два типа одинаковыми, а также для доступа к 
специальным функциям типа (**toHash**, **postblit** и т.д., большинство из 
которых не рассматриваются в этой книге).

**TypeInfo** всегда относится к фактическому типу времени выполнения.
Например, хотя ниже **Violin** и **Guitar** наследуют от **StringInstrument**
напрямую, и косвенно от **MusicalInstrument**, их экземпляры **TypeInfo**
отличаются.  Они соответствуют типам **Violin** и **Guitar**, соответственно:

```D
class MusicalInstrument {
}

class StringInstrument : MusicalInstrument {
}

class Violin : StringInstrument {
}

class Guitar : StringInstrument {
}

void main() {
    TypeInfo v = typeid(Violin);
    TypeInfo g = typeid(Guitar);
    assert(v != g);    // ← the two types are not the same
}
```

Выражение **typeid** выше используется с самим типом **Violin**. Также
**typeid** может принимать _выражение_, в этом случае оно возвращает объект
**TypeInfo** для типа времени выполнения этого выражения. Например, следующая
функция принимает два параметра различных, но связанных типов:

```D
import std.stdio;

// ...

void foo(MusicalInstrument m, StringInstrument s) {
    const isSame = (typeid(m) == typeid(s));

    writefln("The types of the arguments are %s.",
             isSame ? "the same" : "different");
}

// ...

    auto a = new Violin();
    auto b = new Violin();
    foo(a, b);
```

Поскольку в этом конкретном вызове оба аргумента **foo()** представляют собой 
два объекта **Violin**, **foo()** определяет их типы, как одинаковые:

    The types of the arguments are the same.

В отличии от **.sizeof** и **typeof**, никогда не выполняющих свои выражения,
**typeid** всегда выполняет принимаемое выражение:

```D
import std.stdio;

int foo(string when) {
    writefln("Called during '%s'.", when);
    return 0;
}

void main() {
    const s = foo("sizeof").sizeof;     // foo() не вызывается 
    alias T = typeof(foo("typeof"));    // foo() не вызывается
    auto ti = typeid(foo("typeid"));    // foo() вызывается
}
```

Вывод показывает, что вызывалось только выражение в **typeid**:

    Called during 'typeid'.

Причина этого различия заключается в том, что фактические типы выражений во 
время выполнения могут быть неизвестны до тех пор, пока эти выражения не будут 
выполнены. Например, точный тип результата следующей функции будет либо
**Violin**, либо **Guitar**, в зависимости от аргумента **i**:

```D
MusicalInstrument foo(int i) {
    return (i % 2) ? new Violin() : new Guitar();
}
```

Существуют различные подклассы **TypeInfo** для различных типов, таких как
массивы, структуры, классы и т.д. Из них подкласс **TypeInfo_Class** возможно
особенно полезен.





