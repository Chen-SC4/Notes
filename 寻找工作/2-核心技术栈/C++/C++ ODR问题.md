# C++ ODR 问题

## 永远不要在头文件中定义全局变量

全局变量的定义会导致 `ODR（One Definition Rule）`问题，即一个变量在多个编译单元中被定义，下面是一个经典例子：

```plaintext
文件结构
main.cpp
Global.h
PrintGlobal.h
PrintGlobal.cpp
```

`Global.h`:

```cpp
#pragma once
static int global = 0;
```

`PrintGlobal.h`:

```cpp
#pragma once
void PrintGlobal();
```

`PrintGlobal.cpp`:

```cpp
#include "PrintGlobal.h"
#include "Global.h"
#include <iostream>
void PrintGlobal() {
	::global = 100;
	std::cout << global << std::endl;
}
```

`main.cpp`:

```cpp
#include "PrintGlobal.h"
#include "Global.h"
int main() {
	PrintGlobal(); // 100
	std::cout << global << std::endl; // 0
	return 0;
}
```

最终 `main` 函数的输出是 `100` 和 `0`，这是因为 `global` 在 `PrintGlobal.cpp` 和 `main.cpp` 中被定义了两次。

如果 `global` 不是静态变量，那么会在编译期报错：`global` 重复定义。而如果 `gloabl` 是静态变量，就会出现这种前后值不一致的情况。

## 正确方式

全局变量应该在头文件中声明，在 `.cpp` 文件中定义：

`Global.h`：

```cpp
#pragma once
extern int global;
```

`Global.cpp`：

```cpp
#include "Global.h"
static int global = 0;
```

这样 `main` 函数的输出就会是 `100` 和 `100`。
