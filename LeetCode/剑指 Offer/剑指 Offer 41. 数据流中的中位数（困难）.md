### 题目描述

这是 LeetCode 上的 **[剑指 Offer 41. 数据流中的中位数](https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/solution/by-ac_oier-exn5/)** ，难度为 **困难**。

Tag : 「优先队列（堆）」



中位数是有序列表中间的数。如果列表长度是偶数，中位数则是中间两个数的平均值。

例如，

* `[2,3,4]` 的中位数是 $3$
* `[2,3]` 的中位数是 $(2 + 3) / 2 = 2.5$

设计一个支持以下两种操作的数据结构：
* `void addNum(int num)` - 从数据流中添加一个整数到数据结构中。
* `double findMedian()` - 返回目前所有元素的中位数。

示例：
```
addNum(1)
addNum(2)
findMedian() -> 1.5
addNum(3) 
findMedian() -> 2
```

进阶:
* 如果数据流中所有整数都在 $0$ 到 $100$ 范围内，你将如何优化你的算法？
* 如果数据流中 `99%` 的整数都在 $0$ 到 $100$ 范围内，你将如何优化你的算法？

---

### 优先队列（堆）

这是一道经典的数据结构运用题。

具体的，**我们可以使用两个优先队列（堆）来维护整个数据流数据，令维护数据流左半边数据的优先队列（堆）为 `l`，维护数据流右半边数据的优先队列（堆）为 `r`**。

显然，**为了可以在 $O(1)$ 的复杂度内取得当前中位数，我们应当令 `l` 为大根堆，`r` 为小根堆，并人为固定 `l` 和 `r` 之前存在如下的大小关系**：

* **当数据流元素数量为偶数：`l` 和 `r` 大小相同，此时动态中位数为两者堆顶元素的平均值；**
* **当数据流元素数量为奇数：`l` 比 `r` 多一，此时动态中位数为 `l` 的堆顶原数。**

为了满足上述说的奇偶性堆大小关系，在进行 `addNum` 时，我们应当分情况处理：

* 插入前两者大小相同，说明插入前数据流元素个数为偶数，插入后变为奇数。我们期望操作完达到「`l` 的数量为 `r` 多一，同时双堆维持有序」，进一步分情况讨论：
    * 如果 `r` 为空，说明当前插入的是首个元素，直接添加到 `l` 即可；
    * 如果 `r` 不为空，且 `num <= r.peek()`，说明 `num` 的插入位置不会在后半部分（不会在 `r` 中），直接加到 `l` 即可；
    * 如果 `r` 不为空，且 `num > r.peek()`，说明 `num` 的插入位置在后半部分，此时将 `r` 的堆顶元素放到 `l` 中，再把 `num` 放到 `r`（相当于从 `r` 中置换一位出来放到 `l` 中）。

* 插入前两者大小不同，说明前数据流元素个数为奇数，插入后变为偶数。我们期望操作完达到「`l` 和 `r` 数量相等，同时双堆维持有序」，进一步分情况讨论（此时 `l` 必然比 `r` 元素多一）：
    * 如果 `num >= l.peek()`，说明 `num` 的插入位置不会在前半部分（不会在 `l` 中），直接添加到 `r` 即可。
    * 如果 `num < l.peek()`，说明 `num` 的插入位置在前半部分，此时将 `l` 的堆顶元素放到 `r` 中，再把 `num` 放入 `l` 中（相等于从 `l` 中替换一位出来当到 `r` 中）。

代码：
```Java
class MedianFinder {
    PriorityQueue<Integer> l = new PriorityQueue<>((a,b)->b-a);
    PriorityQueue<Integer> r = new PriorityQueue<>((a,b)->a-b);
    public void addNum(int num) {
        int s1 = l.size(), s2 = r.size();
        if (s1 == s2) {
            if (r.isEmpty() || num <= r.peek()) {
                l.add(num);
            } else {
                l.add(r.poll());
                r.add(num);
            }
        } else {
            if (l.peek() <= num) {
                r.add(num);
            } else {
                r.add(l.poll());
                l.add(num);
            }
        }
    }
    public double findMedian() {
        int s1 = l.size(), s2 = r.size();
        if (s1 == s2) {
            return (l.peek() + r.peek()) / 2.0;
        } else {
            return l.peek();
        }
    }
}
```
* 时间复杂度：`addNum` 函数的复杂度为 $O(\log{n})$；`findMedian` 函数的复杂度为 $O(1)$
* 空间复杂度：$O(n)$

---

### 进阶

* 如果数据流中所有整数都在 $0$ 到 $100$ 范围内，你将如何优化你的算法？

可以使用建立长度为 $101$ 的桶，每个桶分别统计每个数的出现次数，同时记录数据流中总的元素数量，每次查找中位数时，先计算出中位数是第几位，从前往后扫描所有的桶得到答案。

**这种做法相比于对顶堆做法，计算量上没有优势，更多的是空间上的优化。**

**对顶堆解法两个操作中耗时操作复杂度为 $O(\log{n})$，$\log$ 操作常数不会超过 $3$，在极限数据 $10^7$ 情况下计算量仍然低于耗时操作复杂度为 $O(C)$（$C$ 固定为 $101$）桶计数解法。**

* 如果数据流中 `99%` 的整数都在 $0$ 到 $100$ 范围内，你将如何优化你的算法？

和上一问解法类似，对于 $1$% 采用哨兵机制进行解决即可，在常规的最小桶和最大桶两侧分别维护一个有序序列，即建立一个代表负无穷和正无穷的桶。

上述两个进阶问题的代码如下，但注意由于真实样例的数据分布不是进阶所描述的那样（不是绝大多数都在 $[0,100]$ 范围内），所以会 `TLE`。

代码：
```Java
class MedianFinder {
    TreeMap<Integer, Integer> head = new TreeMap<>(), tail = new TreeMap<>();
    int[] arr = new int[101];
    int a, b, c;
    public void addNum(int num) {
        if (num >= 0 && num <= 100) {
            arr[num]++;
            b++;
        } else if (num < 0) {
            head.put(num, head.getOrDefault(num, 0) + 1);
            a++;
        } else if (num > 100) {
            tail.put(num, tail.getOrDefault(num, 0) + 1);
            c++;
        }
    }
    public double findMedian() {
        int size = a + b + c;
        if (size % 2 == 0) return (find(size / 2) + find(size / 2 + 1)) / 2.0;
        return find(size / 2 + 1);
    }
    int find(int n) {
        if (n <= a) {
            for (int num : head.keySet()) {
                n -= head.get(num);
                if (n <= 0) return num; 
            }
        } else if (n <= a + b) {
            n -= a;
            for (int i = 0; i <= 100; i++) {
                n -= arr[i];
                if (n <= 0) return i;
            }
        } else {
            n -= a + b;
            for (int num : tail.keySet()) {
                n -= tail.get(num);
                if (n <= 0) return num;
            }
        }
        return -1; // never
    }
}
```

---

### 最后

这是我们「刷穿 LeetCode」系列文章的第 `剑指 Offer 41` 篇，系列开始于 2021/01/01，截止于起始日 LeetCode 上共有 1916 道题目，部分是有锁题，我们将先把所有不带锁的题目刷完。

在这个系列文章里面，除了讲解解题思路以外，还会尽可能给出最为简洁的代码。如果涉及通解还会相应的代码模板。

为了方便各位同学能够电脑上进行调试和提交代码，我建立了相关的仓库：https://github.com/SharingSource/LogicStack-LeetCode 。

在仓库地址里，你可以看到系列文章的题解链接、系列文章的相应代码、LeetCode 原题链接和其他优选题解。

