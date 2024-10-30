## 实现的功能

- 实现了完整的系统调用spawn，用来创建一个新进程。

- 另外实现了带优先级的stride调度算法。



## 问答作业

> *stride 算法原理非常简单，但是有一个比较大的问题。例如两个 pass = 10 的进程，使用 8bit 无符号整形储存 stride， p1.stride = 255, p2.stride = 250，在 p2 执行一个时间片后，理论上下一次应该 p1 执行。*

- **实际情况是轮到 p1 执行吗？为什么？**

在 p2 执行一个时间片后，它的 stride 值会增加，变成 250 + 10 = 260。此时，p1 的 stride 值依然是 255。根据 stride 算法的规则，下一次选择执行的进程应该是 stride 值较小的进程。



>  我们之前要求进程优先级 >= 2 其实就是为了解决这个问题。可以证明， **在不考虑溢出的情况下** , 在进程优先级全部 >= 2 的情况下，如果严格按照算法执行，那么 STRIDE_MAX – STRIDE_MIN <= BigStride / 2。

- **为什么？尝试简单说明（不要求严格证明）。**

每次执行后，stride 的增量是固定的，且进程的优先级越高，其 stride 值的增长越快。所以，最高和最低的 stride 值之间的差距（STRIDE_MAX – STRIDE_MIN）不会超过大增量的一半。



- **已知以上结论，考虑溢出的情况下，可以为 Stride 设计特别的比较器，让 BinaryHeap<Stride> 的 pop 方法能返回真正最小的 Stride。补全下列代码中的 `partial_cmp` 函数，假设两个 Stride 永远不会相等。**

```
use core::cmp::Ordering;

struct Stride(u64);

impl PartialOrd for Stride {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        const MAX_STRIDE: u64 = 256; // 8-bit 无符号整形的最大值

        // 获取有效值，考虑到溢出
        let self_adjusted = self.0 % MAX_STRIDE;
        let other_adjusted = other.0 % MAX_STRIDE;

        // 进行比较
        if self_adjusted < other_adjusted {
            Some(Ordering::Less)
        } else {
            Some(Ordering::Greater)
        }
    }
}

impl PartialEq for Stride {
    fn eq(&self, other: &Self) -> bool {
        false // 假设永远不相等
    }
}

```



## 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 **以下各位** 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

   > 无

2. 此外，我也参考了 **以下资料** ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

   > 《rCore-Camp-Guide-2024A 文档》

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。

