---
title: AC 自动机入门：多模式串一起跳
author: TUST-ACM 算法实验室
date: 2026-09-19
category: 03-算法竞赛
tags: [AC自动机, 字符串, KMP, Trie, 算法]
level: 入门
status: 已完成
---

# AC 自动机入门：多模式串一起跳

> 读完你应该能：把多个模式插入 Trie；BFS 建立 `fail` 指针；在文本上一遍扫完统计每种模式出现次数（或是否出现）；说出 fail 和 KMP 的 `next` 是同一类「失配后跳到哪」。
>
> 一句口号：**Trie 存所有模式。fail 指向「当前串的真后缀里，最长的那个仍是某模式前缀」的结点。文本失配就跳 fail，像多模式的 KMP。**

## 背景 / 为什么要读这篇

[KMP](KMP.md) 一次对付一个模式。题面 1e5 个模式在一篇文本里出现几次，每个 KMP 一遍是 O(文本 × 模式数)，不行。

AC 自动机（Aho–Corasick）：先把模式建成 Trie，再像 KMP 那样给每个结点一个 fail。文本只扫一遍，沿 trie 边或 fail 走。总复杂度与「文本长 + 模式总长 + 出现次数」相关（实现细节影响）。

实验室有人 fail 建成父亲、或统计时没沿 fail 链把「这个结点代表的模式」累上。本篇给一份计数模板，二进制分组、广义自动机以后再说。

## 正文

### 1. Trie + fail

每个结点：`ch[26]`、`fail`、`cnt`（有多少模式以这里结尾）。插入和普通 Trie 一样。

fail：BFS，根的儿子 fail 指向根。结点 u 沿字符 c 的孩子 v：

```text
v.fail = 从 u.fail 一直跳，直到有 c 儿子或到根
然后 v.fail = 那个 c 儿子（没有则根）
```

与 KMP：`next[i]` 是前缀函数；fail 是「当前前缀」在自动机里的结点版。

### 2. 模板（小写字母，统计每种出现次数）

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ACA {
    static const int N = 1000000;  // 总结点数上界
    int ch[N][26], fail[N], cnt[N], tot;
    vector<int> endid;  // 第 i 个模式结束结点，可外置
    ACA() { tot = 1; }  // 1 为根，0 作空
    int insert(const string& s) {
        int u = 1;
        for (char c : s) {
            int x = c - 'a';
            if (!ch[u][x]) ch[u][x] = ++tot;
            u = ch[u][x];
        }
        cnt[u]++;  // 或记录模式编号
        return u;
    }
    void build() {
        queue<int> q;
        for (int i = 0; i < 26; i++) {
            if (ch[1][i]) {
                fail[ch[1][i]] = 1;
                q.push(ch[1][i]);
            } else ch[1][i] = 1;  // 缺边连回根，查询少判断
        }
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int i = 0; i < 26; i++) {
                if (ch[u][i]) {
                    fail[ch[u][i]] = ch[fail[u]][i];
                    q.push(ch[u][i]);
                } else ch[u][i] = ch[fail[u]][i];
            }
        }
    }
    long long query(const string& t) {  // 文本里匹配到的模式次数之和（可重复模式）
        int u = 1;
        long long ans = 0;
        for (char c : t) {
            u = ch[u][c - 'a'];
            for (int p = u; p > 1 && cnt[p] != -1; p = fail[p]) {
                ans += cnt[p];
                cnt[p] = -1;  // 若题只要「出现与否」，标记避免沿 fail 反复加
            }
        }
        return ans;
    }
};
```

**出现与否 vs 出现次数：** 上面 `cnt=-1` 适合「每种模式最多计一次」。要真实次数，不要置 -1，改为沿 fail 加，但可能 O(文本 × 链长) 被卡。优化：建 fail 树，把「经过」差到树上再 DFS 累加——进阶。入门题 n 小或只要存在，先暴力沿 fail。

缺边填成 `ch[fail[u]][i]` 后，匹配时不必 while 跳 fail，像把 KMP 的 while 预计算了。

### 3. 和 KMP、哈希怎么选

| 需求 | 选 |
| --- | --- |
| 单模式 | KMP |
| 多模式是否出现 / 计数（规模允许） | AC 自动机 |
| 任意子串相等 | 哈希 |
| 模式极多、只要集合查询 | 哈希 set 或 AC |

### 4. 复杂度直觉

建 Trie O(模式总长)。BFS fail O(总结点 × 26)。匹配 O(文本) 若缺边已填好；沿 fail 统计另计。空间 O(结点 × 字符集)。字符集大改 map，常数差。

## 常见坑 / 注意事项

- **根是 0 还是 1。** 缺边指向根，0 容易和「没有儿子」混。  
- **fail[根儿子] 不是根。**  
- **统计只加当前 u 的 cnt，不加 fail 链上的。** 模式是当前后缀时漏计。  
- **`cnt=-1` 用在「要次数」的题。**  
- **动态数组没开够 tot。**  
- **模式重复插入。** `cnt++` 或按编号去重，看题。  
- **大写、数字。** 字符集不是 26。

## 趣味课间：实验室经典翻车集

- **「AC 自动机是自动 AC。」** 名字来自三位作者，不是 Accepted。  
- **样例一个模式。** 和 KMP 分不出差别。造两个模式，一个是另一个真后缀。  
- **query 把 cnt 改成 -1 后第二组文本。** 多组要重建或拷贝 cnt。  
- **内存 256×1e6 开在栈上。** 全局或 heap。

## 小结

多模式 → Trie。fail 像 KMP 的 next，BFS 建。文本沿转移走，统计沿 fail 链（或 fail 树）。单模式退回 KMP。

建议练习：

- 洛谷 [P3808 【模板】AC 自动机（简单版）](https://www.luogu.com.cn/problem/P3808)
- 洛谷 [P3796 【模板】AC 自动机（加强版）](https://www.luogu.com.cn/problem/P3796)
- 两个模式 `she`、`he`，文本 `sheshe`，手画 fail

## 参考资料

- [KMP](KMP.md)
- [字符串哈希](字符串哈希.md)
- [OI Wiki：AC 自动机](https://oi-wiki.org/string/ac-automaton/)
