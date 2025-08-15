# Pointer to Implementation (PIMPL) 设计模式

在开发中，常常出现下面的情况，例如对于 `Widget.hpp` 文件：

```C++
#include "Component.hpp"
#include "Person.hpp"
#include "Car.hpp"

class Widget {
private:
	Component component;
	Person person;
	Car car;
	// ...
};
```

上面这个 `Widget.hpp` 主要有两个问题：

1. 依赖过多，如果 `Component.hpp`、`Person.hpp`、`Car.hpp` 中有修改，`Component.hpp` 也需要重新编译，增加了编译时间。如果 `Component.hpp` 是一个很大的库的话，编译时间会很长，头文件中应该尽可能少的包含其他头文件。
2. 暴露了很多细节，例如 `Component`、`Person`、`Car` 的实现细节，我们可能不希望暴露 `Widget` 的实现细节。

为了解决上面的问题，可以使用 PIMPL 设计模式，将 `Widget` 的实现细节封装在一个独立的类中，例如 `WidgetImpl`。

```C++
// ----------------------------------------------------Wdiget.hpp
#pragma once

#include <memory>
#include <string>

class Widget {
private:
    struct Impl;
    std::unique_ptr<Impl> pImpl;

public:
    void printAge() const;
    Widget();
    // 析构函数必须在头文件中声明，而非定义
    ~Widget();
};
// ----------------------------------------------------Wdiget.cpp
#include "widget.hpp"
#include "Component.hpp"
#include "Person.hpp"
#include "Car.hpp"

#include <iostream>

struct Widget::Impl {
	Component component;
	Person person;
	Car car;
	// ...
};

void Widget::printAge() const {
    std::cout << "Age: " << pImpl->person.getAge() << std::endl;
}

Widget::Widget() : pImpl(std::make_unique<Impl>(12)) {}

Widget::~Widget() = default;
```

可以看到在 `widget.hpp` 中不需要包含 `person.hpp`，只需要在 `widget.cpp` 中包含即可。这样可以减少编译时间，同时也隐藏了 `Widget` 的实现细节。

需要注意的是，如果使用的是 `unique_ptr<Impl> pImpl;` 则需要在 `Widget` 的头文件中定义 `~Widget()`，否则会报错。这是因为 `unique_ptr` 会在析构时调用 `delete`，而 `delete` 需要知道 `Impl` 的大小，所以如果不在 `.hpp` 中声明析构函数，编译器就会默认在 `.hpp` 中定义析构函数，此时析构函数就需要知道 `Impl` 的大小，从而报错。

如果使用 `shared_ptr<Impl> pImpl;` 则不需要在 `Widget` 的析构函数中定义 `~Widget()`。
