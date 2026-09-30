---
title: Dijkstra：非负边权的最短路，以及堆里那些过期的面包屑
author: TUST-ACM 算法实验室
date: 2026-09-17
category: 03-算法竞赛
tags: [Dijkstra, 最短路, 图论, 堆, 算法]
level: 入门
status: 已完成
---

# Dijkstra：非负边权的最短路，以及堆里那些过期的面包屑

> 读完你应该能：用邻接表 + 小根堆写出单源最短路；解释为什么边权必须非负；会用「弹出时若距离不是当前最优就丢掉」做懒删除；在 BFS / Dijkstra / Bellman-Ford 之间会选题。
>
> 一句口号：**BFS 是边权全为 1 的 Dijkstra；Dijkstra 是带权的水波纹——每次先扩当前最近的点。**

## 背景 / 为什么要读这篇

[BFS](BFS网格最短路.md) 里水波纹均匀，因为每步都是 1。路上出现「这条边长度 3、那条长度 1」，先被队列碰到的点不一定更近——近路可能在后面才展开。

Dijkstra 的修正很贪心，也因此只在**边权 ≥ 0** 时正确：始终取出「当前已知距离最小、且尚未定论」的点，把它的距离钉死，再去松弛邻居。钉死这一步依赖非负性：后面不可能通过一条负边抄到更短（没有负边可抄）。

[priority_queue](../01-编程语言/priority_queue.md) 已经预习过小根堆和懒删除。本篇把它接到图上。实验室经典事故是：有负边仍 Dijkstra、或堆里过期状态不丢导致 TLE、或 `int` 装不下距离。

## 正文

### 1. 松弛：最短路共同的一句话

有边 `u → v` 权 `w`。若 `dist[u] + w < dist[v]`，则发现一条更短的路到达 v，更新 `dist[v]`。这叫松弛。

所有最短路算法都在松弛。差别是**按什么顺序保证每个点只需要被正确地处理有限次**。Dijkstra 的顺序是：永远先处理当前 `dist` 最小的未完成点。

非负保证：当 u 成为全局最小未完成点时，不会再有另一条路更短——任何还没处理的点 x 都有 `dist[x] ≥ dist[u]`，再走非负边只会更大。于是 `dist[u]` 被钉死。

有负边：这个「不会更短」破产。请用 Bellman-Ford / SPFA（后者易被卡）或把负边从模型里消掉。

### 2. 邻接表：先把图存对

n 个点 m 条边，点从 1 编号。有向边按输入；无向边存两次。

```cpp
    vector<vector<pair<int, int>>> g(n + 1);  // {邻点, 边权}
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        g[u].push_back({v, w});
        g[v].push_back({u, w});  // 无向图才写这行
    }
```

不要用邻接矩阵除非 n ≤ 500 且边稠密。n=1e5、m=1e5 时矩阵是 1e10 的内存幻想。

### 3. 堆优化模板（竞赛默认这一份）

```cpp
#include <bits/stdc++.h>
using namespace std;

const long long INF = 4e18;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m, s;
    cin >> n >> m >> s;
    vector<vector<pair<int, int>>> g(n + 1);
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        g[u].push_back({v, w});
    }

    vector<long long> dist(n + 1, INF);
    priority_queue<pair<long long, int>,
                   vector<pair<long long, int>>,
                   greater<pair<long long, int>>> pq;
    dist[s] = 0;
    pq.push({0, s});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        if (d != dist[u]) continue;  // 过期面包屑
        for (auto [v, w] : g[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }

    for (int i = 1; i <= n; i++) {
        if (i > 1) cout << ' ';
        cout << (dist[i] >= INF / 2 ? -1 : dist[i]);  // 达不到的输出看题面
    }
    cout << '\n';
    return 0;
}
```

关键行：`if (d != dist[u]) continue;`。同一点可能多次入堆，旧的 `{更大的 d, u}` 仍躺着。弹出时若 `d` 不是最新 `dist[u]`，说明这是过期状态，丢掉。这就是懒删除——堆不支持改中间，就让垃圾自己浮上来再扔。

不要在第一次弹出 u 之后用 `vis[u]=1` 并拒绝再次松弛……等等，标准 Dijkstra 确实可以「第一次出堆就钉死」，此时 `d` 一定等于 `dist[u]`（非负前提下）。`vis` 版和懒删除版都对。懒删除少一个数组，多一些垃圾条目；稠密图有时 `vis` 更干净。入门先记懒删除，和堆篇一致。

**第一次出堆即钉死（带 vis）：**

```cpp
vector<int> vis(n + 1, 0);
// ...
if (vis[u]) continue;
vis[u] = 1;
// 然后松弛邻居
```

两种留一种，不要混到既 vis 又不管 `d != dist[u]` 还重复松弛出玄学。

### 4. 朴素 O(n²) 什么时候够用

没有堆：每次在未完成点里扫一遍找 `dist` 最小。`O(n² + m)`。n ≤ 1000、稠密图（m ~ n²）时，它比堆还快——堆的 `O((n+m) log n)` 在 m=n² 时多 log，常数也不免费。

网格带权（每个格子走入代价不同）也可以 Dijkstra，状态仍是格子。边权为 1 请回 BFS，不要炫技。

### 5. 和 BFS、SPFA、Floyd 怎么选

| 图 | 算法 |
| --- | --- |
| 边权全 1 或无权 | BFS |
| 边权非负 | Dijkstra |
| 有负边、无负环，单源 | Bellman-Ford；SPFA 能写但可能被卡 |
| 有负环要检测 | Bellman-Ford 多松弛一轮 |
| n ≤ 400，所有点对 | Floyd |
| 0/1 两种边权 | 0-1 BFS |

最短路不是一种算法，是一类问题。先问边权，再问单源还是全源，再动手。

### 6. 复杂度与 INF

堆优化：`O((n + m) log n)`（懒删除时堆里可能有更多条目，最坏多次更新同一点）。m=1e5、n=1e5 是常规时限。

`INF` 取 `4e18` 或 `1e18`，保证 `dist[u] + w` 不溢出 `long long`。若 `w` 最大 1e9、路径最长 n 条，`n*1e9` 约 1e14，`long long` 够；写成 `int` 的 `0x3f3f3f3f` 在边权 1e9 时两下就爆。**距离用 `long long`。**

达不到：`dist[i]` 仍是 INF。输出 `-1` 还是 INF 看题。比较时用 `dist[i] > INF/2` 比 `== INF` 抗一点点加法污染。

### 7. 路径还原

`pre[v] = u` 在成功松弛时赋值。从终点沿 `pre` 走回源点，再 reverse。源点 `pre[s]=0`。达不到不要还原。

多条最短路要计数：松弛时 `==` 则 `cnt[v] += cnt[u]`，`<` 则重置。这是最短路计数，模数看题。入门先会单条路径。

## 常见坑 / 注意事项

- **有负边仍 Dijkstra。** 错得可以很安静。
- **无向图只存一遍。**
- **`int` 距离。**
- **忘了懒删除 / vis，同一点无限更新。** 正权一般会停，但常数爆炸。
- **`priority_queue` 默认大根堆。** 必须 `greater`，否则变成「先扩最远的点」。
- **源点 `dist[s]` 忘了 0。**
- **点从 0 编号，数组开了 `n+1` 却循环 `1..n`。** 读题。
- **自环、重边。** 邻接表自然处理；若取 min 重边可在建图时合并，不合并也对，就是多几次松弛。
- **`INF + w` 溢出成负数，被当成更短。** `long long` + 合理 INF。
- **网格最短路用了 Dijkstra 且边权为 1。** 正确但慢。BFS。

## 趣味课间：实验室经典翻车集

- **「我用 SPFA 更短。」** 短是短，CF 上卡 SPFA 是传统节目。非负边 Dijkstra 是礼貌。
- **「堆里有 u 我就 vis 掉，邻居更新后 u 更短却进不去。」** 你 vis 早了。钉死只发生在**出堆且 d 为最优**时。更新邻居只 `push`，不提前 vis 邻居。
- **样例是一条链。** 任何算法都对。造一个「先走长边再走短边更优」的叉：S—10—A，S—1—B—1—A。Dijkstra 应先钉 B 再更新 A 为 2，而不是 10。
- **Python 的 heapq 和 C++ 默认相反。** 见堆篇。
- **输出 INF 被判 WA。** 题面写了达不到输出 2147483647 或 -1，照抄，不要发明。

## 小结

非负边权、单源：小根堆取出当前最近，松弛邻居，过期状态扔掉。边权为 1 用 BFS；有负边换算法。距离 `long long`，图用邻接表。

建议练习：

- 洛谷 [P4779 【模板】单源最短路径（标准版）](https://www.luogu.com.cn/problem/P4779)：非负，卡 SPFA
- 洛谷 [P3371 【模板】单源最短路径（弱化版）](https://www.luogu.com.cn/problem/P3371)：n 小可用朴素
- 把「S-10-A，S-1-B-1-A」手跑一遍堆里的弹出顺序
- 边权全改成 1，对照 [BFS](BFS网格最短路.md) 的 `dist` 应一致

数据结构下一篇：[并查集](并查集.md)。DP 线仍可接 LIS。小而密的全源最短路用 [Floyd](Floyd.md)。费用流里正权最短路可换回堆版 Dijkstra，见 [费用流入门](费用流入门.md)。不等式组见 [差分约束](差分约束.md)（负权，不要用本篇板子硬套）。

## 参考资料

- [BFS 网格最短路](BFS网格最短路.md)
- [差分约束](差分约束.md)
- [priority_queue](../01-编程语言/priority_queue.md)
- [OI Wiki：Dijkstra](https://oi-wiki.org/graph/shortest-path/#dijkstra)
- 洛谷最短路模板题
