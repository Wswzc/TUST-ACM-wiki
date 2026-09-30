---
title: map 与 set：有序的键，以及「这个数出现过吗」
author: TUST-ACM 算法实验室
date: 2026-09-16
category: 01-编程语言
tags: [C++, STL, map, set, 哈希, 竞赛]
level: 入门
status: 已完成
---

# map 与 set：有序的键，以及「这个数出现过吗」

> 读完你应该能：用 `set` 判重、取最小 / 最大、找后继；用 `map` 计数、给离散值编号；知道它们内部是树，单次 `O(log n)`，以及什么时候该换成 `unordered_*` 或排好的 `vector`。
>
> 一句口号：**`set` 是只有键的有序集合，`map` 是带值的有序字典。都不要当数组下标用 1e9。**

## 背景 / 为什么要读这篇

[vector 与 sort](vector与sort.md) 解决「一堆数排好」；[二分](../03-算法竞赛/lower_bound与二分.md) 解决「排好之后怎么查」。但比赛里还有更赖皮的需求：

- 数不断插进来、删掉，还要随时问最小值
- 「这个字符串见过几次」
- 值域是 1e9，不能开 `int cnt[1000000001]`

数组下标必须是密密麻麻的整数。`map` / `set` 的键可以是任意能比较的东西：`int`、`long long`、`string`、`pair`。代价是每次操作 `O(log n)`，以及常数比数组大。

实验室经典事故：`int a[1000000000]` 本地直接爆，或评测 MLE；改成 `map<int,int>` 之后 AC，于是有人从此一切都用 `map`。请不要变成那个人——静态、排得完的序列，[lower_bound](../03-算法竞赛/lower_bound与二分.md) 更快。

## 正文

### 1. 先认清四个亲戚

| 容器 | 键有序？ | 允许重复键？ | 存什么 |
| --- | --- | --- | --- |
| `set<T>` | 是 | 否 | 只有键 |
| `multiset<T>` | 是 | 是 | 只有键 |
| `map<K,V>` | 按 K 有序 | 否（每个 K 一个 V） | 键值对 |
| `multimap<K,V>` | 按 K 有序 | 是 | 键值对 |
| `unordered_set` / `unordered_map` | 否（哈希） | 否 | 平均 O(1)，最坏会被卡 |

入门先打穿 `set` 和 `map`。`unordered_*` 在 Codeforces 上有时被精心构造的数据卡成 O(n²)，竞赛里要么自定义 hash，要么继续用树。**TUST-ACM 算法实验室入门训练默认 `map` / `set`，需要常数再换。**

### 2. set：口袋里的有序不重复数

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    set<int> s;
    s.insert(3);
    s.insert(1);
    s.insert(3);          // 无效，还是两个元素
    s.erase(3);

    if (s.count(1)) {     // 0 或 1
        // 有 1
    }
    auto it = s.find(1);
    if (it != s.end()) {
        // *it == 1
    }

    if (!s.empty()) {
        int mn = *s.begin();          // 最小
        int mx = *s.rbegin();         // 最大
    }
    return 0;
}
```

迭代器：`set` 的迭代器像链表，**不支持 `it + k`**，只支持 `++` / `--`。找第 k 小不要指望 `s.begin()+k`，那是 `vector` 的特权。第 k 小要么树状数组，要么政策性地「数据范围允许就丢进 vector 再 sort」。

`lower_bound` 也可以对 `set` 用：

```cpp
auto it = s.lower_bound(x);  // 第一个 ≥ x
if (it != s.end()) { /* *it */ }
if (it != s.begin()) {
    auto pr = prev(it);      // 前驱，注意 begin 不能 --
}
```

后继、前驱是平衡树题的入门动作。手写 `--it` 前必须保证 `it != s.begin()`，否则未定义，和二分解引用 `end()` 是同一类地雷。

**删除迭代器 vs 删除值：**

```cpp
s.erase(x);    // 删掉值为 x 的（最多一个）
s.erase(it);   // 删掉迭代器位置，返回 void（C++11 起 set::erase(it) 返回下一个迭代器，以实现为准，赛时少依赖返回值）
```

遍历中删除要用好返回的下一个迭代器，或先 `++` 再删旧的。入门题尽量不要边遍历边删，先记下来再删。

### 3. multiset：允许重复，删的时候小心

`multiset` 里 `s.count(x)` 可能是 100000，**它会一个一个数**，变成 O(n)。只要问「有没有」，用 `s.find(x) != s.end()`。

删除：

```cpp
ms.erase(x);           // 删掉所有值为 x 的！
ms.erase(ms.find(x));  // 只删一个
```

第一行是 magiset 入门杀手。题面「去掉一个 x」必须 `find` 再 `erase` 迭代器。实验室每年都有人 `erase(x)` 后发现袋子空了。

### 4. map：计数器与离散化

最常见：统计出现次数。

```cpp
map<string, int> cnt;
string w;
while (cin >> w) cnt[w]++;   // 没有这个键时 operator[] 会插入 0 再 ++
```

**`operator[]` 有副作用。** `if (mp[x] == 0)` 会把 `x` 插进去。只查询用：

```cpp
if (mp.find(x) != mp.end()) { ... }
// 或
if (mp.count(x)) { ... }
```

遍历按键升序：

```cpp
for (auto &p : cnt) {
    cout << p.first << ' ' << p.second << '\n';
}
```

C++17 可写成 `for (auto &[k, v] : cnt)`。

**离散化（和前缀和、树状数组配套）：** 值域 1e9，但只有 n 个不同的值。把出现过的值 sort + unique，用 `lower_bound` 得到 1..n 的新编号。`map<int,int>` 也能编号：

```cpp
map<int, int> id;
int tot = 0;
// 对每个值 x：
if (!id.count(x)) id[x] = ++tot;
```

静态、一次性离散化，**更推荐 vector + sort + unique + lower_bound**，常数小、代码短，专篇见 [离散化](../03-算法竞赛/离散化.md)。`map` 适合「边读边出现新值、还要动态查」。

### 5. 键的类型与自定义比较

`map` / `set` 默认用 `<`。`pair`、`string` 都能当键。`vector<int>` 也能比，但当键又慢又丑，入门别用。

浮点当键：`double` 的相等不稳定，`set<double>` 会把 1.0 和 0.999999999 当成两个。钱和几何优先整数化（乘 1e4 变 `long long`）。

自定义结构体当键，必须写 `operator<`，且比较必须是**严格弱序**（别写成 `<=`，否则插入行为诡异）。入门用 `pair` 凑三个关键字，少写结构体。

### 6. 和 vector 二分怎么选

| 场景 | 用什么 |
| --- | --- |
| 一次性读完，多次查询是否存在 / 后继 | `sort` + `lower_bound` |
| 动态插入删除，问最小最大 | `set` / `multiset` |
| 动态插入，问第 k 小 | 平衡树 / 线段树，入门先避开 |
| 字符串计数 | `map<string,int>` 或排完后扫 |
| 下标是 1e9 的「数组」 | `map<int, T>` |
| 下标是 1e6 以内的密整数 | 普通数组 / vector |

同一题若能离线成「先全部插入再查询」，优先排好的 vector。在线、交错改查，才上 `set`。

### 7. 复杂度与常数

树高大约 `2 * log2(n)`。n = 1e5 时，每次 insert 大约几十次比较。1e5 次插入通常 2 秒内能过；1e6 次在 CF 上有时吃紧，看时限和常数。

`mp[x]++` 若 x 从未出现，一次插入。不要写：

```cpp
for (int i = 1; i <= 1e7; i++) mp[i] = 0;  // 插一千万个空键，TLE + MLE
```

需要密下标就开数组。

## 常见坑 / 注意事项

- **`mp[x]` 查询造成插入。** 用 `find` / `count`。
- **`multiset.erase(x)` 全删。** 删一个用迭代器。
- **`set` 迭代器加减整数。** 不行。
- **解引用 `end()` / `--begin()`。** 空容器上 `*s.begin()` 是炸弹。先 `empty()`。
- **`count` 当「有没有」用在 `multiset` 上。** 可能 O(n)。
- **`unordered_map` 被卡。** 入门用 `map`；非要用哈希，赛时准备备用树版本。
- **`map<int,int> mp; cout << mp[0];` 调试插入了 0。** 调试输出都会改容器，用 `find`。
- **键是 `string` 时每次拷贝。** 查询尽量 `mp.find(s)`，大字符串计数注意常数。
- **对 `map` 的迭代器在 `insert` 之后仍使用旧 it。** `map` 插入不使其他迭代器失效，删除会使被删的失效。`vector` 则可能全体失效。别把规则记混。

## 趣味课间：实验室经典翻车集

- **「我开了 `bool vis[1000000001]`。」** 编译器也许不拦你，操作系统会。4e9 个 bool 不是内存，是愿望。
- **「`set` 自动排序，我还 `sort` 了一遍 vector 再逐个 insert。」** 功能对，时间翻倍。要么 vector 二分，要么纯 set，不要叠罗汉。
- **「`for (auto x : s) s.erase(x);`」** 边遍历边拆房子。先拷到 vector 再拆。
- **「Python 的 dict 多好。」** 是挺好。C++ 的 `map` 有序，`unordered_map` 无序。有序这点在「输出按键排序」「求后继」上反而是优势。
- **计数题输出 0。** 没出现过的键不应输出，还是应输出 0？看题面。`operator[]` 会让「没出现」变成「出现过且为 0」，遍历时多出一行。这是 WA 里最安静的一种。

## 小结

`set` 管「有哪些不同的键、最小最大后继」；`map` 管「键对应一个值」，最常用作计数器和稀疏数组。内部是树，不是魔法。能离线排序就离线；要动态再请它们入场。

建议练习：

- 洛谷 [P1102 A-B 数对](https://www.luogu.com.cn/problem/P1102)：计数，`map` 或排序双指针
- 洛谷 [P4305 [JLOI2013] 记忆的碎片](https://www.luogu.com.cn/problem/P4305) 同类「去重保序」：`set` 判出现过，`vector` 记答案顺序
- 洛谷 [P1638 逛画展](https://www.luogu.com.cn/problem/P1638)：区间里多少种，可用 `map` 计数 + 双指针（进阶一点）
- 把 `{3,1,3,2}` 依次 `insert` 进 `set` 和 `multiset`，打印 size，再 `erase(3)`，看还剩什么

算法线见 [贪心入门](../03-算法竞赛/贪心入门.md) 与 [BFS 网格最短路](../03-算法竞赛/BFS网格最短路.md)。语言线下一篇：[priority_queue](priority_queue.md)。

## 参考资料

- [vector 与 sort](vector与sort.md)
- [lower_bound 与二分](../03-算法竞赛/lower_bound与二分.md)
- [cppreference：std::map](https://en.cppreference.com/w/cpp/container/map)
- [cppreference：std::set](https://en.cppreference.com/w/cpp/container/set)
- [OI Wiki：STL 关联容器](https://oi-wiki.org/lang/csl/sequence-container/)
