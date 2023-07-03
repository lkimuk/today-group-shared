## T230428 Extra Parentheses, Different Meaning
今天说一下额外()产生不同意义的情况。

多数情况下，额外的()是不影响语义的，但在以下5种情况，有无()则意义不同，此时()就有了特殊的作用。

一、禁止ADL。

在「洞悉C++函数重载决议」提到过，使用额外的()可以防止ADL。

```cpp
namespace N {
    struct S { };
    void f(S);
}

void g() {
    N::S s;
    f(s);   // OK: calls N::f
    (f)(s); // error: N::f not considered; parentheses
            // prevent argument-dependent lookup
}
```

这是因为，ADL起作用的前提是f是一个unqualified-id，而(f)是一个primary expression，所以ADL被禁止了。

二、结合decltype判断Value Categories。

decltype(e)得到的是变量名称的类型，decltype((e))得到的是表达式的类型。注意：lvalue, rvalue, prvalue这些叫法是针对表达式的，而非value。

因此可以利用decltype((e))来判断Value Categories。

```cpp
std::string s {"text"};
auto&& str = std::move(s);

// The type of names
fmt::print("xvalue : {}\n", std::is_rvalue_reference_v<decltype(str)>);   // true
// The type of expressions
fmt::print("lvalue : {}\n", std::is_lvalue_reference_v<decltype((str))>); // true
```

三、解决vexing parses problem。

C++为了向后兼容C，对象与函数声明解析会产生歧义，这个问题就称为vexing parses。

一个例子：

```cpp
void foo(double v) {
	int m(int(v)); // function declaration
	int n(int());  // function declaration
}
```

这里m和n是函数声明，所以Scott Meyers称之为最令人迷惑的解析（most vexing parses）。

此处加上一个额外的括号就可以解决这个问题：

```cpp
void foo(double v) {
	int m((int(v))); // object declaration
	int n((int()));  // object declaration
}
```

为什么可以呢？这是由于C的语法不允许(variable)里出现变量，只允许(type)里出现类型，而C++的语法(expression)里期望出现的是表达式，于是在解析时便可将C的歧义排除，剩下的就是对象声明。

这也是为何C++之后更推荐使用uniform initialization {}进行初始化的原因，这种初始化方式可以避免解析问题，真正做到一致性初始化（别提vector\~\~）。

四、comma operator。

在逗号表达式中，()会改变参数个数，因为它只会返回最右边的值。

例子：
```cpp
f(a, (t=3, t+2), c); // 3 arguments, (a, t = 5, c)
```

五、macros。

在宏中加()已是老生常谈，这应该是5条中最广为熟知的一条。

```cpp
#define TIMES_BAD(a, b) a * b
#define TIMES_GOOD(a, b) (a) * (b)
TIMES_BAD(1 + 2, 2 + 3);  // 8
TIMES_GOOD(1 + 2, 2 + 3); // 15
```

在宏中有逗号的情况下，也要多加一个()，assert宏是一个典型例子。

```cpp
assert(std::is_same_v<int, int>);   // error: assert does not take two arguments
assert((std::is_same_v<int, int>)); // OK: one argument
```