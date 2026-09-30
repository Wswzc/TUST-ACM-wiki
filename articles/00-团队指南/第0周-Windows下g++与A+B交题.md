---
title: 第 0 周实操：Windows 下 g++ 最短环境与 A+B 交题
author: TUST-ACM 算法实验室
date: 2026-09-01
category: 00-团队指南
tags: [g++, Windows, 环境配置, A+B, 洛谷]
level: 入门
status: 已完成
---

# 第 0 周实操：Windows 下 g++ 最短环境与 A+B 交题

> 读完你应该能：在 Windows 终端里用 `g++` 编译一个文件，用样例测通，并在洛谷拿到 A+B 的 Accepted。不装 Visual Studio，不配一周环境。

## 背景 / 为什么要读这篇

[新人指南](新人指南.md) 第 0 周的目标是「活下来」：环境能跑、能交一道题。很多人卡在第一步——下了几十 GB 的 Visual Studio，只会按绿色三角，换一台机房电脑就不会了。

ICPC / CCPC 现场是 **g++ 命令行编译**。第 0 周就把这条路径走通，后面刷题、对拍、赛时才不会慌。

本篇只覆盖 **Windows**。已经会终端编译的人可以跳过，直接去做下一题。

## 正文

### 1. 我们要的是 g++，不是 Visual Studio

| 要 | 不要（第 0 周） |
| --- | --- |
| MinGW 系的 `g++`（和评测机同一路） | 只为刷题去装完整 Visual Studio |
| 终端里敲编译命令 | 只会点 IDE 的运行按钮 |
| 一个解压即用的工具包 | 配一周插件、主题、代码补全 |

课内大作业以后用 VS / CLion 都可以。竞赛训练先把 `g++` 打通。

### 2. 最短安装：w64devkit

推荐 [w64devkit](https://github.com/skeeto/w64devkit/releases)：一个 zip，自带 `g++`、`gdb`、`make`，解压就能用。不必装 MSYS2，也不必碰安装向导。

1. 打开 Releases，下载最新的 `w64devkit-x.x.x.zip`（普通 Intel / AMD 电脑选这个；ARM 电脑选带 `arm64` 的包）。
2. 解压到一个**没有空格、没有中文**的路径，例如 `C:\w64devkit`。不要放在桌面「新建文件夹」里。
3. 双击解压目录里的 `w64devkit.exe`，会弹出一个黑色终端。以后编译都在这个窗口里做。

在窗口里执行：

```text
g++ --version
```

能看到类似 `g++ (GCC) 14.x.x` 就成功了。若提示不是内部或外部命令，说明你打开的是系统自带的 PowerShell / CMD，而不是 `w64devkit.exe`。关掉，重新双击那个 exe。

可选：把 `C:\w64devkit\bin` 加到系统环境变量 `Path`，之后任意终端都能用 `g++`。第 0 周**不加也可以**，用 `w64devkit.exe` 照样交题。

> 备选：已经装了 MSYS2 或 VS Code 自带的 MinGW，只要 `g++ --version` 能出来，就不要再装第二套。一套能跑就停。

### 3. 写 A+B

在 `C:\acm` 建一个文件夹（同样避免中文路径）。用记事本、VS Code 或 Cursor 新建 `ab.cpp`：

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

保存为 **UTF-8**。有中文注释时不要用「UTF-8 带 BOM」，部分评测机会 CE。

`bits/stdc++.h` 是 GCC 竞赛常用的万能头。MSVC（Visual Studio 的编译器）经常没有这个头——这也是第 0 周不用 MSVC 交题的原因之一。

### 4. 终端编译

在 w64devkit 窗口里进入目录（路径按你的实际位置改）：

```bash
cd /c/acm
g++ -std=c++17 -O2 -Wall ab.cpp -o ab
```

含义：

| 参数 | 作用 |
| --- | --- |
| `-std=c++17` | 和多数 ICPC 赛制接近 |
| `-O2` | 优化，正式赛常用 |
| `-Wall` | 打开警告，第 0 周就养成看警告的习惯 |
| `-o ab` | 生成 `ab.exe` |

没有报错时，目录下会出现 `ab.exe`。有报错：把**完整命令 + 完整英文报错**留下来，再去问人。不要只发一句「编译不过」。

### 5. 本地用样例测

洛谷 [P1001 A+B Problem](https://www.luogu.com.cn/problem/P1001) 的样例一般是：输入 `20 30`，输出 `50`。

先做输入文件。在 `C:\acm` 里新建 `in.txt`，内容只有一行（行尾回车即可）：

```text
20 30
```

**不要在 PowerShell 里写 `.\ab.exe < in.txt`。** `<` 在 PowerShell 里不是重定向，会报解析错误。用下面任一方式：

```bash
# 就在 w64devkit / 普通 CMD 里（推荐）
./ab < in.txt
```

如果人已经在 PowerShell 里：

```powershell
cmd /c "ab.exe < in.txt"
```

或：

```powershell
Get-Content in.txt | .\ab.exe
```

屏幕上出现 `50` 就对了。测的是「标准输入」，不是自己在窗口里敲一遍再回车——正式评测也是把文件喂给程序，不会等人交互。

交互着运行也可以：直接 `./ab`，输入 `20 30`，回车，看到 `50`。交题前至少用文件重定向测一次，避免以后多组数据不会测。

### 6. 交到洛谷

1. 打开 [洛谷](https://www.luogu.com.cn/)，注册并登录。
2. 进入 [P1001](https://www.luogu.com.cn/problem/P1001)，点提交。
3. 语言选 **C++14 / C++17 / C++20** 均可，与你本地 `-std=` 对齐即可；不要选 C、不要选 Pascal。
4. 把 `ab.cpp` **全文**贴进提交框（或上传文件），提交。

评测结果怎么读：

| 结果 | 含义 | 第 0 周怎么处理 |
| --- | --- | --- |
| Accepted (AC) | 过了 | 第 0 周结束。去刷下一道入门题 |
| Compile Error (CE) | 编译失败 | 对照本地 `g++` 报错；是否用了 MSVC 语法、是否选错语言 |
| Wrong Answer (WA) | 输出不对 | 本地样例是否真的测过；多空格、没换行、调试输出忘删 |
| Runtime Error (RE) | 运行崩溃 | 第 0 周的 A+B 几乎不该出现；检查是否提交了空文件 |
| Time Limit Exceeded (TLE) | 超时 | A+B 不该超时；可能死循环或选错题 |

AC 之前不要开始装主题、配 snippet、下十个题单。

## 常见坑 / 注意事项

- **打开了错误的终端。** 系统开始菜单里的 PowerShell 默认没有 `g++`。双击 `w64devkit.exe`，或确认 `Path` 里有 `...\w64devkit\bin`。
- **路径有中文或空格。** `C:\用户\张三\桌面\新建文件夹` 会在后续脚本、重定向里埋雷。用 `C:\acm`。
- **源文件其实是 `.cpp.txt`。** 记事本保存时把扩展名露出来，关掉「隐藏已知文件类型的扩展名」。
- **用 Visual Studio 的「本地 Windows 调试器」交题。** 那是 `cl.exe`，评测机是 `g++`。两边过的代码可以不一样。
- **PowerShell 的 `<`。** 见上文，改用 w64devkit / CMD，或 `cmd /c "ab.exe < in.txt"`。
- **提交时语言选成 C。** `iostream`、`bits/stdc++.h` 会直接 CE。
- **输出多了调试信息。** `cerr` 评测一般看不见；`cout` 里的 `debug:` 会 WA。
- **环境配一天还没交题。** 允许简陋。`g++ --version` 能出、样例能对、洛谷能 AC，就停。

## 小结

第 0 周只验证三件事：`g++` 在终端里存在、本地能用文件喂样例、OJ 上 A+B 是 AC。编辑器美不美观以后再说。

建议接着做：

- 打开 [多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md)，把 T 组、EOF、哨兵三种循环交通
- 样例不够用时上 [对拍](对拍.md)
- 洛谷入门题单里再交 3～5 道纯输入输出题（看清多组数据、读到文件结束等）
- 回到 [新人指南](新人指南.md) 的「第 1 学期」：STL 与基础算法
- 把你这台电脑上多出来的坑（例如某版 Win11 杀毒软件拦 `ab.exe`）补进本篇或另写一篇短文
- 上场前的编译器和语言标准，对照 [2026 赛场环境](../06-前沿动态/2026赛场环境.md)，不要默认本机 CLion 等于评测机

## 参考资料

- [w64devkit Releases](https://github.com/skeeto/w64devkit/releases)
- [洛谷 P1001 A+B Problem](https://www.luogu.com.cn/problem/P1001)
- [OI Wiki：使用 GCC](https://oi-wiki.org/lang/gcc/)
- [TUST-ACM 算法实验室新人指南](新人指南.md)
- [对拍](对拍.md)
- [2026 赛场环境](../06-前沿动态/2026赛场环境.md)
