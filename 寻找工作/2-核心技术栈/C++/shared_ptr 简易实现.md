# std::shared_ptr\<T\> 的简易实现

## 1. 需要实现的主要功能

- 引用计数
	- `shared_ptr` 内部维护着一个 `RefCount`，当它的值为 `0` 时，则会被 `shared_ptr` 自动销毁
- 拷贝函数
	- 当发生拷贝时，`shared_ptr` 原本维护的引用计数需要减一，拷贝过来的引用计数加一
- 移动语义
	- `shared_ptr` 支持移动

## 2. 实现

### 引用计数模块

```cpp
class ControlBlock {
private:
    int RefCount;

public:
    ControlBlock() : RefCount(1) {}
    void Increment()
    {
        ++this->RefCount;
    }
    void Decrement()
    {
        --this->RefCount;
    }
    int GetRefCount() const
    {
        return this->RefCount;
    }
};
```

### 智能指针模块

```cpp
template <typename T, typename Deleter = std::default_delete<T>>
class SharedPointer {
private:
    T* Ptr;
    ControlBlock* ControlBlockPtr;
    // 将当前引用计数减一，如果引用计数减小到0，则释放资源
    void Decrement()
    {
        if (this->ControlBlockPtr) {
            this->ControlBlockPtr->Decrement();
            if (this->ControlBlockPtr->GetRefCount() == 0) {
                Deleter(this->Ptr);
                delete this->ControlBlockPtr;
            }
        }
    }

public:
	// 默认构造
    SharedPointer() : Ptr(nullptr), ControlBlockPtr(nullptr) {}
    SharedPointer(std::nullptr_t) : SharedPointer() {}
    // 有参构造 (原始指针)
    SharedPointer(T* ptr) : Ptr(ptr), ControlBlockPtr(new ControlBlock()) {}
    // 拷贝构造
    SharedPointer(const SharedPointer& other)
        : Ptr(other.Ptr), ControlBlockPtr(other.ControlBlockPtr)
    {
        if (this->ControlBlockPtr) {
            this->ControlBlockPtr->Increment();
        }
    }
    // 移动构造
    SharedPointer(SharedPointer&& other)
        : Ptr(other.Ptr), ControlBlockPtr(other.ControlBlockPtr)
    {
        other.Ptr = nullptr;
        other.ControlBlockPtr = nullptr;
    }
    // 拷贝赋值
    // 需要先把原来的引用计数 -1，然后再将现在的引用计数 +1
    SharedPointer& operator=(const SharedPointer& other)
    {
        if (this == &other) {
            return *this;
        }
        this->Decrement();
        this->Ptr = other.Ptr;
        this->ControlBlockPtr = other.ControlBlockPtr;
        if (this->ControlBlockPtr) {
            this->ControlBlockPtr->Increment();
        }
        return *this;
    }
    // 移动赋值
    // 原来的引用计数 -1，现在的引用计数不变
    SharedPointer& operator=(SharedPointer&& other)
    {
        if (this == &other) {
            return *this;
        }
        this->Decrement();
        this->Ptr = other.Ptr;
        this->ControlBlockPtr = other.ControlBlockPtr;
        other.Ptr = nullptr;
        other.ControlBlockPtr = nullptr;
        return *this;
    }
    // 重置为 nullptr
    void Reset()
    {
        this->Decrement();
        this->Ptr = nullptr;
        this->ControlBlockPtr = nullptr;
    }
    // 重置为 ptr
    void Reset(T* ptr)
    {
        this->Decrement();
        if (ptr != nullptr) {
            this->Ptr = ptr;
            this->ControlBlockPtr = new ControlBlock();
        }
    }
    // 获取引用计数
    int UseCount() const
    {
        return this->ControlBlockPtr ? this->ControlBlockPtr->GetRefCount() : 0;
    }
    // 获取原始指针
    T* Get() const
    {
        return this->Ptr;
    }
    T& operator*() const
    {
        return *this->Ptr;
    }
    T* operator->() const
    {
        return this->Ptr;
    }
    // 析构
    ~SharedPointer()
    {
        this->Decrement();
        this->Ptr = nullptr;
        this->ControlBlockPtr = nullptr;
    }
};
```