---
title: 数位 DP 入门：把 n 的十进制拆成一位一位填
author: TUST-ACM 算法实验室
date: 2026-09-19
category: 03-算法竞赛
tags: [数位DP, DP, 计数, 算法]
level: 入门
status: 已完成
---

# 数位 DP 入门：把 n 的十进制拆成一位一位填

> 读完你应该能：把「1 到 n 中满足某性质的个数」写成按位 DFS；理解 `limit`（是否贴上界）和 `lead`（前导零）；会记忆化三元组 `(pos, limit, st)`；知道为什么 `solve(r) - solve(l-1)` 能求区间。
>
> 一句口号：**从高位往低位填数字。还在贴着 n 的前缀时，这一位不能随便填到 9；一旦比下去了，后面自由。**

## 背景 / 为什么要读这篇

[01 背包](01背包.md) 和 [LIS](LIS.md) 的下标是「物品 / 位置」。数位 DP 的下标是「当前填到第几位」。题面典型：`[L,R]` 里有多少个数各位数字满足……（不含 4、数位和为模 k、相邻差至少 2）。

R 到 1e18，枚举不行。把 R 的十进制拆开，长度最多 20 位，状态是位数 × 是否贴界 × 小小的附加信息。

实验室有人忘了 `lead`，把 007 当成含两个 0 的合法数；有人区间写成 `solve(r)-solve(l)` 少了 `l` 本身。本篇用「不含 4」走通模板。

## 正文

### 1. 区间变成前缀

性质在数上可加（计数）：

```text
ans[L,R] = f(R) - f(L-1)
```

`L=0` 时特判。`f(n)` = 0..n 中合法数的个数（是否含 0 看题）。

### 2. 按位 DFS 骨架

把 n 的各位存进 `a[0]` 高位 … `a[len-1]` 低位，或反过来只要一致。下面 **pos 从高位到低位，pos=0 是最高位**。

```cpp
int a[20], len;
long long dp[20][2];  // 记忆化按需加维

long long dfs(int pos, int limit, int lead /*, 其他状态 */) {
    if (pos == len) return /* lead ? 0 : 1 视是否统计 0 */;
    // 不贴界且无前导零时可以记忆化
    if (!limit && !lead && dp[pos][st] != -1) return dp[pos][st];
    int up = limit ? a[pos] : 9;
    long long r = 0;
    for (int d = 0; d <= up; d++) {
        if (/* d 不合法 */) continue;
        r += dfs(pos + 1,
                 limit && (d == up),
                 lead && (d == 0)
                 /*, 新状态 */);
    }
    if (!limit && !lead) dp[pos][st] = r;
    return r;
}
```

两个开关：

| 变量 | 含义 | 转移 |
| --- | --- | --- |
| `limit` | 前面填的和 n 的前缀完全一样 | 下一位上限是 `a[pos]`，否则 9 |
| `lead` | 到现在全是前导零 | 这一位仍可填 0 且不算「出现了 0」 |

一离开上界，`limit=0`，后面各位 0～9 自由，这才大量重复，记忆化才有意义。**还在 limit 或 lead 时不要记，或记进状态里。** 只记 `dp[pos]` 会把不同上界混在一起。

### 3. 例：0..n 中十进制不含数字 4 的个数

```cpp
#include <bits/stdc++.h>
using namespace std;

int digit[22], tot;
long long memo[22][2];

long long dfs(int pos, int limit, int lead) {
    if (pos == tot) return 1;  // 统计 0..n，空前缀算 1 个数（即 0）
    if (!limit && !lead && memo[pos][0] != -1) return memo[pos][0];
    int up = limit ? digit[pos] : 9;
    long long r = 0;
    for (int d = 0; d <= up; d++) {
        if (d == 4) continue;
        r += dfs(pos + 1, limit && d == up, lead && d == 0);
    }
    if (!limit && !lead) memo[pos][0] = r;
    return r;
}

long long solve(long long n) {
    if (n < 0) return 0;
    tot = 0;
    do {
        digit[tot++] = n % 10;
        n /= 10;
    } while (n);
    reverse(digit, digit + tot);  // 高位在前
    memset(memo, -1, sizeof memo);
    return dfs(0, 1, 1);
}
```

上面记忆化维偷懒了：`lead` 为真时分支少，可不记。更干净的是 `memo[pos][limit][lead]` 或只在 `!limit && !lead` 时写入。

手算 n=15：0～15 去掉 4、14，自己数一遍对照。

### 4. 常见附加状态

| 题 | 额外维 |
| --- | --- |
| 数位和 mod k | `sum % k` |
| 相邻差 ≥ 2 | 上一位数字（前导零时上一位可当 -1） |
| 恰好出现过某个数 | 二进制 mask（位数小） |
| 与 n 的大小关系 | 已有 limit |

维数乘起来 × 20 应在 1e7 内。mask 2^10 可以，2^20 再乘 pos 要算一算。

### 5. 和背包、记忆化搜索

这就是带约束的记忆化 DFS，不是新语言。`limit` 是数位题特有的「还能否自由」。写完用 `solve(n)` 对暴力 `for i=0..n` 小范围对拍，比任何口诀重要。

## 常见坑 / 注意事项

- **`solve(r)-solve(l)` 漏 L。** 用 `l-1`。  
- **`lead` 时把 0 当出现数字 0。**  
- **`limit=1` 仍记忆化且没把上界放进键。**  
- **digit 低位在前却当高位填。**  
- **n=0、L=0。**  
- **模数题忘 `% MOD`。**  
- **「不含 4」却禁了 14 的 1。** 只禁位上是 4。

## 趣味课间：实验室经典翻车集

- **「我递归没有 memo，n=1e18 也能过。」** 没有 limit 剪枝时指数爆炸；有 limit 无 memo 可能仍慢。小 n 对拍 + 大 n 计时。  
- **样例 L、R 很小。** 暴力都能过。对拍到 1e6。  
- **模板从网上抄的 1-based `dp[i][j]` 与自己的 pos 方向相反。** 画 3 位数 走一遍。

## 小结

区间差前缀。高位到低位填，`limit` 管上限，`lead` 管前导零，其余性质塞进状态。只在自由状态记忆化。小范围暴力对拍。

建议练习：

- 洛谷 [P2657 [SCOI2009] windy 数](https://www.luogu.com.cn/problem/P2657)
- 洛谷 [P2602 [ZJOI2010] 数字计数](https://www.luogu.com.cn/problem/P2602)
- `solve(15)` 不含 4，手算对照

DP 下一篇：[状压 DP](状压DP.md)。字符串多模式见 [AC 自动机入门](AC自动机入门.md)。

## 参考资料

- [01 背包](01背包.md)
- [DFS 回溯](DFS回溯.md)
- [状压 DP](状压DP.md)
- [OI Wiki：数位 DP](https://oi-wiki.org/dp/digit-dp/)
