---
title: KMP：模式串在文本里跳着找
author: TUST-ACM 算法实验室
date: 2026-09-17
category: 03-算法竞赛
tags: [KMP, 字符串, next, 失配, 算法]
level: 入门
status: 已完成
---

# KMP：模式串在文本里跳着找

> 读完你应该能：求出模式串的 `next`（最长真前后缀）；用它在文本里 O(n+m) 匹配；解释失配时为什么跳到 `next[j]` 而不是退回 0；知道 `next` 数组 0-based 和 1-based 不要混。
>
> 一句口号：**暴力匹配失败就退回模式串开头；KMP 失败只退到「已经匹配的那截后缀还能当谁的前缀」。**

## 背景 / 为什么要读这篇

[string 读入](../01-编程语言/string读入.md) 之后，`s.find(t)` 能用。题面要的是所有出现位置、或自己实现、或「周期 / 前后缀」——那就露出 `next` 数组。

暴力：文本每个起点对模式扫，最坏 O(nm)。KMP 预处理模式 O(m)，匹配 O(n)。n,m=1e6 必须它（或哈希，哈希有冲突风险）。

实验室有人 `next` 求错，匹配「碰巧」过样例 `aaa`；有人失配写 `j=0` 当成暴力。本篇把前后缀讲成拉链，不背公式。

## 正文

### 1. 真前后缀

字符串 `p` 的**真前缀**不含自己。`abab` 的真前缀：`a, ab, aba`；真后缀：`b, ab, bab`。最长的相同真前后缀是 `ab`，长度 2。

`next[i]` = `p[0..i]` 这一段的最长真前后缀长度（有的教材 `next[i]` 对应 `p[0..i-1]`，差一位。**全文用：`next[i]` 表示以 i **结尾**的前缀 `p[0..i]` 的最长真前后缀长度。** `next[0]=0`。

求 `next` 本身也是匹配：模式自己和自己比。

### 2. 求 next

```cpp
vector<int> build_next(const string& p) {
    int m = (int)p.size();
    vector<int> nxt(m, 0);
    for (int i = 1, j = 0; i < m; i++) {
        while (j > 0 && p[i] != p[j]) j = nxt[j - 1];
        if (p[i] == p[j]) j++;
        nxt[i] = j;
    }
    return nxt;
}
```

`j` 是「当前已经匹配的前后缀长度」。`p[i]` 接不上就 `j = nxt[j-1]` 缩短，直到接上或到 0。和匹配过程同一套 while。

手算 `p = ababaca`：

```text
i    0 1 2 3 4 5 6
p    a b a b a c a
nxt  0 0 1 2 3 0 1
```

`p[0..4]=ababa` 最长真前后缀 `aba` 长度 3。不要背，自己走一遍 `j`。

### 3. 在文本里匹配

```cpp
vector<int> kmp(const string& s, const string& p) {
    auto nxt = build_next(p);
    vector<int> pos;
    int n = (int)s.size(), m = (int)p.size();
    for (int i = 0, j = 0; i < n; i++) {
        while (j > 0 && s[i] != p[j]) j = nxt[j - 1];
        if (s[i] == p[j]) j++;
        if (j == m) {
            pos.push_back(i - m + 1);  // 出现的起始下标
            j = nxt[j - 1];            // 继续找下一个（允许重叠）
        }
    }
    return pos;
}
```

失配不把 i 回退，只缩 j。所以 i 扫过全文一次。重叠：`aaa` 里找 `aa` 会出现 0 和 1，靠 `j = nxt[j-1]` 而不是 `j=0`。

### 4. next 的其他用法

- **最小周期：** 若 `n % (n - next[n-1]) == 0`，最小周期长度是 `n - next[n-1]`（整周期重复时）。`ababab` 的 next 末尾 4，周期 2。不是所有串都整周期，要先判断整除。
- **前缀函数：** 有人把整个 `next` 叫 π。读题、读博客时对一下下标。
- **1-based 模板：** `next[1]=0`，`p` 从 1 存。和 0-based 不要拼在同一份代码。

字符串哈希（滚动哈希）也能匹配，写起来短，存在被卡。模板题、被卡风险高时用 KMP。两者都会更好。

### 5. 复杂度

预处理 O(m)，匹配 O(n)。均摊：j 增加不超过 n 次，while 减少也不超过。空间 O(m)。

## 常见坑 / 注意事项

- **`nxt[j]` 写成 `nxt[j-1]` 漏了或反了。** 0-based 失配是 `nxt[j-1]`。
- **`j==m` 后 `j=0`。** 漏重叠。
- **模式空串。** 特判。
- **`next` 求的是文本不是模式。** 预处理永远针对 p。
- **下标输出 1-based 题面。** `pos+1`。
- **`string` 很长、递归 KMP。** 不要递归。
- **把 `find` 当 KMP 交题。** 课上/模板题要自己写；赛时能 `find` 且只要任意一次出现，STL 可以。

## 趣味课间：实验室经典翻车集

- **「我暴力 n=1000 过了。」** 模板 n=1e6。
- **`aaa` 与 `aa`。** 不重叠版少一个答案。
- **next 画成「失配后到第几个」却用成长度。** 长度是 j，下一个要比的是 `p[j]`。
- **KMP 和 Automaton（AC 自动机）搞混。** 多模式是 AC。单模式先 KMP。

## 小结

`next[i]` 是前缀 `p[0..i]` 的最长真前后缀长度。匹配失败让 j 缩到 `next[j-1]`，i 不回头。求 next 与匹配是同一段 while。

建议练习：

- 洛谷 [P3375 【模板】KMP](https://www.luogu.com.cn/problem/P3375)
- 洛谷 [P4391 [BOI2009] Radio Transmission](https://www.luogu.com.cn/problem/P4391) 或周期相关
- 手算 `ababaca` 的 next，对照程序

多模式：[AC 自动机入门](AC自动机入门.md)。和整串前缀比：[Z 函数](Z函数.md)。哈希：[字符串哈希](字符串哈希.md)。

## 参考资料

- [string 读入](../01-编程语言/string读入.md)
- [OI Wiki：KMP](https://oi-wiki.org/string/kmp/)
- [OI Wiki：前缀函数](https://oi-wiki.org/string/kmp/)
