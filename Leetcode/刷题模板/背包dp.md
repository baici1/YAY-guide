# DP背包问题模板秒杀

## 题单

| 题目                                                         | 题解                                                         | 难度 |       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- | ----- |
| [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/perfect-squares/solution/gong-shui-san-xie-xiang-jie-wan-quan-bei-nqes/) | 中等 | 🤩🤩🤩🤩  |
| [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/coin-change/solution/dong-tai-gui-hua-bei-bao-wen-ti-zhan-zai-3265/) | 中等 | 🤩🤩🤩🤩  |
| [416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/partition-equal-subset-sum/solution/gong-shui-san-xie-bei-bao-wen-ti-xia-con-mr8a/) | 中等 | 🤩🤩🤩🤩🤩 |
| [474. 一和零](https://leetcode-cn.com/problems/ones-and-zeroes/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/ones-and-zeroes/solution/gong-shui-san-xie-xiang-jie-ru-he-zhuan-174wv/) | 中等 | 🤩🤩🤩🤩🤩 |
| [494. 目标和](https://leetcode-cn.com/problems/target-sum/)  | [LeetCode 题解链接](https://leetcode-cn.com/problems/target-sum/solution/gong-shui-san-xie-yi-ti-si-jie-dfs-ji-yi-et5b/) | 中等 | 🤩🤩🤩🤩  |
| [518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/coin-change-2/solution/gong-shui-san-xie-xiang-jie-wan-quan-bei-6hxv/) | 中等 | 🤩🤩🤩🤩  |
| [879. 盈利计划](https://leetcode-cn.com/problems/profitable-schemes/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/profitable-schemes/solution/gong-shui-san-xie-te-shu-duo-wei-fei-yon-7su9/) | 困难 | 🤩🤩🤩🤩🤩 |
| [1049. 最后一块石头的重量 II](https://leetcode-cn.com/problems/last-stone-weight-ii/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/last-stone-weight-ii/solution/gong-shui-san-xie-xiang-jie-wei-he-neng-jgxik/) | 中等 | 🤩🤩🤩🤩  |
| [1155. 掷骰子的N种方法](https://leetcode-cn.com/problems/number-of-dice-rolls-with-target-sum/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/number-of-dice-rolls-with-target-sum/solution/dong-tai-gui-hua-bei-bao-wen-ti-yun-yong-axtf/) | 中等 | 🤩🤩🤩🤩  |
| [1449. 数位成本和为目标值的最大数字](https://leetcode-cn.com/problems/form-largest-integer-with-digits-that-add-up-to-target/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/form-largest-integer-with-digits-that-add-up-to-target/solution/gong-shui-san-xie-fen-liang-bu-kao-lu-we-uy4y/) | 困难 | 🤩🤩🤩🤩  |
| [1995. 统计特殊四元组](https://leetcode-cn.com/problems/count-special-quadruplets/) | [LeetCode 题解链接](https://leetcode-cn.com/problems/count-special-quadruplets/solution/gong-shui-san-xie-yi-ti-si-jie-mei-ju-ha-gmhv/) | 简单 | 🤩🤩🤩🤩  |

> 注：来自三叶姐姐的题单！！！无敌！！

## 前置知识的学习

> 注：首先你得明白基本的背包问题，你也需要将上诉的题单刷一遍。，才能读懂以下模板的总结。

推荐学习：

* 宫水三叶的刷题日记
* 代码随想录

我们先穷尽背包是哪些问题：

* 01背包问题：在物品堆中对同一个物品取一次
* 完全背包问题：在物品堆中对同一个物品可以取多次
* 分组背包问题：在物品堆中对同一个物品会有多个重量选择，但是选择一次
* 多维背包问题：背包容量具有多个维度

以上四个问题下面都会有多个分类小问题，我们一一解决。

## 01背包

### 最值问题

1. 首先对题目进行分析，确定 01背包五大要素所对应的值。
   * 物品数量
   * 物品重量
   * 背包容量
   * 所求的是什么
   * 物品价值
2. 根据题目要求进行初始化。
3. 遍历顺序：使用滚动数组：**外层遍历物品数量，内层倒序遍历背包容量** ，普通dp则都是正序的。
3. 得出递推公式：`dp[j] = max/min(dp[j], dp[j-weight[i]]+value[i])` 一般情况。特殊情况需要分析。
4. 根据题目处理得到的价值，返回结果。

模板如下：

```go
//物品重量：weight[i]
//物品价值：value[i]
//物品数量：len(weight)
//背包容量：bagWeight
//求背包里面物品最大价值
func test_1_wei_bag_problem(weight, value []int, bagWeight int) int {
 // dp的定义：当背包容量为 i 时候最大价值，所以需要 bagWeight+1。
 dp := make([]int,bagWeight+1)
    //dp初始化，在这里可能觉得无意义，但是后续的题目会很重要。
    dp[0]=0
 // 外层遍历物品的重量。
 for i := 0 ;i < len(weight) ; i++ {
        //内层遍历背包容量。注：倒序是为了防止一个物品多次加入。
  for j:= bagWeight; j >= weight[i] ; j-- {
   // dp的递推公式
   dp[j] = max(dp[j], dp[j-weight[i]]+value[i])
  }
 }
 //fmt.Println(dp)
 return dp[bagWeight]
}

func max(a,b int) int {
 if a > b {
  return a
 }
 return b
}
```

> 注：找到 `01` 背包最值问题的五要素是解题的关键，很多题目的五要素需要经过分析与处理得知。

#### 解决问题

[416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

根据题目的分析可知：

* 物品数量：`nums` 的元素个数
* 物品重量：`nums[i]`
* 背包容量：`nums` 和的二分之一
* 求获得的背包最大价值，
* 物品的价值与物品重量一致。

```go
func canPartition(nums []int) bool {
    //价值与重量等价都是 nums[i] 物品数量为 len(nums) 背包容量为sum/2
    sum:=0
    for _,v:=range nums{
        sum+=v
    }
    if sum%2==1{
        return false
    }
    sum/=2
    dp:=make([]int,sum+1)
    //遍历物品数量
    for i:=0;i<len(nums);i++{
        //倒序遍历背包容量，防止重复加入
        for j:=sum;j>=nums[i];j--{
            dp[j]=max(dp[j],dp[j-nums[i]]+nums[i])
        }
    }
    return dp[sum]==sum
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

[1049. 最后一块石头的重量 II](https://leetcode-cn.com/problems/last-stone-weight-ii/)

此题与上面题目解法类似。提示：**将一个石头堆分成尽可能的等重量地两堆石头。**

* 物品数量：`stones` 的元素个数
* 物品重量：`stones[i]`
* 背包容量：`stones` 和的二分之一
* 01背包最值问题
* 物品的价值与物品重量一致。

```go
func lastStoneWeightII(stones []int) int {
    //与分割等和子集类似 01背包求价值 价值是 sum/2
    sum:=0
    for _,v:=range stones{
        sum+=v
    }
    num:=sum
    sum/=2
    dp:=make([]int,sum+1)
    for i:=0;i<len(stones);i++{
        for j:=sum;j>=stones[i];j--{
            dp[j]=max(dp[j],dp[j-stones[i]]+stones[i])
        }
    }
    return num-dp[sum]*2
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

### 组合问题

1. 四要素：

* 物品数量
* 物品重量
* 背包容量
* 所求的是什么

2. 初始化：`dp[0]=1` 这里指一维，如果是多维也需要将 `dp[0]…[0]=1`
3. 遍历顺序：**外层遍历物品数量，内层倒序遍历背包容量。** 注：内层遍历时倒序
4. 递推公式：根据求组合数，所以是 `dp[j]+=dp[j-nums[i]]` 不懂的需要去看完整的教程！
5. 处理结果值，返回答案。

模板如下：

```go
//物品重量：weight[i]
//物品价值：value[i]
//物品数量：len(weight)
//背包容量：bagWeight
//求组合数
func test_1_wei_bag_problem(weight, value []int, bagWeight int) int {
 // dp的定义：当背包容量为 i 时候最大价值，所以需要 bagWeight+1。
 dp := make([]int,bagWeight+1)
    //dp初始化，
    dp[0]=1
 // 外层遍历物品的重量。
 for i := 0 ;i < len(weight) ; i++ {
        //内层遍历背包容量。注：倒序是为了防止一个物品多次加入。
  for j:= bagWeight; j >=weight[i]; j--{
   // dp的递推公式
   dp[j]+=dp[j-weight[i]]
  }
 }
 //fmt.Println(dp)
 return dp[bagWeight]
}

```

> 注：对于求组合数，价值一般都是不考虑，都是与重量一致，所以是四要素。

#### 解决问题

 [494. 目标和](https://leetcode-cn.com/problems/target-sum/)

此题需要经过处理才可以使用背包。根据以下两个公式可知需要求的背包容量：

* `left-right=target`
* `right+left=sum`

五要素分别是：

* 物品数量：len(nums)
* 物品重量：nums[i]
* 背包容量：left
* 所求的是什么：01背包组合数

```go
func findTargetSumWays(nums []int, target int) int {
    //01背包求组合数
    sum:=0
    for _,v:=range nums{
        sum+=v
    }
    //left-right=target
    //right+left=sum
    if (sum+target)%2!=0||abs(target)>sum{
        return 0
    }
    left:=(sum+target)/2
    dp:=make([]int,left+1)
    dp[0]=1
    //01背包求组合数,遍历物品数量，遍历背包容量
    for i:=0;i<len(nums);i++{
        for j:=left;j>=nums[i];j--{
            dp[j]+=dp[j-nums[i]]
        }
    }
    return dp[left]
}
func abs(a int)int{
    if a>0{
        return a
    }
    return -a
}
```

### 路径还原问题

当遇到01背包输出满足最大价值路径时候，这时候不能使用滚动数组，需要采用普通的dp来处理。

> 注：滚动数组最后获得的是最后状态的dp，不能够回溯之前的状态还原路径。

模板：

```go
func test_2_wei_bag_problem1(weight, value []int, bagweight int) int {
 // 定义dp数组
 dp := make([][]int, len(weight))
 for i, _ := range dp {
  dp[i] = make([]int, bagweight+1)
 }
 // 初始化
 for j := bagweight; j >= weight[0]; j-- {
  dp[0][j] = dp[0][j-weight[0]] + value[0]
 }
 // 递推公式
 for i := 1; i < len(weight); i++ {
  //正序,也可以倒序
  for  j := weight[i];j<= bagweight ; j++ {
   dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i])
  }  
 }
    //还原路径
    path:=make([]int,len(weight))
    j:=bagweight
    for i:=len(weight);i>0;i--{
        u:=weight[i]
        if j>=u&&dp[i][j]==dp[i-1][j-u]+i{
            path[i]=1
            j-=u
        }
    }
    if j!=0{
        path[0]=1
    }
 return dp[len(weight)-1][bagweight]
}

func max(a,b int) int {
 if a > b {
  return a
 }
 return b
}

```

#### 解决问题

[6029. 射箭比赛中的最大得分](https://leetcode-cn.com/problems/maximum-points-in-an-archery-competition/)

代码如下：

```go
func maximumBobPoints(numArrows int, aliceArrows []int) []int {
    //物品数量 12
    //物品重量是aliceArrows[i]+1
    //物品价值 是 i
    //01背包
    //背包容量 numArrows
    dp:=make([][]int,12)
    for i:=0;i<12;i++{
        dp[i]=make([]int,numArrows+1)
    }
    //初始化
    for i:=aliceArrows[0]+1;i<=numArrows;i++{
        dp[0][i]=0
    }
    for i:=1;i<12;i++{
        for j:=0;j<=numArrows;j++{
            if j<aliceArrows[i]+1{
                dp[i][j]=dp[i-1][j]
            }else {
                dp[i][j]=max(dp[i-1][j],dp[i-1][j-aliceArrows[i]-1]+i)
            }
          
        }
    }
    bob:=make([]int,12)
    fmt.Println(dp)
    //遍历
    j:=numArrows
    for i:=11;i>0;i--{
        u:=aliceArrows[i]+1
        if j>=u&&dp[i][j]==dp[max(i-1,0)][j-u]+i{
            bob[i]=u
            j-=u
        }
    }
    
   if j!=0{
       bob[0]=j
   }
    return bob
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

## 完全背包

### 最值问题

1. 找到五要素：

* 物品数量
* 物品重量
* 背包容量
* 物品价值
* 所求的是什么

2. 进行初始化，这得看题目要求。
3. 遍历顺序：使用滚动数组：**外层遍历物品数量，内层正序遍历背包容量** 。注：内层遍历顺序与01背包相反。
4. 得出递推公式：`dp[j] = max/min(dp[j], dp[j-weight[i]]+value[i])`
5. 根据题意处理结果并返回。

模板如下：

```go
// test_CompletePack1 先遍历物品, 在遍历背包
func test_CompletePack1(weight, value []int, bagWeight int) int {
 // 定义dp数组 和初始化
 dp := make([]int, bagWeight+1)
 // 遍历顺序
 for i := 0; i < len(weight); i++ {
  // 正序会多次添加 value[i]
  for j := weight[i]; j <= bagWeight; j++ {
   // 推导公式
   dp[j] = max(dp[j], dp[j-weight[i]]+value[i])
   // debug
   //fmt.Println(dp)
  }
 }
 return dp[bagWeight]
}
```

> 注：使用滚动数组的遍历顺序需要与01背包区别开来，至于理由可以去看[代码随想录](https://programmercarl.com/%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80%E5%AE%8C%E5%85%A8%E8%83%8C%E5%8C%85.html#%E5%85%B6%E4%BB%96%E8%AF%AD%E8%A8%80%E7%89%88%E6%9C%AC)

#### 解决问题

 [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

此题五元素如下：

* 物品数量：小于n的完全平方数个数
* 物品重量：完全平方数
* 背包容量：n
* 物品价值：1
* 所求的是什么：完全背包求个数。

```go
func numSquares(n int) int {
    dp:=make([]int,n+1)
    //完全背包
    //物品数量
    for i:=1;i<=n;i++{
        dp[i]=math.MaxInt32
    }
    for i:=1;i*i<=n;i++{
        for j:=i*i;j<=n;j++{
            dp[j]=min(dp[j],dp[j-i*i]+1)
        }
    }
    return dp[n]
}
func min(a,b int)int{
    if a>b{
        return b
    }
    return a
}
```

 [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

此题五元素如下：

* 物品数量：硬币数量
* 物品重量：硬币对应的面额
* 背包容量：amount
* 物品价值：1
* 所求的是什么：完全背包求个数

```go
func coinChange(coins []int, amount int) int {
    dp:=make([]int,amount+1)
    for i:=0;i<=amount;i++{
        dp[i]=math.MaxInt32
    }
    dp[0]=0
    for i:=0;i<len(coins);i++{
        for j:=coins[i];j<=amount;j++{
            dp[j]=min(dp[j],dp[j-coins[i]]+1)
        }
    }
    if dp[amount]==math.MaxInt32{
        return -1
    }
    return dp[amount]
}
func min(a,b int)int{
    if a>b{
        return b
    }
    return a
}
```

### 组合问题

1. 四要素：

* 物品数量
* 物品重量
* 背包容量
* 所求的是什么

2. 初始化：`dp[0]=1` 这里指一维，如果是多维也需要将 `dp[0]…[0]=1`
3. 遍历顺序：**外层遍历物品数量，内层正序遍历背包容量。** 注：内层遍历是正序。
4. 递推公式：根据求组合数，所以是 `dp[j]+=dp[j-nums[i]]` 不懂的需要去看完整的教程！
5. 处理结果值，返回答案。

```go
//物品重量：weight[i]
//物品价值：value[i]
//物品数量：len(weight)
//背包容量：bagWeight
//求组合数
func test_1_wei_bag_problem(weight, value []int, bagWeight int) int {
 // dp的定义：当背包容量为 i 时候最大价值，所以需要 bagWeight+1。
 dp := make([]int,bagWeight+1)
    //dp初始化，
    dp[0]=1
 // 外层遍历物品的重量。
 for i := 0 ;i < len(weight) ; i++ {
        //内层遍历背包容量。注：倒序是为了防止一个物品多次加入。
  for j:= weight[i]; j <=bagWeight; j++{
   // dp的递推公式
   dp[j]+=dp[j-weight[i]]
  }
 }
 //fmt.Println(dp)
 return dp[bagWeight]
}
```

#### 解决问题

 [518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/)

* 物品数量：硬币数量
* 物品重量：硬币对应的面额
* 背包容量：amount
* 所求的是什么：完全背包求组合数

```go
func change(amount int, coins []int) int {
    //求的是组合数，现在的组合数，加上以前能够满足要求的组合数，
    dp:=make([]int,amount+1)//dp的定义是当金额是n的最大的组合数。
    dp[0]=1
    //外层for循环遍历物品，内层for遍历背包。
    for i:=0;i<len(coins);i++{
        for j:=coins[i];j<=amount;j++{
            dp[j]+=dp[j-coins[i]]
        }
    }
    return dp[amount]
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

### 排列问题

1. 四要素：

* 物品数量
* 物品重量
* 背包容量
* 所求的是什么

2. 初始化：`dp[0]=1` 这里指一维，如果是多维也需要将 `dp[0]…[0]=1`
3. 遍历顺序：**外层遍历背包容量，内层遍历物品数量。**注：这里遍历顺序与组合问题相反。
4. 递推公式：根据求组合数，所以是 `dp[j]+=dp[j-nums[i]]` 不懂的需要去看完整的教程！
5. 处理结果值，返回答案。

#### 解决问题

 [377. 组合总和 Ⅳ](https://leetcode-cn.com/problems/combination-sum-iv/)

* 物品数量：数组元素个数
* 物品重量：数组元素值
* 背包容量：target
* 所求的是什么：完全背包求排列数

```go
func combinationSum4(nums []int, target int) int {
    dp:=make([]int,target+1)
    dp[0]=1
    //外层遍历背包容量
    for i:=0;i<=target;i++{
        //内层遍历物品数量
        for j:=0;j<len(nums);j++{
            if i>=nums[j]{
                dp[i]+=dp[i-nums[j]]
            }
        }
    }
    return dp[target]
}
```

### 路径还原问题

## 分组背包

一个物品组里面最多选择一个物品。

这里相当于将之前物品进行二维的升级处理。根本上的思路其实与普通01背包是一致的。

dp数组维度推荐是二维 `dp[i][j]`。

* i：代表第 i 个物品组
* j：代表当前背包重量。

注：虽然可以降维但是对于此题来讲**可读性优先。**同时降维意义不大，不能够减少时间复杂度。

遍历顺序：

* 外层遍历物品组数量。
* 第二层遍历背包容量。
* 内层遍历物品组内部的物品。

注：其实内部是正序还是倒序，对于二维一般性 dp数组来说可以不需要考虑，都选择正序，但是如果你选择一维滚动数组，需要考虑 01背包不能重复拿的问题。根据这一点我推荐你选择二维。

递推公式：

* 最值问题：`dp[i][j] = Math.max(dp[i][j], dp[i - 1][j - vi[k]] + wi[k]);`
* 组合数问题：`dp[i][j]+=dp[i-1][j-k]`

关于初始化：需要根据题目来进行处理。

### 解决问题

 [1155. 掷骰子的N种方法](https://leetcode-cn.com/problems/number-of-dice-rolls-with-target-sum/)

直接贴出代码：

```go
func numRollsToTarget(n int, k int, target int) int {
    const mod int=1e9+7
    dp:=make([][]int,n+1)
    for i:=0;i<=n;i++{
        dp[i]=make([]int,target+1)
    }
    dp[0][0]=1
    for i:=1;i<=n;i++{
        for j:=k;j>=1;j--{
            for m:=target;m>=j;m--{
                dp[i][m]+=dp[i-1][m-j]
                dp[i][m]%=mod
            }
        }
    }
    return dp[n][target]
}
```

## 多维背包

其实分组背包是将物品进行多维化，也可以对背包进行多维化。

在处理层面是与 01 背包 /完全背包其实一样的，只是对背包维度加一了。

所以在这里解法你可以参考 01背包与完全背包，遇到了多维背包，你所做的就是将背包维度+1.

接下来的题目将直接给出解法。

### 解决问题

 [474. 一和零](https://leetcode-cn.com/problems/ones-and-zeroes/)

多维背包求最值问题

```go
type str struct{
    zero int
    one int
}
func findMaxForm(strs []string, m int, n int) int {
    //01背包问题
    nums:=[]str{}
    for i:=0;i<len(strs);i++{
        num0:=0
        num1:=0
        for j:=0;j<len(strs[i]);j++{
            if strs[i][j]=='0'{
                num0++
            }else {
                num1++
            }
        }
        nums=append(nums,str{
            num0,
            num1,
        })
    }
   // fmt.Println(nums)
    dp:=make([][]int,m+1)
    for i:=0;i<=m;i++{
        dp[i]=make([]int,n+1)
    }
    //遍历物品数量
    for i:=0;i<len(nums);i++{
        //遍历背包容量，背包由两种组成
        for j:=m;j>=nums[i].zero;j--{
            for k:=n;k>=nums[i].one;k--{
                dp[j][k]=max(dp[j][k],dp[j-nums[i].zero][k-nums[i].one]+1)
            }
        }
    }
  //  fmt.Println(dp)
    return dp[m][n]
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

[1995. 统计特殊四元组](https://leetcode-cn.com/problems/count-special-quadruplets/)

多维01背包求组合数问题

```go
func countQuadruplets(nums []int) int {
    dp:=[101][4]int{}
    dp[0][0]=1
    ans:=0
    for i:=0;i<len(nums);i++{
        ans+=dp[nums[i]][3]
        for j:=100;j>=nums[i];j--{
            for k:=3;k>=1;k--{
                dp[j][k]+=dp[j-nums[i]][k-1]
            }
        }
    }
    
    return ans
}
```

## 特殊题

题目本质上是背包问题，但是由于题意需要修改一下递推公式或者夹着其他解题思想。

#### 解决问题

[1449. 数位成本和为目标值的最大数字](https://leetcode-cn.com/problems/form-largest-integer-with-digits-that-add-up-to-target/)

**此题你需要通过 完全背包问题获得字符串的最大长度，然后通过贪心思想回溯获得动规路径。**

```go
func largestNumber(cost []int, target int) string {
    //这是一道综合题，不再是单纯地背包dp题目。
    //根据dp求出最长地长度
    dp:=make([]int,target+1)
    for i:=0;i<=target;i++{
        dp[i]=math.MinInt32
    }
    dp[0]=0
    //完全背包
    for i:=1;i<=9;i++{
        u:=cost[i-1]
        for j:=u;j<=target;j++{
            dp[j]=max(dp[j],dp[j-u]+1)
        }
    }
    fmt.Println(dp)
    //回溯路径获得规划路径
    ans:=[]byte{}
    j:=target
    for i:=9;i>=1;i--{
        u:=cost[i-1]
        for j>=u&&dp[j]==dp[j-u]+1{
            ans=append(ans,byte(i+'0'))
            j-=u
        }
    }
    return string(ans)
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

[879. 盈利计划](https://leetcode-cn.com/problems/profitable-schemes/)

此题与之前遇到的最值问题不同，之前是不超过，这题则是不小于，那么dp 递推公式需要有所改变。

同时此题也是一种多维背包。

```go
func profitableSchemes(n int, minProfit int, group []int, profit []int) int {
    const mod int=1e9+7
    //物品数量是group，求组合数，是01背包，价值是profit 重量是group[i],背包容量是人员n与minProfit
    dp:=make([][]int,n+1)
    for i:=0;i<=n;i++{
        dp[i]=make([]int,minProfit+1)
    }
    for i:=0;i<=n;i++{
        dp[i][0]=1
    }
    for index,g:=range group{
        for i:=n;i>=g;i--{
            for j:=minProfit;j>=0;j--{
                dp[i][j]+=dp[i-group[index]][max(0,j-profit[index])]
                dp[i][j]%=mod
            }
        }
    }
    return dp[n][minProfit]
}
func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

## 总结

到这里，我们已经解决了题单上的背包问题，我们做一个总结。

首先，你需要找到题目的中背包五元素。

当你遇到是01背包，使用滚动数组遍历顺序：**外层遍历物品数量，内层倒序遍历背包容量。**

当你遇到是完全背包，使用滚动数组遍历顺序：**外层遍历物品数量，内层正序遍历背包容量。**

明白了背包类型，接下来考虑题目要求：

最值问题：递推公式：`dp[j] = max(dp[j], dp[j-weight[i]]+value[i])`

组合问题：递推公式：`dp[j]+=dp[j-weight[i]]`

初始化则根据题目类型进行考虑：

一般最值问题初始化：`dp[0]=0`

组合问题初始化：`dp[0]=1`

特殊点：

当你遇到完全背包排列问题，遍历顺序是**外层遍历背包容量，内层遍历物品数量。**

分组背包与多维背包推荐采用二维DP数组，其余参考传统背包解法。

**希望你能够通过这几道题目秒杀背包问题。**😋😋😋
