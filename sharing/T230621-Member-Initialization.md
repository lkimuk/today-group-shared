## T230621 Member Initialization
这次谈成员初始化。

C++存在3种方式初始化成员，分别为：
- ctor member initializer list
- ctor body
- default member initializer (in-class initializer)

关于 ctor member initializer list 和 ctor body 这2种方式，记得 *Effective C++* 有条款单独讨论过，但那书太早了，只包含C++98/03，default member initializer 是C++11才有的方式。

在C++98，只有 `static const int` 才能够直接在类内直接初始化，它可以保证初始化是在编译期完成的。
```cpp
// C++98
struct S {
	static const int x = 42; // OK
	const int y = 42; // Error
};
```

C++11允许非静态数据成员也可以以这种形式初始化，这种就叫 default member initializer。

下面分别用三种方式来初始化：
```cpp
// M1
struct S {
	int x = 42; // default member initializer
};

// M2
// the above snippet is equivalent to
struct S {
	int x;
	S() : x{ 42 } {} // ctor member initializer list
};

// M3
// the above snippet is also equivalent to
struct S {
	int x;
	S() {
		x = 42; // ctor body
	}
};
```

这三种方式初始化所得到的效果完全相同，也没有丝毫性能差异。

你可能会想，M2 难道不是比 M3 更好？对于 POD 成员来说，没有区别。但对于 non-POD 成员，M2 能够避免一次没必要的默认构造函数调用，所以一般都使用 M2，而不使用 M3。

还是举个例子：
```cpp
struct A {
	int x;
	A() { x = 42; }
	A(int x) { x = x; } // 对于构造函数来说，参数名和成员名可以完全相同
};

struct B {
	A a;
	B() { a.x = 42; }
};
```

对于 A 来说，使用 M2 和 M3 初始化没区别，但对于 B，由于它的成员是一个类，所以构造函数在调用 `a.x = 42;` 之前，会先调用 A 的默认构造函数。所以采用 M2 是一种更好的方式：
```cpp
struct B {
	A a;
	B() : a(42) {}
};
```

此时就可以免去一次额外的默认默认构造函数，如果默认构造函数中有许多工作，这种方式便能提高性能。

如果一个类没有默认构造函数，那就必须使用 ctor member initializer list 这种方式。

那为什么 C++11 又要搞出一个 default member initializer 呢？

考虑一下多个构造函数初始化的方式：
```cpp
struct S {
	int x;
	double y;
	std::string s;

	S() : x(1), y(2.0), s("text1") {}
	S(int val) : x(val), y(2.0), s("text1") {}
	S(double val) : x(1), y(val), s("text1") {}
};
```

你得为每个构造函数都编写一些相同的初始化代码，这带来了大量重复，通过 default member initializer 可以简化以上代码：
```cpp
struct S {
	int x = 1;
	double y = 2.0;
	std::string s{ "text1" };

	S() {}
	S(int val) : x(val) {}
	S(double val) : y(val) {}
};
```

这就是它的主要目的。

如果 default member initializer 和 ctor member initializer list 同时出现，那么前者会被忽略，所以也不会导致重复初始化。

最后，default member initializer 在C++11会使类变成 non-aggregate type，所以就无法再使用 aggregate initialization，但是C++14之后取消了这一限制。