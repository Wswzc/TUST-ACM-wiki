---
title: Floyd：所有点对之间的最短路
author: TUST-ACM 算法实验室
date: 2026-09-29
category: 03-算法竞赛
tags: [Floyd, 最短路, 图论, 算法]
level: 入门
status: 已完成
---

# Floyd：所有点对之间的最短路

> 读完你应该能：写出三重循环且 **k 在最外层**；用邻接矩阵处理 n≤400 的全源最短路；用同样的循环判负环、做传递闭包；说明什么时候必须改 [Dijkstra](Dijkstra.md)。
>
> 一句口号：**允许经过的中间点从 1 放到 n。k 不在最外面，归纳就断了。**

## 背景 / 为什么要读这篇

[Dijkstra](Dijkstra.md) 是单源、非负权，堆优化后适合 n=1e5。题若要**每一对点**的最短路，n 只有几百，Floyd 更短：一个矩阵，三重循环。

实验室有人把 k 写在最内层，小图碰巧对、稍大就错；有人 `INF+INF` 溢出变成负数，出现假的更短边。本篇把循环顺序和 INF 钉死。

## 正文

### 1. 松弛

`d[i][j]`：当前只允许中间点编号 `< k`（或 `≤k`，看你的循环）时，i 到 j 的最短路。加入点 k：

```text
d[i][j] = min(d[i][j], d[i][k] + d[k][j])
```

k 必须已经用「更小的中间点集合」更新完，所以 **k 在最外层**。

```cpp
const long long INF = 4e18;
long long d[405][405];

void floyd(int n) {
    for (int k = 1; k <= n; k++)
        for (int i = 1; i <= n; i++)
            if (d[i][k] < INF)
                for (int j = 1; j <= n; j++)
                    if (d[k][j] < INF)
                        d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
}
```

初始化：`d[i][i]=0`，有边 `d[u][v]=min(已有, w)`（重边取最小），其余 INF。无向图双向都赋。

复杂度 O(n³)。n=400 约 6e7，能过；n=1e3 通常不行。

### 2. 负环

无负环时 Floyd 正确，负边可以有。若存在负环，某个 `d[i][i]<0`（算法结束后）。用它判「是否有负环」。有负环时最短路无定义，不要输出那张表。

[差分约束](差分约束.md) 的负环用 SPFA 更常见，因为 n 可能不是几百、图是稀疏的。稠密小图可以用 Floyd。

### 3. 传递闭包

只关心「能不能走到」，把 min-plus 换成布尔或：

```cpp
// g[i][j] 初始：i==j 或有边则为 true
for (int k = 1; k <= n; k++)
    for (int i = 1; i <= n; i++)
        if (g[i][k])
            for (int j = 1; j <= n; j++)
                g[i][j] = g[i][j] || g[k][j];
```

这是 Floyd 的骨架，运算从「加后取 min」换成「或」。竞赛里 n≤500 的可达性经常这么写，比每个点一次 BFS 更好写；点再多就每个点 BFS/DFS。

### 4. 和 Dijkstra、BFS

| 图 | 选择 |
| --- | --- |
| 全源、n≤400、可有负边无负环 | Floyd |
| 单源、非负、n 大 | Dijkstra |
| 边权 1 的单源 | BFS |
| 单源、有负边、无负环、稀疏 | SPFA（可能被卡）或 Johnson |

不要对 n=1e5 写 Floyd。也不要在非负稀疏图上用 Floyd 代替堆优化 Dijkstra 去交模板题。

## 常见坑 / 注意事项

- **k 在最内层。**  
- **`INF` 用 `0x3f3f3f3f`，两条边一加溢出。** 判断两边都 `< INF` 再松弛，或 INF 取 `4e18` 且别让 `4e18+4e18`。  
- **忘记 `d[i][i]=0`。**  
- **有向边建成双向。**  
- **负环仍输出距离。**  
- **重边用赋值覆盖，没取 min。**

## 趣味课间：实验室经典翻车集

- **「三重循环谁在外都一样。」** 把 k 放里面，用 4 个点一条需要「先松弛中间再松弛」的路径就能打脸。  
- **样例 n=2。** 不经过第三点。造 1→2→3，边权让直连更差。  
- **INF 是 1e9，边权 1e9，`1e9+1e9` 在 int 里溢出成负，出现更短的假路。**  
- **传递闭包用 Floyd 的 min，布尔题输出距离。** 运算和题要匹配。

## 小结

k 最外层，用 k 松弛所有点对。n³，适合小而密。负环看对角线。可达性把加法换成或。大图单源回 Dijkstra 或 BFS。

建议练习：

- 洛谷 [P2910 [USACO08OPEN] Clear And Present Danger](https://www.luogu.com.cn/problem/P2910) 或任意 Floyd 模板  
- 手算 3 个点、k 从 1 到 3 的矩阵  
- 造一个负环，看 `d[i][i]`

## 参考资料

- [Dijkstra](Dijkstra.md)
- [BFS 网格最短路](BFS网格最短路.md)
- [差分约束](差分约束.md)
- [OI Wiki：Floyd](https://oi-wiki.org/graph/shortest-path/#floyd)
