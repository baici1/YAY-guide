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

gcController 会记录
