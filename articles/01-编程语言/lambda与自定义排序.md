---
title: lambda 与自定义排序：第三关键字怎么写进 sort
author: TUST-ACM 算法实验室
date: 2026-09-23
category: 01-编程语言
tags: [C++, lambda, sort, STL, 语言]
level: 入门
status: 已完成
---

# lambda 与自定义排序：第三关键字怎么写进 sort

> 读完你应该能：用 lambda 给 `sort` 写「第一维升、第二维降」；给结构体写 `operator<` 或比较器；避开相等元素返回 `true` 导致的崩排；知道 [priority_queue](priority_queue.md) 的比较器和 `sort` 的「小于」方向容易反着写。
>
> 一句口号：**比较器回答「谁该排在前面」；必须构成严格弱序，相等时双方都返回 false。**

## 背景 / 为什么要读这篇

[vector 与 sort](vector与sort.md) 把默认 `pair` 升序钉过了，并说「第一维升、第二维降，下一篇再展开」。区域赛排结构体：按分数降序、分数相同按学号升序、再相同按报名时间。默认 `sort` 不会读你的心。

实验室有人写 `return a.x <= b.x`，样例 n=3 碰巧过，n=1e5 运行时错误或死循环；有人把大根堆的比较器抄进 `sort`，顺序整盘反掉。本篇只讲竞赛够用的 lambda 和结构体。

## 正文

### 1. lambda 塞进 `sort`

C++11 起就能用。形状：`[捕获](参数) { 函数体 }`。竞赛比较器通常不捕获，或只捕获数组。

```cpp
vector<pair<int, int>> v;
// 第一维升序，第二维降序
sort(v.begin(), v.end(), [](const pair<int, int>& a, const pair<int, int>& b) {
    if (a.first != b.first) return a.first < b.first;
    return a.second > b.second;
});
```

`const &` 避免拷贝大结构体。两层 `if` 比一行 `tie` 更好读、更好对拍。

三关键字：一层一层比，**先写会让顺序确定下来的那一维**。

```cpp
struct Node { int score, id, t; };
vector<Node> a;
sort(a.begin(), a.end(), [](const Node& x, const Node& y) {
    if (x.score != y.score) return x.score > y.score;  // 分高在前
    if (x.id != y.id) return x.id < y.id;
    return x.t < y.t;
});
```

### 2. 严格弱序：相等不能「小于」

`sort` 认为：`comp(a,b)==true` 表示 a 应排在 b 前。要求：

- 不能 `comp(a,a)==true`  
- `comp(a,b)` 与 `comp(b,a)` 不能同时为 true  
- 相等的两个元素，两次都是 false

因此写 `<` 或 `>`，**不要写 `<=` `>=`**。`<=` 让相等元素互相「小于」，是未定义行为：有的评测机 RE，有的静默排错。

错：

```cpp
return a.x <= b.x;  // a==b 时 true
```

对：

```cpp
return a.x < b.x;
```

需要稳定排序（相等元素保持输入顺序）用 `stable_sort`，比较器仍然必须严格。

### 3. 结构体 `operator<`

要进 `set` / `map` 当键，或懒得每次写 lambda，可以定义小于：

```cpp
struct Node {
    int x, y;
    bool operator<(const Node& o) const {
        if (x != o.x) return x < o.x;
        return y < o.y;
    }
};
```

`sort` 默认用 `operator<`。`priority_queue<Node>` 默认也用 `<`，但堆是**大根**：`top` 是「小于意义下最大的」。想小根堆：比较器反过来，或 `priority_queue<Node, vector<Node>, greater<>>`（需 `operator>` 或特化）。堆和 `sort` 抄同一份 lambda 时，**顺序经常反**，分开测。

### 4. 捕获：能用，少用

`[=]` 按值捕获、`[&]` 按引用。竞赛里按 `a[i]` 在外部数组排序下标：

```cpp
vector<int> idx(n), w(n);
iota(idx.begin(), idx.end(), 0);
sort(idx.begin(), idx.end(), [&](int i, int j) {
    if (w[i] != w[j]) return w[i] < w[j];
    return i < j;  // 下标当第二关键字，保证严格
});
```

`[&]` 捕获 `w`。不要在比较器里改 `w`。不要捕获已经死掉的局部指针。

[2026 赛场环境](../06-前沿动态/2026赛场环境.md) 的 `gnu++17` / `gnu++20` 都带 lambda。不要用 C++98 风格的仿函数类撑满草稿纸，除非模板里已经写死。

## 常见坑 / 注意事项

- **`<=` 当比较器。**  
- **只比第一维，第二维乱序还当稳定。** `sort` 不稳定。要么比到底，要么 `stable_sort`。  
- **`priority_queue` 与 `sort` 共用一个「谁更小」却期望相反的顶。**  
- **lambda 写在循环里每次 `sort` 捕获错变量。**  
- **`int` 相减当比较 `return a-b`。** 溢出。用 `<`。  
- **`vector<string>` 当数字排。** 仍是字典序，见 vector 篇。

## 趣味课间：实验室经典翻车集

- **「我加了 `<=` 更稳。」** 更不稳。相等元素在 STL 里会互殴。  
- **样例分数全不同。** 第二关键字永远走不到。造两个同分数。  
- **下标数组排序忘了 `i<j` 保底。** `w[i]==w[j]` 时两次 false 才合法；有人写成 `return true`。  
- **用 AI 生成比较器，里面 `std::tie` 和 `>` 混一行。** 自己读不懂就 [对拍](../00-团队指南/对拍.md) 小 n 全排列。

## 小结

lambda 给 `sort` 第三、第四关键字。分层 `!=` 再 `<`/`>`。相等双方 false。堆的比较器和排序不要混抄。默认 `pair` 仍够用时，不必上 lambda。

建议练习：

- 把 [vector 与 sort](vector与sort.md) 里的 `pair` 改成「first 升、second 降」，用 n=3 手排对照  
- 结构体三关键字，造两个同分不同 id  
- 同一组数据分别 `sort` 与小根 `priority_queue` 弹出，看顺序是否符合预期

树上小集合并进大集合：[启发式合并](../03-算法竞赛/启发式合并.md)。先手胜负：[博弈论入门](../03-算法竞赛/博弈论入门.md)。

## 参考资料

- [vector 与 sort](vector与sort.md)
- [priority_queue](priority_queue.md)
- [对拍](../00-团队指南/对拍.md)
- [OI Wiki：STL](https://oi-wiki.org/lang/csl/)
