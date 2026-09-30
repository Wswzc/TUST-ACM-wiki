---
title: LCA 倍增：树上两个点的最近公共祖先
author: TUST-ACM 算法实验室
date: 2026-09-17
category: 03-算法竞赛
tags: [LCA, 倍增, 树, 图论, 算法]
level: 入门
status: 已完成
---

# LCA 倍增：树上两个点的最近公共祖先

> 读完你应该能：DFS 预处理深度和 `fa[x][k]`（往上跳 2^k 步）；用倍增求出两点的 LCA；用深度差把两点跳到同一层；知道树上路径 `(u,v)` 经过 LCA，距离是 `dep[u]+dep[v]-2*dep[lca]`。
>
> 一句口号：**先爬到同一层，再一起往上跳。每次跳 2^k，能跳就跳，不能就减小 k。**

## 背景 / 为什么要读这篇

树上「两点的最近公共祖先」出现的频率，接近网格上的 BFS。路径交点、子树判定、差分改树上路径，都先要 LCA。

暴力：两个点轮流爬父亲，O(n) 每次。q=1e5 不行。倍增预处理 O(n log n)，询问 O(log n)，和 [二分](lower_bound与二分.md) 同一套「二进制拆」。

实验室有人 `fa[x][0]` 没赋成父亲；有人先跳 LCA 再对齐深度，顺序反了会跳飞。本篇把顺序写成仪式：先对齐，再同跳。

## 正文

### 1. 定义与树上距离

有根树（无根就指定 1 为根）。`lca(u,v)` 是 u、v 的公共祖先里深度最大的那个。

```text
dist(u,v) = dep[u] + dep[v] - 2 * dep[lca(u,v)]
```

u 在 v 的子树内 ⇔ `lca(u,v) == v`（或反过来）。路径覆盖、树上差分会用到。

### 2. 预处理：父亲的 2^k 倍

`fa[x][k]` = x 的 2^k 级祖先。`fa[x][0]` 是直接父亲。

```text
fa[x][k] = fa[ fa[x][k-1] ][k-1]
```

跳两段 2^{k-1} 就是 2^k。k 到 16 或 17 足够（2^16=65536，2^17>1e5）。n=1e6 用 20。

DFS 记 `dep` 和 `fa[][0]`，再 dp 出所有 k。

```cpp
const int LOG = 20;
int n, s;  // s 为根
vector<vector<int>> g, fa;
vector<int> dep;

void dfs(int u, int p) {
    fa[u][0] = p;
    for (int k = 1; k < LOG; k++)
        fa[u][k] = fa[ fa[u][k - 1] ][k - 1];
    for (int v : g[u]) {
        if (v == p) continue;
        dep[v] = dep[u] + 1;
        dfs(v, u);
    }
}
```

`fa[0][*] = 0`，跳到 0 表示没有祖先。根的父亲是 0。

也可以两层循环在 DFS 之后统一算 `fa[][k]`，不要在 DFS 里漏了某层。上面写法在进入 u 时 `fa[u][0]` 已有，立刻把更高 k 填完，儿子 DFS 时能用到。注意：**先填完 u 的所有 k，再 dfs 儿子。**

### 3. 询问：对齐再同跳

```cpp
int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);  // 让 u 更深
    int d = dep[u] - dep[v];
    for (int k = 0; k < LOG; k++)
        if (d >> k & 1) u = fa[u][k];  // 把 u 抬到和 v 同一层
    if (u == v) return u;              // v 是 u 的祖先
    for (int k = LOG - 1; k >= 0; k--) {
        if (fa[u][k] != fa[v][k]) {
            u = fa[u][k];
            v = fa[v][k];
        }
    }
    return fa[u][0];  // 再上一步就是 LCA
}
```

第二段必须从大 k 到小 k：像二进制从高位试。能跳（跳完祖先还不同）才跳，保证停在 LCA 的两个儿子上，最后 `fa[u][0]` 即 LCA。

从 0 往上跳会错过：可能一步过大。这和二分「从大区间缩小」是同一肌肉。

### 4. 完整骨架

```cpp
#include <bits/stdc++.h>
using namespace std;

const int LOG = 20;
int n, m, s;
vector<vector<int>> g, fa;
vector<int> dep;

void dfs(int u, int p) {
    fa[u][0] = p;
    for (int k = 1; k < LOG; k++) fa[u][k] = fa[fa[u][k - 1]][k - 1];
    for (int v : g[u]) if (v != p) {
        dep[v] = dep[u] + 1;
        dfs(v, u);
    }
}

int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    int d = dep[u] - dep[v];
    for (int k = 0; k < LOG; k++) if (d >> k & 1) u = fa[u][k];
    if (u == v) return u;
    for (int k = LOG - 1; k >= 0; k--)
        if (fa[u][k] != fa[v][k]) {
            u = fa[u][k];
            v = fa[v][k];
        }
    return fa[u][0];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m >> s;
    g.assign(n + 1, {});
    fa.assign(n + 1, vector<int>(LOG, 0));
    dep.assign(n + 1, 0);
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs(s, 0);
    while (m--) {
        int u, v;
        cin >> u >> v;
        cout << lca(u, v) << '\n';
    }
    return 0;
}
```

树是 n-1 条无向边。有根、有向边读题。

### 5. 别的算法什么时候上

| 方法 | 预处理 | 询问 | 备注 |
| --- | --- | --- | --- |
| 暴力爬 | 0 | O(n) | 小数据 |
| 倍增 | O(n log n) | O(log n) | **入门默认** |
| 欧拉序 + ST 表 | O(n log n) | O(1) | RMQ，常数好 |
| 树链剖分 | O(n) | O(log n) | 还要路径改点 |

先会倍增。ST 表版把 LCA 变成欧拉序上深度最小的点，和 [单调队列](单调栈与单调队列.md) 的 RMQ 是亲戚。

### 6. 复杂度

预处理 O(n log n)，空间 `n * LOG`。n=5e5、LOG=19 大约几十 MB 量级，注意。询问 O(LOG)。

深链递归 DFS 可能爆栈，n=1e6 链可改栈或 `bfs` 记父（按层）。多数模板题 n=5e5 递归能过，极端再换。

## 常见坑 / 注意事项

- **先 LCA 再对齐深度。** 必须先抬到同一层。
- **第二段 k 从 0 往上。** 会跳过 LCA。
- **`fa[root][0]` 不是 0。** 根父亲必须是 0。
- **`LOG` 太小。** n=1e5 至少 17。
- **无向边只存一次。**
- **u 已是 v 祖先时还同跳。** `if (u==v) return` 漏写，`fa` 可能乱。
- **距离公式写成 `dep[u]-dep[v]`。** 只有祖先-子孙才行。
- **多组数据 `fa` 没清。** `assign` 重建。

## 趣味课间：实验室经典翻车集

- **「我 BFS 同时从两点爬。」** 深度不同时对不齐，需要先对齐——又回到倍增第一步。
- **样例根是 1，你 dfs(1) 但题根是 s。** P3379 有 s。
- **路径差分 `d[u]++, d[v]++, d[lca]--, d[fa[lca]]--`。** 标记记错一步，整棵子树答案偏斜。先会查询 LCA，再做差分。
- **把 LCA 当最短路。** 树上最短路就是 `dist`，图上带环请 Dijkstra。

## 小结

`fa[x][k]` 往上 2^k。询问：深的先抬平，再从大 k 同步跳，最后一步父亲即 LCA。距离两条链长度相加减两段到 LCA。

建议练习：

- 洛谷 [P3379 【模板】最近公共祖先（LCA）](https://www.luogu.com.cn/problem/P3379)
- 过后再做树上距离、[树上差分](树上差分.md)、[P2680 运输计划](https://www.luogu.com.cn/problem/P2680)（进阶）
- 画一条 8 个点的链，手算 `fa[8][0..3]`，再问 `lca(6,8)`

字符串下一篇：[KMP](KMP.md)。路径加计数：[树上差分](树上差分.md)。路径条数：[点分治](点分治.md)。一组关键点：[虚树](虚树.md)。

## 参考资料

- [lower_bound 与二分](lower_bound与二分.md)
- [OI Wiki：最近公共祖先](https://oi-wiki.org/graph/lca/)
- [OI Wiki：倍增](https://oi-wiki.org/basic/binary-lifting/)
