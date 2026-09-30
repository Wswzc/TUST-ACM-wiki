---
title: 多组输入与 EOF：A+B 之后第一道坎
author: TUST-ACM 算法实验室
date: 2026-09-01
category: 03-算法竞赛
tags: [输入输出, EOF, cin, 多组数据, 竞赛]
level: 入门
status: 已完成
---

# 多组输入与 EOF：A+B 之后第一道坎

> 读完你应该能：打开题面「输入格式」，判断这题是单组、T 组、读到结束，还是读到 `0 0` 这种哨兵；本地用文件测通，而不是在窗口里盲敲。

## 背景 / 为什么要读这篇

[第 0 周](../00-团队指南/第0周-Windows下g++与A+B交题.md) 的洛谷 P1001 只有一组 `a b`。下一道题往往还是 A+B，评测却变成 WA 或卡住——不是加法错了，是**没按评测机的方式把数据读完**。

竞赛程序不是「运行一次、问你一次、打印一次」。评测机会把整个输入文件重定向进来；你必须在程序里循环读，读到题目规定的结束条件为止。

## 正文

### 先读题，再选模板

动手前只问一件事：**输入什么时候结束？** 常见就四种。题面写哪种，就用哪段代码，不要凭记忆混用。

| 类型 | 题面常见说法 | 怎么读 |
| --- | --- | --- |
| 单组 | 「一行两个整数」 | 读一次 |
| 先给 T | 「第一行一个整数 T，接下来 T 行」 | `cin >> t; while (t--)` |
| 读到文件结束 | 「多组数据，直到输入结束 / EOF」 | `while (cin >> a >> b)` |
| 哨兵结束 | 「当 a、b 都为 0 时结束，该组不处理」 | 读进去，判断再 break |

下面四段都可以直接编译。测的时候用文件，不要只靠键盘。

### 1. 单组（你已经会了）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int a, b;
    cin >> a >> b;
    cout << a + b << '\n';
    return 0;
}
```

### 2. 第一行是组数 T

样例长这样：

```text
2
1 2
5 5
```

应对输出两行：`3` 和 `10`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;
    while (t--) {
        int a, b;
        cin >> a >> b;
        cout << a + b << '\n';
    }
    return 0;
}
```

`t--` 是「还剩几组」；组数为 0 时循环直接不进，这是合法输入，不要特判崩溃。

### 3. 读到 EOF

样例可能是任意多行，文件到头就停：

```text
1 2
5 5
100 200
```

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int a, b;
    while (cin >> a >> b) {
        cout << a + b << '\n';
    }
    return 0;
}
```

`cin >> a >> b` 在读失败（文件结束、或根本不是整数）时变成假，循环停止。这就是「读到 EOF」。

**不要在交互运行时干等。** 你在窗口里用手输入，程序不知道你「输完了」。Windows 下要先回到行首，按 `Ctrl+Z` 再回车，才会送出 EOF；w64devkit / Linux 一般是 `Ctrl+D`。第 0 周已经会用文件重定向的话，直接：

```bash
./ab < in.txt
```

文件一读完自然结束，不必按快捷键。

### 4. 哨兵：读到 `0 0` 停止

样例：

```text
1 2
5 5
0 0
```

最后一组不算，只输出 `3` 和 `10`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int a, b;
    while (cin >> a >> b) {
        if (a == 0 && b == 0) break;
        cout << a + b << '\n';
    }
    return 0;
}
```

先读再判断。写成 `while (cin >> a >> b && (a || b))` 也可以，但遇到「只有一个为 0 仍要算」的题会翻车，不如按题意写 `if`。

### 本地怎么测多组

`in.txt` 原样粘贴样例，包含多行。w64devkit / CMD：

```bash
g++ -std=c++17 -O2 -Wall ab.cpp -o ab
./ab < in.txt
```

把屏幕输出和题面「输出」逐行对：行数、空行、最后有没有多余的 `0`。样例过了再交；样例不过，交上去几乎必 WA。

### 和 cin 有关的两个开关

第 0 周模板里的两行建议保留：

```cpp
ios::sync_with_stdio(false);
cin.tie(nullptr);
```

含义很窄：关掉 C 和 C++ 两套输入的同步、解开 `cin` 和 `cout` 的绑定，读得更快。代价是：**不要再混用 `scanf` / `printf` 和 `cin` / `cout`。** 入门期选一条路走到底。竞赛里 `cin` 加这两行，对绝大多数入门题足够。

## 常见坑 / 注意事项

- **把 T 组写成单组。** 样例第一行碰巧也是两个数时，本地「好像对了」，评测后面还有几百组，结果少输出或乱读。
- **EOF 题只读一行。** 评测文件有 1000 行，你只加了一次，答案不完整 → WA。
- **哨兵组也拿去算。** `0 0` 输出了 `0`，多一行 → WA。
- **交互测试 EOF 题，程序不退出。** 评测机有文件结尾；你的键盘没有。用 `< in.txt`，或 `Ctrl+Z` / `Ctrl+D`。
- **`while (cin >> a)` 漏了第二个数。** 两个整数必须两个都读成功，用 `cin >> a >> b`。
- **输出格式。** 有的题「每组之间空一行」，最后一组后面不能空行。样例有空行就照抄，不要发明格式。
- **用 `endl` 刷缓冲区。** 对 A+B 无所谓；数据量上来后改 `'\n'`。现在养成即可。
- **题面写了多组，你按单组交过了样例。** 样例往往只有一组。以文字描述为准，不以样例行数为准。

## 小结

A+B 会了以后，真正决定 AC 还是 WA 的经常是输入循环。四种模板够应付入门阶段几乎所有「输入输出练习」。选模板的依据永远是题面，不是上一次交过的代码。

建议接着做：

- 把上面 2、3、4 各找一道题交掉（HDU 1090～1096 就是专门练这个的；洛谷则盯每题的「输入格式」）
- 打开 [vector 与 sort：最小刷题习惯](../01-编程语言/vector与sort.md)
- 输入已经不卡手之后，回到 [新人指南](../00-团队指南/新人指南.md) 的第 1 学期

## 参考资料

- [第 0 周实操：Windows 下 g++ 与 A+B 交题](../00-团队指南/第0周-Windows下g++与A+B交题.md)
- [OI Wiki：输入输出](https://oi-wiki.org/contest/io/)
- HDU 1000 / 1090～1096：经典 A+B 输入输出练习（题号以当时题面为准）
