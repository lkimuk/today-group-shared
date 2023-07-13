## T230713 Dispatch table
本次谈逻辑分派。

三种最基本的逻辑关系为相似、承接和因果关系。

逻辑分派主要指的是因果关系，因果关系里面又包含条件关系。因果关系是一种非常特殊且重要的关系，表示两个功能块之间具备极强的依赖性。功能 B 只有依赖于功能 A 才能存在，彼此不能分开。

像简单功能模块使用的 `if-else`，`constexpr if`,   Parametric Polymorphism 当中使用的 Partial Specialization/Tag dispatching/SFINAE/Concepts，Ad hoc Polymorphism 当中使用的 overload resolution，还有 Subtyping Polymorphism 当中使用的 virtual function，都属于逻辑分派。

结构从本质上来说，是一种逻辑。程序结构本质是程序的逻辑体现，混乱的逻辑代表混乱的结构，清晰的逻辑代表清晰的结构，结构越简洁、越清晰，程序越具有稳定性，扩展维护起来也比较轻松。

早期的这些逻辑分派，大多依赖于 type，反而是最朴素的 `if-else` 依赖于 value。依赖 type 往往需要定义很多的类型，甚至很简单的功能也要定义一个类，随着代码模块增多，类会越来越多，这叫类膨胀问题。而依赖于 value 是一种更偏向于 Ad hoc Polymorphism 的方式，Functional Programming 就属于此类，此时功能的最小单元通常使用函数。

Dispatch table 就属于一种轻量级的逻辑分派方式。

一个简单的示例：

```cpp
```cpp
int main(){
  std::map<const char , std::function<double(double,double)>> dispatch_table{
    { '+', [](double a, double b) { return a + b;} },
    { '-', [](double a, double b) { return a - b;} },
    { '*', [](double a, double b) { return a * b;} },
    { '/', [](double a, double b) { return a / b;} } };

  std::print("10.24 + 4.5 = {}\n", dispatch_table['+'](10.24, 5.20));
  std::print("10.24 - 4.5 = {}\n", dispatch_table['-'](10.24, 5.20));
  std::print("10.24 * 4.5 = {}\n", dispatch_table['*'](10.24, 5.20));
  std::print("10.24 / 4.5 = {}\n", dispatch_table['/'](10.24, 5.20));

  dispatch_table['^'] = [](double a, double b) { return std::pow(a,b); };
  std::print("3.5 ^ 4.5= {}\n", dispatch_table['^'](10.24, 5.20));
};
```

Dispatch table 用到的核心特性包含 `std::map`, `std::functional`, uniform initialization, initializer list 和 lambdas。

这些偏现代的语法，使 C++ 实现这种逻辑分派也快像 python 一样简洁了。

这种逻辑分派方式往往会在网络通信中使用，网络打乱了程序原有的流程，从一个功能模块可以跳到非常远的功能模块。此时，你若是想让在这两个模块之间建立因果逻辑，传统的那些逻辑方式就多有不便。

在功能模块 A 中定义相关的标识符号，再在功能模块 B 中定义一个 Dispatch table，就能动态根据标识符号跳到相关的处理逻辑。这就实现了跨模块间的逻辑控制。这种方式不仅具备可扩展性，而且设置处理逻辑的方式非常轻便，还可随意添加、删除、复用处理逻辑。

但是真正使用的时候，不会像上述例子那样简单。我们往往会定义一个 delegate 类，封装一下再用。

下面是我在项目中使用的一个真实例子：

```cpp
namespace okec::utils 
{

template <typename IdentifierType, typename CallbackType>
class delegate {
public:
	using value_type = std::map<IdentifierType, CallbackType>;

	template<typename T>
	auto insert(T&& id, CallbackType callback) -> bool {
		return associations_.emplace(typename value_type::value_type{ std::forward<T>(id), callback }).second;
	}

	template<typename T>
	auto remove(T&& id) -> bool {
		return associations_.erase(std::forward<T>(id)) == 1;
	}

	auto clear() -> void {
		associations_.clear();
	}

	auto begin() -> typename value_type::const_iterator {
		return associations_.cbegin();
	}

	auto end() -> typename value_type::const_iterator {
		return associations_.cend();
	}

	template<typename T>
	auto find(T&& id, typename value_type::const_iterator& iter) -> bool {
		iter = associations_.find(std::forward<T>(id));
		if (iter != associations_.end())
			return true;

		return false;
	}

private:
	value_type associations_;
};

} // namespace okec::utils
```

通过封装的这一层，将具体标识类型和处理逻辑抽象，这样就能够复用这一组件。

基于这个组件，我就可以定义一个具体的消息处理类：

```cpp
namespace okec
{

template <typename CallbackType = std::function<void()>>
class message_handler {
	using delegate_type = utils::delegate<std::string_view, CallbackType>;

public:
	auto add_handler(std::string_view msg_type, CallbackType callback) -> void {
		delegate_.insert(msg_type, callback);
	}

	template <typename... Args>
	auto dispatch(const std::string& msg_type, Args... args) -> bool {
		typename delegate_type::value_type::const_iterator iter;
		bool ret = delegate_.find(msg_type, iter);
		if (ret) {
			iter->second(args...);
		}

		return ret;
	}

private:
	delegate_type delegate_;
};

} // namespace okec
```

然后在具体的通信模块中，就能够直接使用该消息处理类，直接分发处理不同的逻辑。

```cpp
class my_project {
    using callback_type = std::function<void()>;

public:

	auto read_handler(socket s) {
		while (packet = socket->recv()) {
			if (packet) {
	            auto msg_type = get_message_type(packet);
	            m_msg_handler.dispatch(msg_type);
	        }
		}
	}

	auto set_request_handler(std::string_view msg_type, callback_type callback) -> void {
		 m_msg_handler.add_handler(msg_type, callback);
	}

private:
	 message_handler<callback_type> m_msg_handler;
};
```