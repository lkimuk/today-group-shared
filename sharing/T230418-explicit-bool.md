## T230418: explicit(bool)
explicit(bool)是C++20引入的一个特性，称为Conditionally explicit。

核心目的是简化泛型类型的实现，提高性能，减少编译时间。

举个简单的例子：
```cpp
void foo(std::string);
foo("a");   // OK
foo("a"sv); // Error
```
因为没有相应的稳式转换， std::string对于std::string_view的构造是explicit的，所以编译失败。这是普通的explicit用法，若foo()的参数变为wrapper<std::string>，这里的wrapper就是前面所说的泛型类型，此时依旧想保持foo()的调用行为，在20之前可以使用SFINAE。
```cpp
template<class T>
struct wrapper {
  template<class U, std::enable_if_t<std::is_convertible_v<U, T>>* = nullptr>
  wrapper(U const& u) : t_(u) {}
  
  template<class U, std::enable_if_t<!std::is_convertible_v<U, T>>* = nullptr>
  explicit wrapper(U const& u) : t_(u) {}

  T t_;
};
```
这里通过SFINAE动态地设置了explicit，除去SFINAE本身的缺点，一种显而易见的缺点就是得写两个重载函数。此时就可以使用explicit(bool)来简化代码：
```cpp
template<class T> 
struct wrapper { 
  template<class U> 
  explicit(!std::is_convertible_v<U, T>) 
  wrapper(U const& u) : t_(u) {} 

  T t_; 
};
```

这就是Conditionally explicit的原理与目的，标准中的许多类型，都使用了该手法，比如std::pair, std::span, std::optional, etc.

下面是一些其他扩展。

1、 你可能会在有些源码中看到explicit(true)或是explicit(false)这些的代码，乍看好像多此一举，不明所以。这种方式的目的其实跟写注释差不多，就是标明函数是explicit还是implicit，一目了然。

2、explicit(bool)也可以结合Concepts使用，std::pair的实现就是一个例子。

3、explicit(bool)能够避免疏忽导致的昂贵转换开销，并且能够保持类型语义安全。

4、还有一种用法是根据参数包来动态决定是否explicit，但是具体什么时候用得上，可能得真有需求了才能知道。
```cpp
struct Foo {
    template <typename ... Ts>
    explicit(sizeof...(Ts) == 1) Foo(Ts&&...) {}
};

Foo good = {1,2}; // OK
Foo bad = {1}; // error
```