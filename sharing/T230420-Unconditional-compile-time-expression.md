## T230420: Unconditional compile-time expression
今天说两个关于编译期的小技巧。

看如下例子：

```cpp
struct S {
    int val;
    constexpr int size() const {
        return val * (val + 1) / 2;
    }
};

constexpr auto bad(S s) {
    // doesn't compile
    return std::array<int, s.size()>{};
}

int main() {
    constexpr S s{42};
    auto my_array = bad(s);
}
```

这里bad()的参数s就称为Conditional compile-time expression，它不能强保证在编译期执行，因为main()中的s可能不是constexpr的，因此bad()函数会编译失败。

为了让bad()函数一定在编译期执行，需要让其参数变为Unconditional compile-time expression。

有两种方式。

第一种方式，可以传递一个lambda作为constexpr function的参数，通过执行该lambda来得到一个constexpr value。

代码为：

```cpp
template <typename Callable>
constexpr auto good(Callable callable) {
    return std::array<int, callable().size()>{};
}

int main() {
	constexpr S s{42};
    constexpr auto make_compile_time_data = [] {
        return s;
    };
    auto my_array = good(make_compile_time_data);
}
```

这种方式我在constexpr string那篇文章中用过，这是一种不错的技巧。

但是你得额外提供一个lambda函数，若不想这么麻烦，可以采用第二种方式，这种方式是借助了NTTP。

```cpp
template <auto> struct compile_time_param {};
template <auto Data> inline auto compile_time_cast = compile_time_param<Data>{};

template <S s>
constexpr auto also_good(compile_time_param<s>) {
    return std::array<int, s.size()>{};
}

int main() {
	constexpr S s{42};
	auto my_array = also_good(compile_time_cast<s>);
}
```

NTTP和constexpr variables有着一样的性质，采用这种方式也可以将Conditional compile-time expression转换为Unconditional compile-time expression，提供强编译期保证。

这两个技巧在编译期编程中十分有用，可以用起来。