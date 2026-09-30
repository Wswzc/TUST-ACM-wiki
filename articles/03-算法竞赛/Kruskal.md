---
title: Kruskal：把边从便宜到贵铺成一棵树
author: TUST-ACM 算法实验室
date: 2026-09-17
category: 03-算法竞赛
tags: [Kruskal, MST, 最小生成树, 并查集, 贪心]
level: 入门
status: 已完成
---

# Kruskal：把边从便宜到贵铺成一棵树

> 读完你应该能：用「排序 + 并查集」写出最小生成树；判断图不连通时怎么办；知道 Kruskal 适合稀疏图、Prim 适合稠密图；能解释为什么不形成环的最便宜边一定可以选。
>
> 一句口号：**边按重量排队，能连两个不同帮派就铺上，铺满 n-1 条就停。**

## 背景 / 为什么要读这篇

[并查集](并查集.md) 已经会判「两点是否一伙」和「这条边会不会成环」。最小生成树（MST）把这两件事做成算法：在无向带权图上选一组边，连通所有点，且边权之和最小。

修路、光纤、集群之间连线——题面常常这味。它也是 [贪心](贪心入门.md) 为数不多「定理保证正确」的现场：Kruskal 的交换论证几乎是教科书原句。

实验室有人边没排序就 unite，得到一棵树但不是最小；有人无向边只存一次却把有向图当 MST；有人连通块 >1 时仍输出一个「总和」，而题面要求不可能。本篇把流程写成不会跳步的清单。

## 正文

### 1. 树、生成树、最小

n 个点的无向连通图：

- **树：** n-1 条边、连通、无环（三条等价）
- **生成树：** 包含全部顶点的树
- **MST：** 边权之和最小的生成树（权可以相同，树可以不唯一，**权和唯一**）

不连通：没有生成树。此时 Kruskal 会发现选不满 n-1 条，输出「无解」或「森林的最小权」——看题面。最小瓶颈生成树、次小生成树是后话。

### 2. 算法就三步

1. 把所有边按权 **从小到大** 排序  
2. 依次看每条边 `(u,v,w)`：若 `u,v` 不在同一集合，选它，`unite`  
3. 选满 n-1 条则成功；边用完还不够则原图不连通  

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Edge {
    int u, v, w;
    bool operator<(const Edge& o) const { return w < o.w; }
};

struct DSU {
    vector<int> fa, rk;
    DSU(int n) : fa(n + 1), rk(n + 1, 0) {
        iota(fa.begin(), fa.end(), 0);
    }
    int find(int x) { return fa[x] == x ? x : fa[x] = find(fa[x]); }
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (rk[x] < rk[y]) swap(x, y);
        fa[y] = x;
        if (rk[x] == rk[y]) rk[x]++;
        return true;
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<Edge> e(m);
    for (int i = 0; i < m; i++) cin >> e[i].u >> e[i].v >> e[i].w;
    sort(e.begin(), e.end());

    DSU dsu(n);
    long long ans = 0;
    int used = 0;
    for (auto [u, v, w] : e) {
        if (dsu.unite(u, v)) {
            ans += w;
            used++;
            if (used == n - 1) break;
        }
    }
    if (used != n - 1) cout << "orz\n";  // 不连通，输出随题面
    else cout << ans << '\n';
    return 0;
}
```

边权、总和用 `long long`。`used == n-1` 是连通的充要检查（原图连通时一定能选满，因为生成树存在）。

### 3. 为什么贪心对：最便宜的安全边

当前已经选了一些边，形成若干连通块。下一条「两端不同块」的最便宜边 e：假设某棵 MST 不含 e，那 MST 里连接这两块的那条边 e' 权 ≥ e（否则 Kruskal 会先考虑 e'）。用 e 换 e'，仍连通，权和不增。所以存在 MST 包含 e。

成环的边两端已在同块，加上去破坏「树」，跳过。这就是并查集 `unite` 返回 `false` 的时刻。

按权从大到小选最大生成树：比较器反过来，其余一字不改。

### 4. Kruskal vs Prim

| | Kruskal | Prim |
| --- | --- | --- |
| 想法 | 扫边 | 从点长出一棵树 |
| 数据结构 | 排序 + 并查集 | 堆（像 Dijkstra） |
| 复杂度 | O(m log m) | 堆优化 O((n+m) log n) |
| 更合适 | **稀疏图** m ~ n | **稠密图** m ~ n²，或堆已写熟 |

入门先 Kruskal：排序人人会，并查集刚学完。Prim 的松弛和 [Dijkstra](Dijkstra.md) 几乎一张皮（dist 是「到树的距离」不是「到源点」）。稠密图 n=2000 完全图，Prim 朴素 O(n²) 更干净。

### 5. 题面变种

- **只要权和：** 上面模板  
- **要输出选了哪些边：** `unite` 成功时把边下标推进 `vector`  
- **第 k 小生成树 / 次小：** 先 MST，再枚举树上边替换，进阶  
- **严格次小：** 注意不能和 MST 权和相同还当次小，看题  
- **最小生成森林、恰好 k 个连通块：** 选到 `sets == k` 停，或选 n-k 条边  

洛谷模板常见：不连通输出 `orz`。先读题。

### 6. 复杂度

排序 O(m log m) 占主导。并查集几乎 O(m)。m=1e5 轻松。点数 1e5、边 1e5 是常规。

## 常见坑 / 注意事项

- **忘了 sort。** 树有了，最小没有。
- **无向图当成有向。** MST 定义在无向图。有向是最小树形图，另一座山。
- **不检查 `used == n-1`。** 不连通仍输出 ans。
- **自环。** `unite` 同一点失败，自动跳过，还好。
- **重边。** 排序后便宜的先考虑，正确。
- **`int` 攒 ans。** 边权 1e9、n=1e5。
- **点编号 0 或从 1。** DSU 大小与编号一致。
- **把最短路当 MST。** 最短路是点到点；MST 是一棵撑住全图的树。从 1 出发的 Dijkstra 树一般不是 MST。

## 趣味课间：实验室经典翻车集

- **「我 Dijkstra 一遍，留下的边。」** 那是最短路树。反例很小：三角形边权 1,100,100，从一点 Dijkstra 可能留两条 100，MST 却是 1+100。
- **样例只有一棵星星。** 任何生成树都最小。造一个有环、环上有一条明显便宜边的图。
- **Kruskal 写完去套有向边。** 评测机不负责解释图论定义。
- **按秩没写、路径没压缩、m=1e5 链。** 并查集篇的 TLE 在这里复发。

## 小结

边排序，并查集收边，满 n-1 条就是 MST。不连通要单独说。稀疏 Kruskal，稠密再考虑 Prim。贪心正确性来自「最便宜的跨块边可换上去」。

建议练习：

- 洛谷 [P3366 【模板】最小生成树](https://www.luogu.com.cn/problem/P3366)
- 洛谷 [P1195 口袋的天空](https://www.luogu.com.cn/problem/P1195)：保留 k 个连通块
- 手画 4 个点若干边，按权勾选，对照程序的 `used` 和 `ans`

图论下一篇：[拓扑排序](拓扑排序.md)。路径最大边做成 LCA：[Kruskal 重构树](Kruskal重构树.md)。

## 参考资料

- [并查集](并查集.md)
- [贪心入门](贪心入门.md)
- [OI Wiki：最小生成树](https://oi-wiki.org/graph/mst/)
