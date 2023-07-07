## inline constexpr
今天谈谈 inline constexpr。

上次讲过 static constexpr，它用于 function scope/class scope ，此时 constexpr 会隐式 inline (class scope)，static 则表示存储时期为 static。组合起来使用，一是能够直接在类中初始化通过 default member initializer 定义静态成员，二是所有对象能够共享数据，三是可以强保证发生于编译期，四是能够强保证 Lambdas 隐式捕获此数据。同时，也能够解决 SIOF。

上次也讲过 static inline，但是没有谈及 inline constexpr。

讨论之前，大家再回忆一个提过的准则：constexpr 在 file scope 下会隐式 internal linkage。

所以在 file scope 下不会写 static constexpr，这里的 static 是冗余的。但是单独只写一个 constexpr 修饰数据，默认的 internal linkage 会导致在多个 TUs 间拷贝数据，如果数据大小是 4000 bytes，那么两个 TUs 链接之后程序大小就有 8000 bytes。

通过增加一个 inline，就能够将 constexpr 隐式加上的 internal linkage 变成 external linkage，从而消除拷贝开销，所有 TUs 共享一份数据。

因此一般全局的这种变量，都会使用 inline constexpr 修饰，你既不会见到 inline static constexpr 这种写法，也不会见到 constexpr 这种写法。

一种常用的地方是替代宏，通过 inline constexpr 去定义一些常量。

另一种常用的地方是借助 Concepts 实现 Parametric Polymorphism时，通过 inline constexpr 定义一些模板变量来对类型进行边界检查。

如果不是 file scope，此时 static constexpr 就不能省，在 class scope 下反而应该省略 inline。

通过本次分享大家应该能够熟练地组合使用 inline、static、constexpr 了。
