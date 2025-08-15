# Java 基础

## 1. BIO、NIO 和 AIO

### BIO

传统的阻塞 I/O 模式，调用方在发起 I/O 操作时会被阻塞，直到操作完成后才继续执行。

- 优点
	- 实现简单
- 缺点
	- 在高并发场景下，性能较差

BIO 适用于连接数较少，逻辑比较简单的场景。

### NIO

NIO 是为了解决 BIO 的一些缺点而引入的。它的核心是“非阻塞”和“多路复用”。非阻塞 I/O 模式，调用方在发起 I/O 操作后可以立刻返回。通常会与 I/O 多路复用技术 (Select、Poll、Epoll) 结合使用，使得一个线程可以同时管理多个连接。

- 优点
	- 效率较高
- 缺点
	- 实现复杂

### AIO

AIO 是在 NIO 的基础上进一步发展的，它的核心是“异步”。异步 I/O 模式，调用方在发起 I/O 操作后可以立刻返回，并且在操作完成后会通过回调函数通知调用方。

- 优点
	- 理论上性能更高
- 缺点
	- 实现复杂
	- 操作系统不一定支持 `AIO`

## HashMap 和 HashSet

- HashSet 是基于 `HashMap` 实现的集合类。它不允许重复元素，用于存储一组无序的唯一元素。
- HashMap 是基于哈希表实现的键值对集合类。它允许使用 `null` 作为键或值，并且不保证元素的顺序。**键**必须是唯一的，而**值**可以重复。

HashSet 的表现与 HashMap 的键的表现一致，事实上，HashSet 内部就使用了 HashMap 来存储元素。HashSet 中的每个元素都作为 HashMap 的键，而值则是一个没什么意义的常量对象。

## ConcurrentHashMap 和 Hashtable

ConcurrentHashMap 和 Hashtable 都是线程安全的。

### Hashtable

Hashtable 通过在几乎所有的公共方法上使用 `synchronized` 关键字来实现线程安全。它的性能较差，因为每次访问都需要获取锁。在新的 Java 版本中，Hashtable 已经不推荐使用。

### ConcurrentHashMap

ConcurrentHashMap 通过分段锁 (CAS + 局部锁) 的方式来实现线程安全。它将整个 HashMap 分成多个段，每个段都有自己的锁。这样可以提高并发性能，因为多个线程可以同时访问不同的段。在高并发的场景下 ConcurrentHashMap 的性能要优于 Hashtable。

## 3. Stream

