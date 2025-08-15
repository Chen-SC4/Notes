# 内存池 Memory Pool

## 整体框架

[FrameDesign.excalidraw](../../../../全局静态资源/Excalidraw/FrameDesign.excalidraw.md)

内存池的整体框架如上图所示，主要分为三个区域：`Thread Cache`、`Central Cache` 和 `Page Cache`。其中，`Thread Cache` 用于存储线程私有的内存池，`Central Cache` 用于存储共享的内存池，而 `Page Cache` 则用于存储页面级别的缓存数据。

> -  `Thread Cache` 就像是每个线程的私人小抽屉，专门存储小块内存 (小于 256KB)。它允许线程快速分配和释放内存，因为不需要锁，这意味着没有等待时间，特别适合处理小对象，比如几百字节的数据。研究表明，这种设计减少了线程间的竞争，提升了小内存分配的效率。
> - `Central Cache` 像是大家共用的储物间，当线程的个人抽屉（`Thread Cache`）用完了，线程会来这里拿更多内存。它管理较大内存块，可能是通过“span”这种结构，`span` 是一组页面。使用 `Central Cache` 时可能需要锁，因为它是共享资源，所以可能会稍微慢一些，但它确保线程能获取足够的内存。
> - `Page Cache` 像是大仓库，负责与操作系统打交道，处理大块内存分配（比如超过 1MB）。它似乎也用于为 `Central Cache` 补充内存，页面大小可能是8K，具体取决于系统设置。研究显示，它适合处理大对象或当其他缓存需要更多资源时。

## Thread Cache 基础结构

[ThreadCache.excalidraw](../../../../全局静态资源/Excalidraw/ThreadCache.excalidraw.md)

`Thread Cache` 维护多个自由列表（`Free List`），每个列表对应一种大小类别（`size-class`），大约有百种类别，管理不同大小的内存块。例如，8字节、16字节、24字节等，按一定的规则递增。

当线程需要小内存时，先从自己的 `Thread Cache` 查找对应大小的自由列表。如果列表不空，直接取用第一个对象，无需锁，分配速度极快。

当线程需要更大的内存或者 `Freelist` 为空时，它会向 `Central Cache` 请求更多内存。`Central Cache` 会将大块内存分割成适当大小的块，并将它们放入 `Thread Cache` 的自由列表中。

除此之外，`Thread Cache` 还会进行垃圾回收。当自由列表中的闲置内存块满足一定条件时，它会将其中的一部分内存归还给 `Central Cache`，以便其他线程使用。

## Central Cache 基础结构

[CentralCache.excalidraw](../../../../全局静态资源/Excalidraw/CentralCache.excalidraw.md)

`Central Cache` 是 内存池的中间层，负责管理共享的内存资源，当 `Thread Cache` 不足时为线程补充。

`Central Cache` 由一个个「桶」组成 (图中橙色的部分)，每个桶对应一个大小类别，这一点与 `Thread Cache` 类似。桶中存放一个 `Span` 构成的双向链表。

`Span` 是 `Central Cache` 中的基本单位，而 `Span` 本身的基本单位是页（`Page`），每个 `Span` 由多个页组成。页的大小通常是 4KB 或 8KB，具体取决于系统设置。`Span` 的大小类别与 `Thread Cache` 一致。

例如，对于 `8B` 这个桶来说，其中的 `Span` 的大小就是 `8KB` (假设一页为 `8KB`)，`Central Cache` 会把这个 `8KB` 的 `Span` 所有的内存块划分为 `1024` 个 `8B` 的内存块并链在一起。

`Central Cache` 也会进行垃圾回收，当 `Thread Cache` 归还内存块时，如果刚好把一个 `Span` 内的所有内存块都归还了，那么 `Central Cache` 就会把这个 `Span` 归还给 `Page Cache` 。这样可以将 `Span` 下的内存块进行重新排序，并减少内存碎片。

## Page Cache 基础结构

[PageCache.excalidraw](../../../../全局静态资源/Excalidraw/PageCache.excalidraw.md)

`Page Cache` 是内存池的底层，负责与操作系统进行交互，管理大块内存的分配和释放。

之前在 `Central Cache` 中提到过，`Span` 的基本单位是页。这里的 `Page Cache` 的结构与 `Central Cache` 类似，也是由多个桶组成，只是桶对应的不再是大小类别，而是 `Span` 中页的数量。

例如 `1Page` 的桶表示，这个桶中所保存的 `Span` 中只有一个页，`2Page` 的桶表示，这个桶中所保存的 `Span` 中有两个页，以此类推。

`Page Cache` 会与操作系统进行交互，申请和释放大块内存。它会将大块内存划分为多个 `Span`，并将这些 `Span` 放入对应的桶中。

## 申请内存的流程

为了简单起见，我们假设在一开始整个内存池都是空的。

1. 线程需要申请 `8B` 的内存块
	1. 线程首先检查自己的 `Thread Cache` 中是否有 `8B` 的内存块，如果有，直接返回。
	2. 如果没有，线程就会通过 `Central Cache` 提供的接口申请若干个 `8B` 的内存块，具体多少需要看算法的实现。
2. `Central Cache` 检查自己的 `8B` 的桶中是否有空闲的 `Span`
	1. 如果有，直接返回。
	2. 如果没有，`Central Cache` 就会向 `Page Cache` 申请一个新的 `Span`，并将这个 `Span` 中的内存块划分为多个 `8B` 的内存块。
3. `Page Cache` 检查自己的 `1Page` 的桶中是否有空闲的 `Span`
	1. 如果有，直接返回。
	2. 如果没有，`Page Cache` 会寻找比 `1Page` 大的桶中是否有空闲的 `Span`
		1. 如果有，将这个 `Span` 划分出一个 `1Page` 出来，剩下的放到对应的桶中
		2. 如果没有，`Page Cache` 会向 操作系统 申请一个新的 `128Page` 的 `Span`，然后重复检查。

可以看到，在一开始，内存池为空时，`Page Cache` 会先向操作系统申请一个很大的 `Span`，然后将这个 `Span` 按照页为单位进行划分，将得到的一个大小合适的 `Span` 交给 `Central Cache`。

`Central Cache` 会将这个 `Span` 划分为多个内存块，并根据算法，将一部分内存块交给 `Thread Cache`，剩下到放到桶中的 `Span` 下。