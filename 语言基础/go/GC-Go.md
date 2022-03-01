# GC-Go语言

> 推荐阅读：
>
> [粗线条话GC（三）](https://mp.weixin.qq.com/s?__biz=Mzg5NjIwNzIxNQ==&mid=2247484560&idx=2&sn=904e5b8cf262f1a127a9d9550776cc8d&chksm=c005d490f7725d86db7df38c3dcaa563b44b945ec26751efa477166a997ad153748c3c85ec73&scene=21#wechat_redirect)

Go语言垃圾回收采用标记–清扫算法，支持主体并发增量式回收，使用插入与删除两者写屏障结合的混合写屏障。

## 基本情况

> 三种 GC 模式

Golang中垃圾回收支持三种模式：

1. `gcBackgroundMode`，默认模式，标记与清扫过程都是并发执行的；
2. `gcForceMode`，只在清扫阶段支持并发；
3. `gcForceBlockMode`，GC全程需要STW；

```go
const (
    gcBackgroundMode gcMode = iota // concurrent GC and sweep
    gcForceMode                    // stop-the-world GC now, concurrent sweep
    gcForceBlockMode               // stop-the-world GC now and STW sweep (forced by user)
)
```

> 两个全局变量

关于 `GC` 执行过程，有两个重要的全局变量：`gcController` 和 `work`。

1. `gcController` 主要用于支持标记工作顺利执行

```go

var gcController gcControllerState
type gcControllerState struct {
    scanWork int64
    bgScanCredit int64
    assistTime int64
    dedicatedMarkTime int64
    fractionalMarkTime int64
    idleMarkTime int64
    markStartTime int64

    dedicatedMarkWorkersNeeded int64
    fractionalUtilizationGoal float64
    ......
}
```

`gcController` 会记录一个 `mark cycle` (标记周期)中不同类型的 `mark worker` 是否还需要启动，是否需要进行 `assist mark`（辅助标记），已经执行了多少扫描工作，以及不同类型的 `mark worker` 分别执行了多长时间等信息。

2. `work` 用于存储全局工作信息

```go

var work struct {
    full  lfstack          // lock-free list of full blocks workbuf
    ......
    bytesMarked uint64

    markrootNext uint32 // next markroot job
    markrootJobs uint32 // number of markroot jobs

    nFlushCacheRoots                               int
    nDataRoots, nBSSRoots, nSpanRoots, nStackRoots int

    markDoneSema uint32

    bgMarkReady note   // signal background mark worker has started
    bgMarkDone  uint32 // cas to 1 when at a background mark completion point
    mode gcMode
    ......
}
```

`work` 提供全局工作队列缓存，并记录栈、数据段等需要扫描的 `root` 节点的相关信息；还会记录当前是第几个 `GC cycle`，当前 `GC cycle` 已经标记了多少字节，已经 `STW` 了多长时间，以及控制 `GC` 向下一阶段过度的信息等等。

## 并发 GC 过程

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoedswHQTMxribGq5yNCQNGyqxI2UUwLdI5Ag9FxncDK8FMRsV0ibOClibaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

默认的 `gcBackgroundMode` （默认模式）下 `GC` 执行的大致过程如下：

1. **Mark Setup**
   * 完成上一轮 `GC` 未完成的清扫工作
   * 为每个 P 创建一个 `mark worker` 协程，陷入休眠，等待标记阶段得到调度（`findRunnableGCWorker`）
2. **第一次 STW**
   * 开启写屏障
   * 开启一个新一轮 GC，`gcphase` 置为 `_GCMark`，记录 `GC` 阶段标识；
   * `writeBarrier.enabled` 置为 `true`，开启写屏障，`gcBlackenEnabled` 置为 1 表示允许 `GC` 进行标记工作。
   * 在 `work` 中记录 `bss` 段，数据段，栈中那些 `root` 节点的必要信息，为 `root` 节点标记工作做准备。
3. **Start The World，进入并发标记阶段。**
   * 后台 `mark worker` 得到调度执行时，会根据 `gcController` 中记录的相关信息决定 `worker` 的类型。
   * 不管什么类型的 `worker` 都会先执行未完成的 `root` 标记工作，扫描协程栈时，只会暂停对应协程，通过 `stacmap` 标记扫描，结束后再将其恢复。
   * `root` 标记工作完成后,需要继续追踪的 `root` 节点已经被记录到工作队列中，后台 `mark worker` 会继续处理工作队列中的节点，它们就是所谓的灰色节点。通过灰色节点可能发现更多灰色节点加入工作队列，处理完的灰色节点成为黑色节点。
   * 当没有 `root` 标记任务与灰色节点时，`GC` 就可以进入 **Mark Termination** 阶段了。
4. **第二次 STW**
   * `gcphase` 置为 `_GCMarkTermination`
   * 停止后台 `mark worker` 和 `assist worker`
   * `gcBlackenEnabled` 置为 0
   * `gphase` 置为 `_GCOff`
   * 关闭写屏障，`writeBarrier.enabled` 置为 `false`
5. **Start The World，进入清扫阶段**
   * 进入 `_GCOff` 阶段以后，再新分配的对象就是白色的了。
   * `runtime.main` 在程序初始化时会创建用于清扫的协程 `bgsweep`，存储在全局变量 `sweep` 中，到清扫阶段这个后台的 `sweeper` 会被加入到 `run queue` 中，它得到调度执行时会执行清扫任务。因为清扫工作也是增量进行的，每一轮 GC 开始之前，都要确保完成上一轮 GC 未完成的清扫工作。

## 如何应对碎片化内存

我们知道 GC 应用的是**标记-清扫算法**的垃圾回收器，会存在一个问题，不可避免会造成内存碎片化。

* 内存碎片化会增加内存分配的负担
* 降低内存使用率
* 要找到大小合适的内存块的代价会因碎片化而增加。

解决这一问题的办法主要是使用**多个链表**，不同链表管理不同大小的内存块。

因为 `mutator` 通常不会频繁申请大块内存，所以多链表管理的内存块规格主要面向中小分块，既可以满足大部分内存分配需求，又避免维护大块空闲链表而压迫到内存。

`Go` 语言的内存管理是基于 `TCMalloc` (Thread-Caching Malloc) 模型设计的，`TCMalloc` 是一种典型的分级、多链表内存管理模型，可以很好的应对碎片化内存。

> **Golang 内存管理大致结构**

内存管理模块负责向操作系统申请内存并管理起来。

`Golang` 的内存管理分为三级：

* mheap
* mcentral
* mcache

`mheap` 管理着虚拟地址空间中一大段连续的内存，我们之前说的堆分配，就是在这里分配。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoeXm9nSTUG5qNjtdoxCUx7BDqEXDTyOHCFica8Uj0XRPNiaSewc7UpV8dA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

多个 `page` 组成一个 `span` ，多个 `span` 组成一个 `arena`。

`span` 对应的数据结构是 `mspan`，每个 `span` 都只存储一种大小的元素，类型规格记录在 `mspan.spanClass`中，类型规格覆盖了小于等于 32K 的 66 种大小，类型编号 1~66。大于 32K 的大对象直接在`mheap` 中分配，对应 `mspan` 的类型编号为 0，这样一共有 67 种。

同时 `spanClass` 还记录着该 `span` 存储元素是否含有指针，含有指针的属于 `scan` 类型，不含有指针的属于 `no-scan` 类型，所以总的来说 `span` 分类一共有 134 种。对于 `no-scan` 类型的 `mspan`，`GC` 并不关心。

`mheap.central` 提供全局 `span` 缓存，它按照 `spanClass` 类型区分共 134 个 `mcentral`。每个 `mcentral` 管理一种 `spanclass` 的 `mspan`，并且会将有空闲空间和没有空闲空间的 `mspan` 分别管理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L61XQqlgUVvtaOa97dMJSDORXAyjQ18482ticf0Cg7CgciadbdiaZqQEXpy1tzsMhbU9SkVkT19bU75A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



每个 `P` 都有一个 `mcache` 用作本地 `span` 缓存，与 `mcentral` 一样，每种规格类型对应 `scan` 和 `no-scan` 两个链表。小对象分配时先从本地 `mcache` 中获取，没有的话就去 `mcentral` 获取并设置到 `P`，`mcentral` 中也没有的话，会向 `mheap` 申请。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoea9coXmmy8DAR4ic6Pa7EW4tSOYTt20VicMMdQrd49ib2ZkicdgwTEorx0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 扫描和标记相关元数据

1. `root` 节点扫描

`Go` 语言在编译阶段会生成 `bss` 段，数据段等对应的元数据，存储在可执行文件中，通过对应自己的 `modualdata`，根据其中存储的 `gcdatamask`、`gcbssmask` 等信息可以确定特定 `root` 节点是否需要添加到工作队列中。

协程栈的元数据存储在 `stackmap` 中，扫描协程栈时，通过对应元数据可以知道栈上的**局部变量、参数、返回值**等对象中哪些是存活的指针。

确定了 `root` 节点是否为指针，还要再进一步判断这些节点是否指向堆内存。

2. 堆扫描与标记

`mheap` 中每个 `arena` 对应一个 `HeapArena`，记录 `arena` 的**元数据信息**。

介绍一下`HeapArena` 中`bitmap` 和 `spans` 字段。

`bitmap` 中一个 `byte` 可以标记 `arena` 中连续四个指针大小的内存。对于每一个 `word` 来说，对应了两个 `bit`，

* 低位 `bit` 用于标记是否为指针，0 为非指针，1 为指针；
* 高位 `bit` 用于标记是否要继续扫描，高位 `bit` 为 1 就代表扫描完当前 `word` 并不能完成当前数据对象的扫描。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoeIY5pQgaIRlBC7icGubQk2O3hZYFQwmtHMucY0zWQVFKm7KicicrmtlbNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`spans` 是一个 `*mspan` 类型的数组，用于记录当前 `arena` 中每一页对应到哪一个 `mspan`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoeMBD3XOrWuOp41OCs4oXDW0xB6q7gZQXJV2JSfEtr9WHJc9HXSDGbeg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

所以，我们可以基于 `HeapArena` 记录的元数据信息，通过一个对象的地址，就可以根据 `HeapArena.bitmap` 信息扫描它内部是否含有指针，也可以根据对象地址计算出它在哪一页，进而查到存在哪一个 `mspan` 中

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoerEBNIk4ylMZ9dqsmOqboQ1bUm18Kem0Z9991aYcuHWEp2eLhZ7QPpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而每个 `span` 都对应两个位图标记：`mspan.allocBits` 和 `mspan.gcmarkBits`。

* `allocBits` 中每一位用于标记一个对象存储单元是否已分配。
* `gcmarkBits` 中每一位用于标记一个对象是否存活。

> **Golang中GC的三色标记**

* 灰色：`gcmarkBits` 置为 1 并加入工作队列中
* 黑色：`gcmarkBits` 置为 1
* 白色：`gcMarkBits` 置为 0

## 并发标志的分工问题与写屏障记录集竞争问题

每个 `P` 都有一个本地工作队列（`p.gcw`）和一个写屏障缓冲（`p.wbBuf`）。

本地工作队列（`p.gcw`）里面有两个 `workbuf`：分别是 `wbuf1` 和 `wbuf2`。工作过程如下：

* 添加任务从 `wbuf1` 添加
* 当 `wbuf1` 满了时候，就会交换 `wbuf1` 和 `wbuf2` 。
* 如果交换之后还是满的，那么就会把当前的 `wbuf1` 的工作 `flush` 到全局工作缓存中。

`mark worker` 执行 `GC` 标记工作消耗工作队列时,会处理本地工作队列和全局工作缓存中工作量的均衡问题（`runtime.gcDrain`和`runtime.gcDrainN`中）

* 如果全局工作缓存为空，就把当前的 `p` 的工作分一些到全局工作队列中。
  * 如果 `wbuf2` 不为空，那么就将 `wbuf2` 整个 `flush` 到全局工作缓存中
  * 如果 `wbuf2` 为空，那么考虑 `wbuf1`，如果 `wbuf1` 元素个数大于 4 ，就把 `wbuf1` 一半工作放到全局工作缓存中。
* 如果本地工作队列为空，就会从全局工作缓存中获取任务放到本地队列中。

当 `mutator` 触发写屏障时并不会直接操作工作队列，而是把相关指针写入当前 `p` 的写屏障缓冲区(`p.wbBuf`)中。当 `wbBuf` 已满或 `mark worker` 通过工作队列获取不到任务时,会把写屏障缓冲内容 `flush` 到工作缓存中,

通过这样**区分本地工作队列与全局工作缓存，并为每个 P 设置了写屏障缓冲区**，缓解了执行并发标记工作时操作工作队列的竞争问题。

## GC 对 CPU 的使用率

`GC` 默认的 `CPU` 的使用率为 25%。在 `GC` 执行的初始化阶段，会根据当前 `CPU` 核数 X 以 `CPU` 目标使用率来计算需要启动的 `mark worker` 数量。 

但是会出现：`6*25%=1.5` 计算结果不为整数的情况，为了应对此情况，会对该结果进行 `rounding`（+0.5）。

但是这样的 `rounding` 会和目标使用率出现显著偏差（>0.3），所以在 `mark worker` 中引入了不同的工作模式:

* **`Dedicated`** 模式的 `worker` 会执行标记任务直到被抢占；
* **Fractional** 模式的 `worker` 除了被抢占外，还可以在达到目标使用率时主动让出。

🌰：

如果有4个核，经过计算 `4*25%=1`，需要启动一个 `Dedicated` 模式的 `worker`。

如果有6个核，经过计算 `6*25%=1.5`，`rounding` 之后等于2，但是误差 `2/1.5-1=0.33`，大于0.3，所以启动一个 `Dedicated` 模式的 `worker` 和一个 `Fractional` 模式的 `worker` 来辅助完成额外的目标。

·调度器执行 `findRunnableGcWorker` 恢复 `mark worker` 时，需要设置 `worker` 运行的模式：

* 如果 `Dedicated` 模式的 `worker` 数目没有达到上限，就设置为 `Dedicated` 模式。
* 接下来就要看是否需要 `Fractional` 模式的 `worker` 辅助工作，需要的话设置为 `Fractional` 模式

P会记录自己执行 `Fractional` 模式的 `worker` 的时间，如果当前 `P` 执行 `Fractional` 模式的时间与本轮标记工作已经执行的时间的比率达到 `fractionalUtilizationGoal`，`Fractional` 模式的 `worker` 就可以主动让出了。

通过以上方式有效控制 `GC` 的 `CPU` 使用率。

## 并发GC如何缓解内存分配压力？

为了避免 `GC` 执行过程中，内存分配压力过大，还实现了 `GC Assist` 机制，包括“辅助标记”和“辅助清扫”。

辅助标价：

如果协程在 `GC` 标记工作没有完成，想要分配内存，他就要负担一定的标记工作，申请的内存越大，对应负担的标记任务就越多，这是一种借贷偿还的机制。

当 `mark worker` 完成一定量的标记任务就会在全局 `gcController` 存一笔信用，有债务需要偿还，可以从`gcController`这里 `steal` 尽量多的信用来抵消自己欠下的债务。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibjI8pEWI9L4C74nN8hPUQ0bRKdzicldoeYQa1gicrhBS0tInnkXqNzeaTWc2bnSkS2gKQLIliaqw3eSJFZqyEwmEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在清扫阶段内存分配可能会触发“辅助清扫”。

🌰：

* 直接从 `mheap` 分配大对象时，为了维持内存分配量与清扫页面数的线性关系，可能需要执行一定量的清扫工作。
* 从本地缓存中直接分配一个 `span` 时，若存在尚未清扫的可用 `span`，也需要先清扫这个 `span` 再分配使用

