---
title: vector 与 sort：最小刷题习惯
author: TUST-ACM 算法实验室
date: 2026-09-16
category: 01-编程语言
tags: [C++, STL, vector, sort, 竞赛]
level: 入门
status: 已完成
---

# vector 与 sort：最小刷题习惯

> 读完你应该能：用 `vector<int>` 读入 n 个数，`sort` 排好，按题面输出。不再手写冒泡，也不再被变长数组和 `qsort` 绊住。

## 背景 / 为什么要读这篇

输入循环会了之后，下一类入门题几乎都是：**先读 n，再读 n 个数，排序 / 找最大 / 去重**。课内可能刚讲完数组和双重循环；竞赛里默认用 STL 的 `vector` 和 `sort`。

这不是炫技。`sort` 是 `O(n log n)` 的introsort，现场赛手写 `O(n^2)` 冒泡，n = 1e5 就会 TLE。`vector` 知道自己的长度，少一批「数组开小了 / 开在函数里爆栈」的 RE。

本篇只讲刷题最少要用的那一部分。`map`、`set`、自定义结构体的复杂比较，以后单开。

## 正文

### 1. 用 vector，不要用裸数组硬撑

竞赛里一段「n 个数」的默认写法：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) {
        cin >> a[i];
    }
    return 0;
}
```

`vector<int> a(n)` 表示有 n 个 `int`，下标 `0 .. n-1`，初值是 0。长度事先知道时，**先开好再按下标读**，不要 `push_back` 一百次又忘记 `cin >> n`。

长度事先不知道、一边读一边塞时才用 `push_back`：

```cpp
vector<int> a;
int x;
while (cin >> x) {
    a.push_back(x);
}
```

这和 [多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md) 是同一件事：结束条件仍由题面决定。

常用接口先记三个：

| 写法 | 含义 |
| --- | --- |
| `a.size()` | 当前元素个数，类型是无符号的 `size_t` |
| `a[i]` | 第 i 个，从 0 开始 |
| `a.push_back(x)` | 在末尾追加 |

循环里不要写 `for (int i = 0; i < a.size() - 1; i++)` 然后假设 `a` 永远非空：`size()` 是无符号数，空 vector 时 `a.size() - 1` 会下溢成一个巨大的数，循环会炸。空容器先判断，或写成 `i + 1 < a.size()`。

### 2. sort：默认从小到大

```cpp
sort(a.begin(), a.end());
```

区间是**左闭右开**：排的是 `[begin, end)`，也就是全部元素。这和数学课的区间写法一样，少排一个、多排一个都是 WA。

完整最小题：读 n 和 n 个整数，升序输出，空格分隔。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) cin >> a[i];

    sort(a.begin(), a.end());

    for (int i = 0; i < n; i++) {
        if (i) cout << ' ';
        cout << a[i];
    }
    cout << '\n';
    return 0;
}
```

从大到小：

```cpp
sort(a.begin(), a.end(), greater<int>());
```

只排一段，例如下标 `l` 到 `r` 含两端（0-based）：

```cpp
sort(a.begin() + l, a.begin() + r + 1);
```

右端必须 `+ 1`，因为 `end` 是开的。写成 `a.begin() + r` 会漏掉 `a[r]`。

### 3. 排序后去重（几乎立刻会用到）

很多题「输出不同的数、从小到大」。正确顺序是：**先 sort，再 unique，再删掉尾部垃圾。**

```cpp
sort(a.begin(), a.end());
a.erase(unique(a.begin(), a.end()), a.end());
```

`unique` 只把相邻的重复项挤走，所以必须先排序。它返回「新逻辑结尾」的迭代器，后面那些槽位仍是垃圾，必须 `erase` 掉。只 `unique` 不 `erase`，`a.size()` 不变，输出会多东西。

### 4. 结构体 / pair 的第一层

两个关键字：先按第一维，再按第二维。竞赛里用 `pair` 就够入门：

```cpp
vector<pair<int, int>> v(n);
for (int i = 0; i < n; i++) {
    cin >> v[i].first >> v[i].second;
}
sort(v.begin(), v.end());  // 先 first，相同再 second，都是升序
```

要「第一维升序、第二维降序」再用 lambda，见 [lambda 与自定义排序](lambda与自定义排序.md)。现在先保证默认 `pair` 排序能写对。

### 5. 复杂度与何时不能手写

| 做法 | 常见复杂度 | n = 1e5 |
| --- | --- | --- |
| 冒泡 / 选择 | O(n^2) | 超时 |
| `sort` | O(n log n) | 通常可通过 |
| 已经有序时再 `sort` | 仍是 O(n log n)，常数可接受 | 不要因此改回冒泡 |

题面 `1 <= n <= 1e5` 就不要双重循环两两比较。`n <= 1000` 时冒泡能过，但不要养成习惯——同一套代码下一题 n 开大就废。

## 常见坑 / 注意事项

- **下标从 1 开始却开了 `vector<int> a(n)`。** 你写 `a[1] .. a[n]` 时 `a[n]` 已经越界。要么开 `n+1` 并浪费 `a[0]`，要么全程 0-based。入门建议 0-based，和 `sort` 一致。
- **`sort(a, a + n)` 用在 vector 上。** 那是 C 数组写法。vector 用 `a.begin()` / `a.end()`。
- **对 `vector<string>` 按数值大小排序。** `"9"` 会排在 `"10"` 后面，因为默认是字典序。数字就用 `vector<int>` 或 `vector<long long>`。
- **数据范围用了 `int` 装不下。** 排序本身没错，加法、乘法先考虑 `long long`。`sort` 对 `long long` 同样可用。
- **输出末尾多空格。** 多数 OJ 不卡，有的卡。用「先输出分隔符再输出数」或 `i ? " " : ""` 更稳。
- **`unique` 前没 `sort`。** 只能去掉相邻重复，`1 2 1` 去不掉两个 1。
- **为了排序去 `#include <algorithm>` 却忘了 `bits/stdc++.h`。** 竞赛万能头已经包含；若不用万能头，需要 `<vector>` 和 `<algorithm>`。
- **把 `a.size()` 赋给 `int` 时的警告。** 入门期 `int n = (int)a.size();` 即可，n 在竞赛限制内不会超出 `int`。

## 小结

刷题最小 STL 就是两块：`vector` 管变长序列，`sort` 管顺序。会读 n 个数、会排、会按格式输出，入门题里「排序 / 名次 / 简单贪心」就可以上手了。

建议接着做：

- 洛谷 [P1177 【模板】排序](https://www.luogu.com.cn/problem/P1177)（直接 `sort` 即可，用来确认环境与输出格式）
- 洛谷 [P1059 明明的随机数](https://www.luogu.com.cn/problem/P1059)（`sort` + `unique`）
- 下一篇：[在有序的世界里砍半：lower_bound 与二分](../03-算法竞赛/lower_bound与二分.md)
- 第三关键字：[lambda 与自定义排序](lambda与自定义排序.md)
- 语言习惯仍按 [新人指南](../00-团队指南/新人指南.md) 第 1 学期往下：`map` / `set` / `priority_queue`

## 参考资料

- [多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md)
- [lambda 与自定义排序](lambda与自定义排序.md)
- [cppreference：std::sort](https://en.cppreference.com/w/cpp/algorithm/sort)
- [cppreference：std::vector](https://en.cppreference.com/w/cpp/container/vector)
- [OI Wiki：STL 简介](https://oi-wiki.org/lang/csl/)
