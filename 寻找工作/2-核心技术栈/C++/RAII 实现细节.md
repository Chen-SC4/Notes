# RAII 实现细节

## 1. 注意拷贝构造函数

```C++
class Resource {
public:
    Resource() {
        std::cout << "Resource acquired\n";
        this->pointer = new int[100];
    }

    Resource(const Resource &other) = delete;

    // 移动构造函数
    Resource(Resource &&other) {
        this->pointer = other.pointer;
        other.pointer = nullptr;
    }

    ~Resource() {
        if (pointer != nullptr) {
            std::cout << "Resource destroyed\n";
            delete[] pointer;
        }
    }

private:
    int *pointer;
};
```

在实现 `RAII` 时需要注意拷贝构造函数，编译器默认实现的拷贝构造函数是浅拷贝，容易出现双重释放的问题。

## 2. 记得使用智能指针 (推荐)

实现 `RAII` 需要考虑各种资源，以及移动构造/赋值、拷贝构造/赋值函数的实现，非常麻烦。要是直接在使用的时候使用智能指针 `unique_ptr` 和 `shared_ptr` 就不需要考虑这些问题了。这种方法比较灵活，既可以利用智能指针实现安全的资源管理，也可以不用智能指针进行简单的测试。

```C++
class Resource {
public:
    Resource() {
        std::cout << "Resource acquired\n";
        this->pointer = new int[100];
    }

    explicit Resource(const Resource &other) = delete;
    Resource(Resource &&other) = delete;
    Resource &operator=(const Resource &other) = delete;
    Resource &operator=(Resource &&other) = delete;

    ~Resource() {
        if (pointer != nullptr) {
            std::cout << "Resource destroyed\n";
            delete[] pointer;
        }
    }

private:
    int *pointer;
};

void foo(unique_ptr<Resource> res) {
}

int main() {
    unique_ptr<Resource> res1 = make_unique<Resource>();
    // 移动
    unique_ptr<Resource> res2 = std::move(res1);
    foo(std::move(res2));
}
```

## 3. 如果不想用智能指针

```C++
class Resource {
private:
    struct InnerResource {
        // 内部资源
        void *p;

        InnerResource() {
            cout << "Resource Acquired" << endl;
            p = new char[1024];
        }
        ~InnerResource() {
            cout << "Resource Released" << endl;
            delete[] static_cast<char *>(p);
        }
    };

    shared_ptr<InnerResource> Self;

public:
    Resource() : Self(make_shared<InnerResource>()) {}

    void Use() {
        cout << "Resource Used" << endl;
        // 使用资源
        static_cast<char *>(Self->p)[0] = 'A';
    }
};

void foo1(Resource r) {}
void foo2(Resource const &r) {}

int main() {
    Resource r;
    r.Use();
    foo1(r);
    foo2(r);
    return 0;
}
```

如果不想用智能指针，则可以在资源的外侧再套一层类，然后这个类拥有资源的 `shared_ptr`。

### 3.1 让资源支持 `Clone()`

```C++
class Resource {
private:
    struct InnerResource {
	    // .........................
        // 内部资源
        InnerResource(InnerResource const &other) {
            cout << "Resource Acquired" << endl;
            p = new char[1024];
            // 深拷贝
            p[0] = other.p[0];
        }

    };

    shared_ptr<InnerResource> Self;

    Resource(shared_ptr<InnerResource> self) {
        Self = self;
    }

public:
	// .............................
    // 深拷贝 (克隆)
    Resource Clone() const {
        return Resource(make_shared<InnerResource>(*Self));
    }
};
```

实现深拷贝的三个步骤：

1. `Clone` 函数的实现，`make_shared<InnerResource>(*Self)` 会调用 `InnerResource` 的拷贝构造函数
2. `InnerResource` 的拷贝构造函数的实现
3. 从 `shared_ptr<InnerResource>`  构造 `Resource` 的有参构造函数

### 3.2 声明与定义分开实现

上面的实现都是在同一个文件中，那么如何将声明与定义分开实现呢？

`resource.hpp`:

```C++
#include <memory>

class Resource {
private:
    struct InnerResource;

    std::shared_ptr<InnerResource> Self;

    Resource(std::shared_ptr<InnerResource> self);

public:
    Resource();
    ~Resource();

    void Use();

    // 深拷贝 (克隆)
    Resource Clone() const;
};
```

`resource.cpp`:

```C++
#include "resource.hpp"
#include <cstdio>
#include <iostream>

struct Resource::InnerResource {
    // 内部资源
    char *p;

    InnerResource() {
        std::cout << "Resource Acquired" << std::endl;
        p = new char[1024];
    }

    InnerResource(InnerResource const &other) {
        std::cout << "Resource Acquired" << std::endl;
        p = new char[1024];
        // 深拷贝
        p[0] = other.p[0];
    }

    ~InnerResource() {
        std::cout << "Resource Released" << std::endl;
        delete[] static_cast<char *>(p);
    }
};

Resource::Resource(std::shared_ptr<InnerResource> self) {
    Self = self;
}

Resource::Resource() : Self(std::make_shared<InnerResource>()) {}

Resource::~Resource() = default;

void Resource::Use() {
    // 使用资源
    printf("Resource Used: %p\n", Self->p);
}

// 深拷贝 (克隆)
Resource Resource::Clone() const {
    return Resource(std::make_shared<InnerResource>(*Self));
}

```

在上面的 `hpp` 的文件中，运用到了 `PIMPL` 设计模式，将 `InnerResource` 的声明放在了 `Resource` 的声明中。在 `cpp` 文件中实现了 `InnerResource` 的定义。关于 `PIMPL` 设计模式可以参考 [PIMPL 设计模式 - C++ 特有的设计模式](PIMPL%20设计模式%20-%20C++%20特有的设计模式.md)。

