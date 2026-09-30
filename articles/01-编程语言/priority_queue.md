---
title: priority_queue：随时取出最值的那只手
author: TUST-ACM 算法实验室
date: 2026-09-17
category: 01-编程语言
tags: [C++, STL, priority_queue, 堆, 贪心]
level: 入门
status: 已完成
---

# priority_queue：随时取出最值的那只手

> 读完你应该能：声明大根堆和小根堆；会 `push` / `top` / `pop` 且不在空堆上取顶；用它写出合并果子；知道「懒删除」和「堆不能随便改中间那个元素」这两条边界。
>
> 一句口号：**`set` 能找任意键，`priority_queue` 只会把最值递到你鼻子底下——但递得更快、代码更短。**

## 背景 / 为什么要读这篇

[贪心入门](../03-算法竞赛/贪心入门.md) 里合并果子的策略是：每次选最小的两堆合在一起。若每回对数组 `sort`，n = 1e4 还能活，n = 1e5 时反复排序是 `O(n² log n)` 的慢性死亡。

你需要的结构只有一句话：**动态维护一批数，每次拿出最小（或最大）的。** 这就是堆。C++ 里现成的叫 `priority_queue`。

[map 与 set](map与set.md) 也能维护最值（`*s.begin()`），还能删除任意值、找后继。堆更专一：不支持高效「删除指定值」，换来更小的常数和更直白的语义。合并果子、Huffman、Dijkstra 的入门实现，都从这只手开始。

实验室里有人把 `priority_queue` 默认当成小根堆，因为「priority 听起来应该先服务弱者」。STL 默认是**大根堆**：`top()` 是最大的。每年因此 WA 的合并果子可以开一桌。

## 正文

### 1. 默认大根堆，小根堆要自己写类型

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    priority_queue<int> mx;                        // 大根堆
    priority_queue<int, vector<int>, greater<int>> mn;  // 小根堆

    mx.push(3);
    mx.push(1);
    mx.push(5);
    cout << mx.top() << '\n';  // 5
    mx.pop();
    cout << mx.top() << '\n';  // 3

    mn.push(3);
    mn.push(1);
    mn.push(5);
    cout << mn.top() << '\n';  // 1
    return 0;
}
```

模板参数三个位置：

| 位置 | 含义 | 默认 |
| --- | --- | --- |
| 第 1 个 | 元素类型 | 必填 |
| 第 2 个 | 底层容器 | `vector<T>` |
| 第 3 个 | 比较器 | `less<T>` → 大根堆 |

`greater<int>` 把「更小的视为优先级更高」，于是 `top()` 是最小。口诀：**默认要最大；要最小就 `greater`。** 比较器写反，合并果子的答案会变成「每次合并最大两堆」，数字大得像天文。

常用操作：

| 操作 | 复杂度 | 注意 |
| --- | --- | --- |
| `push(x)` | O(log n) | |
| `top()` | O(1) | **空堆未定义** |
| `pop()` | O(log n) | 只删顶，不返回值 |
| `size()` / `empty()` | O(1) | `pop` 前先 `empty()` |

没有 `push_back`，没有下标，没有迭代器遍历「第 3 大」。你只能看见堆顶。想遍历全部，只能不断 `pop` 到另一个容器里——那会拆掉堆本身。需要有序遍历用 `set` 或先倒进 `vector` 再 `sort`。

### 2. 空堆、拷贝、以及 `pop` 不返回值

C 语言里有人怀念 `x = heap_extract()`。C++ 是：

```cpp
int x = pq.top();
pq.pop();
```

两行。合成一行的宏没有必要。空堆上 `top()` / `pop()` 是未定义行为：本地可能崩，评测机可能给你一个「看起来像答案」的垃圾数。循环里写成：

```cpp
while (!pq.empty()) {
    int x = pq.top();
    pq.pop();
    // ...
}
```

`priority_queue` 没有 `find`，没有 `erase(x)`。这不是缺陷，是产品定位。

### 3. 合并果子：小根堆的第一道必做题

洛谷 [P1090 合并果子](https://www.luogu.com.cn/problem/P1090)：n 堆果子，每次合并两堆，花费为两堆重量之和，合并结果作为新堆放回去。求最小总花费。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    priority_queue<long long, vector<long long>, greater<long long>> pq;
    for (int i = 0; i < n; i++) {
        long long x;
        cin >> x;
        pq.push(x);
    }

    long long ans = 0;
    while (pq.size() > 1) {
        long long a = pq.top(); pq.pop();
        long long b = pq.top(); pq.pop();
        ans += a + b;
        pq.push(a + b);
    }
    cout << ans << '\n';
    return 0;
}
```

细节：

- **`long long`。** 重量累加很容易超 `int`。
- **循环条件 `size() > 1`。** 剩一堆时没有两堆可合，那就是最终堆。n = 1 时答案是 0（没合并过）。
- **先取两个再 `push` 回去。** 不要取一个就急着加进 `ans`。
- 这就是 Huffman 树的花费：权重大的尽量放浅，所以每次合最小的。贪心正确性见上一篇的「每次取最值」。

### 4. 大根堆的日常：前 k 大、中位数的一半

「n 个数里前 k 大」若 k 很小：维护一个 **大小为 k 的小根堆**，堆顶是「当前第 k 大」。新来的数比堆顶大就踢掉堆顶再塞进去。最后堆里就是前 k 大，堆顶是第 k 大。

```cpp
priority_queue<int, vector<int>, greater<int>> pq;  // 小根堆，装最大的 k 个
for (int x : a) {
    pq.push(x);
    if ((int)pq.size() > k) pq.pop();
}
// pq.top() 是第 k 大（1-based 的 k）
```

对偶：前 k 小用大小为 k 的大根堆。和 [二分](../03-算法竞赛/lower_bound与二分.md) 里「第 k 小」不同：那里数组静态可 `sort`；这里可以是流式、边读边问。

两个堆对顶可以动态维护中位数：左边大根堆、右边小根堆，保持左边个数等于右边或比右边多 1。入门知道有这招即可，实现留到需要时再写，平衡细节比合并果子烦。

### 5. pair 进堆：Dijkstra 预习

```cpp
using pli = pair<long long, int>;  // {距离, 点编号}
priority_queue<pli, vector<pli>, greater<pli>> pq;
pq.push({0, s});
```

`pair` 先比 `first` 再比 `second`。`greater` 后 `{距离小的}` 在堆顶。最短路里距离会变，**堆里可能躺着过期的 `{旧距离, 点}`**。标准做法不是改堆中间（改不了），而是：弹出时若 `d != dist[u]` 就丢弃。这叫**懒删除**——过期条目像没扫地的面包屑，看见再扔。

入门题很少需要懒删除；合并果子不需要。看见 Dijkstra / 「带减少 key 的堆」时再回来。不要为了删一个中间元素去手写二叉堆的 `decrease-key`，赛时容易写炸。

### 6. 和 set / 数组怎么选

| 需求 | 用什么 |
| --- | --- |
| 只要反复取最小或最大，插入即可 | `priority_queue` |
| 还要删除任意值、查是否存在、后继 | `set` / `multiset` |
| 一次性全部读完，只要全局第 k | `nth_element` 或 `sort` |
| 合并果子、Huffman、K 路归并 | 小根堆 |
| 图论最短路（非负边） | 小根堆 + 懒删除 |

`multiset` 能当堆用（`begin()` 是最小），还能 `erase` 指定值。常数通常比 `priority_queue` 大。能只用堆顶就不要上 `set`。

### 7. 自定义结构体

```cpp
struct Node {
    int u, w;
    bool operator<(const Node& o) const {
        return w < o.w;  // 注意：priority_queue 用 < 时，w 小的「优先级更低」，top 是 w 大的
    }
};
priority_queue<Node> pq;
```

这里最绕：`priority_queue` 的比较器和 `sort` 的直觉相反——`<` 定义的是「谁该沉底」。写错就堆顶反了。入门**优先 `pair` + `greater`**，少写 `operator<`。非写不可时，用小数据打印 `top()` 验证一次。

## 常见坑 / 注意事项

- **默认当成小根堆。** 合并果子必须 `greater`。
- **空堆 `top()`。** n=1 的合并果子、while 条件写成 `!empty()` 却一次 `pop` 两个。
- **`pop()` 当返回值。** 它是 `void`。
- **`int` 装合并花费。** 见果子。
- **在堆里改已经 `push` 进去的对象。** 改不到。懒删除或重新 `push` 一份新的。
- **`priority_queue<int> pq = a;` 从 vector 初始化。** 可以，构造函数接受容器；之后仍是堆。不要假设底层 `vector` 下标有序。
- **遍历 `for (auto x : pq)`。** 没有这样的接口。
- **比较器写成 `greater<>` 却元素是 `pair`，想按 `second` 排序。** 默认仍按 `first`。要按第二维，自己包一层或把关键字放 `first`。

## 趣味课间：实验室经典翻车集

- **「我 push 了负数，堆坏了。」** 没有。比较器用 `<` 对负数同样成立。坏的是你以为堆只装自然数。
- **「Python 的 heapq 默认最小堆。」** 对。C++ 默认最大。跨国抄板子先看国籍。
- **「我用 `sort` 模拟堆，样例过了。」** n 小的样例是共犯。合并果子数据范围就是来查这个的。
- **两个 `top` 之间忘了 `pop`。** 取了两次同一个最小，第三堆在旁边冷笑。
- **Dijkstra 不过期丢弃。** 同一点进堆十几次，复杂度从 `O((n+m)log n)` 滑向「能过样例的随机算法」。

## 小结

堆是「动态最值」的专用口袋。默认最大，`greater` 后最小；`top`/`pop` 前看空；合并果子是小根堆的成人礼。能离线 `sort` 就离线；要边插边取最值，再请它来。

建议练习：

- 洛谷 [P1090 合并果子](https://www.luogu.com.cn/problem/P1090)：本篇主菜
- 洛谷 [P1631 序列合并](https://www.luogu.com.cn/problem/P1631)：堆维护「当前最小的和」
- 洛谷 [P3378 【模板】堆](https://www.luogu.com.cn/problem/P3378)：对照手写与 STL
- 手推 `{1,2,9,3,8}` 的合并果子花费，再跑程序

图论向下一篇：[BFS：网格上的最短路](../03-算法竞赛/BFS网格最短路.md)。结构体怎么进堆： [lambda 与自定义排序](lambda与自定义排序.md)。

## 参考资料

- [贪心入门](../03-算法竞赛/贪心入门.md)
- [map 与 set](map与set.md)
- [lambda 与自定义排序](lambda与自定义排序.md)
- [cppreference：std::priority_queue](https://en.cppreference.com/w/cpp/container/priority_queue)
- [OI Wiki：堆](https://oi-wiki.org/ds/binary-heap/)
- [OI Wiki：Huffman](https://oi-wiki.org/misc/huffman-tree/)
