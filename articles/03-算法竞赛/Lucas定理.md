---
title: Lucas：模质数的组合数，拆成 p 进制
author: TUST-ACM 算法实验室
date: 2026-09-30
category: 03-算法竞赛
tags: [Lucas, 组合数, 逆元, 数论, 算法]
level: 入门
status: 已完成
---

# Lucas：模质数的组合数，拆成 p 进制

> 读完你应该能：在质数模 p 下预处理阶乘和逆元，算出 `C(n,m) mod p`（n,m < p）；用 Lucas 把 `n,m` 写成 p 进制，逐位相乘；知道 p 不是质数时这套定理不能用，要换 [扩展欧几里得](扩展欧几里得.md) 或分解模数。
>
> 一句口号：**大组合数对质数取模，就是每一位的小组合数乘起来。**

## 背景 / 为什么要读这篇

[数论入门](数论入门.md) 里组合数是 `n! / k! / (n-k)!`，n=1e6、模 1e9+7 时预处理阶乘就够。题若 n、m 到 1e18，模数 p 是 1e5 量级的**质数**，阶乘预处理到 1e18 不可能。Lucas 定理把问题收成「小于 p 的组合数」：

```text
C(n, m) ≡ Π C(n_i, m_i)  (mod p)
```

其中 `n_i`、`m_i` 是 n、m 的 p 进制各位。某一位 `m_i > n_i` 时，那一位组合数是 0，整个答案是 0。

实验室有人 p 不质数仍用费马求逆元；有人 `C(n%p, m%p)` 在 `m%p > n%p` 时访问负的阶乘下标。本篇先写小组合数，再递归 Lucas。

## 正文

### 1. p 以内的组合数

p 是质数。预处理 `fact[0..p-1]` 和 `invfact`。逆元用费马 `a^{p-2}`，因为 p 是质数且 `a` 不被 p 整除。阶乘里 `fact[i]` 在 i≥p 时是 0，所以**只预处理到 p-1**，Lucas 的每一位本来就 `< p`。

```cpp
long long fact[100005], invf[100005], MOD;

long long qpow(long long a, long long b) {
    long long r = 1;
    for (; b; b >>= 1, a = a * a % MOD)
        if (b & 1) r = r * a % MOD;
    return r;
}

void init(long long p) {
    MOD = p;
    fact[0] = 1;
    for (int i = 1; i < p; i++) fact[i] = fact[i - 1] * i % MOD;
    invf[p - 1] = qpow(fact[p - 1], p - 2);
    for (int i = (int)p - 2; i >= 0; i--) invf[i] = invf[i + 1] * (i + 1) % MOD;
}

long long Csmall(int n, int m) {
    if (m < 0 || m > n) return 0;
    return fact[n] * invf[m] % MOD * invf[n - m] % MOD;
}
```

`m>n` 必须返回 0，不能让 `invf` 下标为负。

### 2. Lucas

```cpp
long long lucas(long long n, long long m) {
    if (m == 0) return 1;
    return Csmall((int)(n % MOD), (int)(m % MOD)) * lucas(n / MOD, m / MOD) % MOD;
}
```

`m==0` 时结果是 1，包括 `n==0`。递归深度是 log_p(n)，1e18 对 p≥2 只有六十层左右。

手算：p=3，`C(5,2)=10 ≡ 1`。5 的三进制是 (1,2)，2 是 (0,2)。`C(1,0)*C(2,2)=1*1=1`。对上了。

### 3. 什么时候不要用

| 模数 | 做法 |
| --- | --- |
| 质数，n < p，p≤1e6 | 阶乘，不必 Lucas |
| 质数，n 很大，p≤1e6 | Lucas |
| 质数但 p > 1e6 且 n 也大 | 阶乘开不下，Lucas 的 `Csmall` 也不能 O(p) 预处理；按位用乘法公式现算，位数很少 |
| 合数 | Lucas 不成立。模数不大可分解后 CRT；或用扩展 Lucas（另一篇，更长） |

`C(n,m)` 不取模、只要整数值：n≤60 用 `__int128` 乘除约分，或 DP `C[i][j]=C[i-1][j]+C[i-1][j-1]`。不要对合数模调用上面的 `qpow(..., p-2)`。

## 常见坑 / 注意事项

- **`Csmall` 不判断 `m>n`。** Lucas 必然会问到这一位。  
- **`fact` 开到 n=1e18。** 只能开到 p。  
- **逆元从 1 乘到 p-1，但 `fact[p-1]` 若 p 很大非常数。** p≤1e6 才预处理。  
- **多测 p 变了仍用旧阶乘。** 换模数要 `init`。  
- **`n%MOD` 用 int 接 1e18。** 先对 `long long` 取模再转。  
- **答案要组合数本身，却只输出模 p。** 读题。

## 趣味课间：实验室经典翻车集

- **「组合数就是杨辉三角。」** n=1e18 的三角形画不下。Lucas 是质数模下的压缩。  
- **样例 p 大于 n。** Lucas 退化为一次 `Csmall`，测不出进制拆分。用上面的 p=3、C(5,2)。  
- **p=4 时费马仍然能跑出数。** 那个数不是组合数。先证 p 是质数，或相信题面。  
- **和 CRT 叠在一起，每一层 Lucas 用了不同的 p 却共用 `fact`。** 每个质因子单独 `init`。

## 小结

p 质数且不大：阶乘到 p-1。`C(n,m) mod p` 等于 p 进制每一位的小组合数之积。某一位 m 更大就是 0。合数模不要用这一定理。

建议练习：

- 洛谷 [P3807 【模板】卢卡斯定理](https://www.luogu.com.cn/problem/P3807)
- 手算 p=5、C(6,2)：6=(1,1)，2=(0,2)，应为 0（因为 C(1,2)=0），而 15÷5 余 0，对得上  
- 换一个 p > n 的数据，确认和直接阶乘一致

## 参考资料

- [数论入门](数论入门.md)
- [扩展欧几里得](扩展欧几里得.md)
- [中国剩余定理](中国剩余定理.md)
- [OI Wiki：Lucas](https://oi-wiki.org/math/number-theory/lucas/)
