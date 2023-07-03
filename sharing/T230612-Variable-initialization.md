## T230612 Variable initialization
前几天群里聊到了初始化，所以这次选择该主题。

这个主题也是一个基本内容，乃大厦之基，联系太过广泛，所以也是可深可浅。

局部值、全局值、成员值、静态值、内联值、线程值……每一个单独拿出来可能还好理解，但一组合起来就比较复杂了。有些内容，如果不懂操作系统、编译原理和汇编，还难以真正理解其来龙去脉。尤其是静态值，一个static本身就有十几种不同的意思，再和inline, constexpr, constinit……一组合，可谓是复杂到了极点。

这次先来统一下变量初始化的相关术语，更多内容后面再来说。

C++初始化包含default initialization, value initialization, direct initialization, copy initialization, list initialization, aggregate initialization, reference initialization这么几种。

声明一个变量，不做任何初始化（或是类中的成员没有初始化），就会采用default initialization。

比如
```cpp
type t;
new type;
```

这里type可以是primitive types和class-types，primitive types的默认初始化是保持未初始化，class-types会调用default ctor。

要让primitive types和class-types都初始化，就需要采用value initialization。

例子：
```cpp
type t{};
type t();
new type();
new type{};
```

对于class-types来说，它的效果和default initialization相同，但这种方式，primitive types会进行zero initialization。也就是说，假如是`int t; int t{}`，前者未初始化，后者会初始化为0。

初始化对象时，如果不使用显式的=，则会采用direct initialization。

例子：
```cpp
type obj(arg1, ...);
type(arg1, ...);
new type(arg1, ...);
static_cast<type>(obj);
[arg](){} // arg captured by value
```

初始化对象时，如果使用显式=或是隐式拷贝，则会采用copy initialization。

例子：
```cpp
type obj = other;
foo(other);   // pass by value
return other; // return by value
throw obj;
```

使用{}初始化叫list initialization，它又可分为value list initialization, direct list initialization和copy list initialization。

例子：
```cpp
type obj{}; // value list initialization
type{arg};  // direct list initialization
type obj = {arg}; // copy list initialization
```

list initialization有一种特殊情况称为aggregate initialization，用于初始化数组或是简单的struct/class（叫aggregate，不能有ctors，所有成员必须public）。

例子：
```cpp
type t = {arg1, ...}
type t{arg1, arg2};
```

C++20下，aggregate initialization又增加了一种新的初始化方式，叫designated initializers，可以更方便地初始化。

```cpp
point p { .x = .1, .y = .2 };
```

最后是reference initialization，就是用来初始化引用的。

```cpp
int& b= a;
type& ref { other };
type& ref = { arg1, arg2, ... }
point& ref = { .x = p.x, .y = p.y }
```

术语统一了，后面再来看其他的内容。