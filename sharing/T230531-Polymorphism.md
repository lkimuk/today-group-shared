## T230531 Polymorphism
有一段时间没写TGS了，继续更新。

今天谈一个基本概念——Polymorphism，这是大多数初学者最初接触的概念，要说它简单，那你可能想简单了。它是一个可以串起C++98-26发展历程的一个概念，像之前的五星「重载决议」内容，都仅仅只是这条逻辑链下的某一个小支点，它至少链接了50+条概念，非常复杂。

Polymorphism最基本的目标就是，同一功能，多种实现。

首先引出的就是Type System，因为功能需要针对不同的类型，来定制多种实现。类型一致性必须定义明确，才能在此基础上构建功能，主要存在两种定义，一种是Nominal，一种是Structural。

Nominal又叫name-based type system，这种方式以名称来定义类型一致性，打个比方，夏侯婴、夏侯惇、夏侯渊在这种类型系统中就被认为是同一种类型，具有类型一致性，那么同一种功能就可以作用到所有这种类型上去。Structural又叫property-based type system，这种方式以属性来定义类型一致性，某些人都说粤语，这是一个共同属性，那么就被视为是同一种类型，只要附带这个属性，就具备调用功能的条件。

采用Nominal的主要是Subtype Polymorphism和Ad hoc Polymorphism，而采用Structural的方式的主要是Parametric Polymorphism。

Subtype Polymorphism是Object-oriented programming中才有的，C++通过Inheritance+virtual functions来支持这种方式，根据同一继承体系的名称来定义类型一致性，只要在高层级名称中提供默认行为，其他低层级名称就可以使用这种默认行为，也可以重新改写行为，由此支持同一功能，多种实现这一目标。它的问题在于，一些第三方库可能无法派生，或是有些类型不满足is-a关系，强制其进入同一type system，是一种侵入式的行为。

因此C++产生了一种技术来解决这种缺点，就是Type Erasure。Inheritance+virtual functions的这种方式全称应该为Runtime nominative subtype polymorphism，而Type Erasure全称应该为Runtime structural subtype polymorphism。它用来在运行期模拟实现Structural这种类型一致性，从而避免了继承这种方式的缺点，标准中的std::function和std::any就是采用这种设计方式。

CRTP则是在编译期模拟Subtype Polymorphism，所以它的全称应该为Compile-time nominative subtype polymorphism。

Subtype Polymorphism主要是一种描述抽象与具体关系的概念，主要的方式是Dynamic Polymorphism，通过一些技巧，C++中也可以实现Dynamic structural subtype polymorphism和Static subtype polymorphism。

再接着是Ad hoc Polymorphism，就是通过函数重载来达到同一功能、多种实现的目标。C++支持的核心技术就是重载决议，标准提供一个默认行为，扩展行为通过重载来实现。不过标准没搞好，导致被ADL二段式折磨至今，STL引入的CPOs就是一种临时解决之法，还有tag_invoke和C++26的CFO都是在为此买单。

函数重载全称可以称为Compile-time nominative ad hoc polymorphism，模板特化也是一种Ad hoc Polymorphism，在某种程度上也可以认为是类的重载。

最后来说Parametric Polymorphism，C++通过泛型编程来支持这种方式。我们可以编写泛型函数和泛型的数据类型，于是功能不再依赖具体类型。泛型函数是函数模板，泛型类型是变量模板和类模板，使用SFINAE/Concepts来约束类型。

这种方式就是Compile-time structural parametric polymorphism，Concepts使得这种方式实现起来更加容易，对于某些属性进行约束，扩展类型只要满足该约束，就可以使用已有功能。

类型是对值的约束，Concepts是对类型的约束，所以这种方式抽象了类型，聚焦于结构，非常灵活，Ranges库大量使用了这种设计。

这么多Polymorphism方式使得C++程序设计非常灵活，当然细节也非常之多，哪种方式适合哪种情境，那就得具体问题具体分析了。有时间了接着写这个话题~