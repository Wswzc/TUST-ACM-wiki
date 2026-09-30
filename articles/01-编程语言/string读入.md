---
title: string 读入：空格、整行，以及和数字混排
author: TUST-ACM 算法实验室
date: 2026-09-16
category: 01-编程语言
tags: [C++, string, getline, 输入输出, STL]
level: 入门
status: 已完成
---

# string 读入：空格、整行，以及和数字混排

> 读完你应该能：看见题面里的「单词 / 整行 / 先 n 再若干行姓名」，立刻选 `cin >> s` 还是 `getline`；知道为什么「先读整数再读一行」会读到空字符串；会用 `find` / `substr` / `stoi` 做入门题，而不是把字符串当 `char a[100]` 手搓到凌晨。

## 背景 / 为什么要读这篇

数字的输入坑，[多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md) 已经收拾过了。字符串是下一只妖怪，而且它更会装成「我本地明明对了」。

典型案发现场：

```text
3
Alice
Bob Smith
Carol
```

你写 `cin >> n` 然后循环 `getline`。第一行姓名读到的是……空的。`Bob Smith` 被劈成 `Bob` 和 `Smith` 两个队员。评测机不解释，只丢 WA。

实验室每年都有人在签到题上栽这跟头。不是算法，是**空格和换行在 `cin` 眼里根本不是一回事**。本篇把这层窗户纸撕掉，再把刷题真正用得到的 `string` 接口摊开。

## 正文

### 1. 先建立世界观：空白符是刀

C++ 里和字符串有关的读入，本质只有两把刀：

| 刀 | 停在哪里 | 典型题面 |
| --- | --- | --- |
| `cin >> s` | 空白（空格、Tab、换行） | 「由小写字母组成的单词」「不含空格的字符串」 |
| `getline(cin, s)` | 换行 `\n` | 「一整行」「可能含空格的姓名 / 句子」 |

`cin >> s` 会**跳过开头的空白**，再读到下一块空白之前。所以它读不到空格，也读不到「空行」——空行在它看来只是更多空白，继续往下吃。

`getline` **不跳过**已经等在那儿的换行。当前面刚用 `>>` 读完一个整数，那个整数后面的 `\n` 还在缓冲区里，`getline` 会立刻收工，给你一个空串。这就是开头那个鬼故事的全部科学解释。

口诀：**要单词用 `>>`，要整行用 `getline`，两者交接处记得把换行扔掉。**

### 2. 三种合法读法（各配一份能交的代码）

#### 2.1 不含空格：`cin >> s`

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<string> a(n);
    for (int i = 0; i < n; i++) cin >> a[i];
    // 每个 a[i] 是一个单词
    return 0;
}
```

多组直到 EOF：`while (cin >> s)`，和读整数同一套路。

#### 2.2 整行：`getline`

```cpp
string line;
getline(cin, line);  // 读到 '\n'，'\n' 被吃掉，不进 line
```

一行里既有数字又有句子，常见写法是：**先 `>>` 读完所有「严格按空白分开」的字段，再决定要不要换 `getline`。** 不要在同一层逻辑里随机切换。

#### 2.3 先 n 再 n 行（含空格）——入门第一杀手

错误：

```cpp
int n;
cin >> n;
vector<string> name(n);
for (int i = 0; i < n; i++) getline(cin, name[i]);
// name[0] 往往是空串，后面全部错位
```

缓冲区此刻长这样（`|` 表示还没读的部分）：

```text
3\nAlice\nBob Smith\nCarol\n
 ↑
 cin >> n 停在这里，'\n' 还在
```

紧接着的 `getline` 看到 `\n`，读到空串，指针才走到 `Alice`。于是 `Alice` 变成第二个名字，最后一行没人读。

两种修法，记住一种即可。

**修法 A：扔掉那一个换行**

```cpp
int n;
cin >> n;
cin.ignore(numeric_limits<streamsize>::max(), '\n');  // 丢掉本行剩余，含 '\n'
vector<string> name(n);
for (int i = 0; i < n; i++) getline(cin, name[i]);
```

入门也可以粗暴一点：`string dummy; getline(cin, dummy);` 专门吞掉 `3` 后面那半截。

**修法 B：混用时先把 n 也放进一行里拆**

```cpp
string first;
getline(cin, first);
int n = stoi(first);
vector<string> name(n);
for (int i = 0; i < n; i++) getline(cin, name[i]);
```

全程 `getline`，没有「`>>` 残留换行」这回事。题面保证第一行只有一个整数时，这很干净。

### 3. string 当数组用：下标是字节，不是「第几个汉字」

```cpp
string s = "abc";
s.size();     // 3
s.empty();    // false
s[0];         // 'a'，类型是 char
s.back();     // 'c'
s.push_back('d');
s.pop_back();
```

下标从 0 到 `s.size()-1`。空串不要 `s[0]`，也不要 `s.back()`。

**中文陷阱。** 洛谷、Codeforces 的字符串题绝大多数是 ASCII（字母、数字、符号）。你若在 Windows 记事本里写 `string s = "天津";` 再对 `s[0]` 改大小写，改的是 UTF-8 的某一个字节，不是「天」这个字。竞赛入门**不要用下标遍历汉字**。处理中文题面之前，先确认编码；本篇默认 ASCII。

`s[i]` 和 `char` 比较时，写成 `s[i] == 'a'`，不要 `s[i] == "a"`——后者是字符串，类型都不对，能编译也不是你想的意思（指针比较）。

大小写：

```cpp
char to_lower(char c) {
    if (c >= 'A' && c <= 'Z') return (char)(c - 'A' + 'a');
    return c;
}
```

`<cctype>` 里的 `tolower` 要注意：它吃的是 `unsigned char` 的值。直接 `tolower(s[i])` 在 `char` 为负时是未定义行为。ASCII 范围内自己减 `'A'` 更省心，也更好记。

### 4. 拼接：`+` 很甜，循环里很贵

```cpp
string a = "TUST";
string b = "-ACM";
string c = a + b;       // "TUST-ACM"
c += " 算法实验室";     // 追加
```

`s = s + t` 每次可能复制整个 `s`。在循环里把 n 段拼起来：

```cpp
string ans;
for (int i = 0; i < n; i++) ans = ans + a[i];  // 最坏 O(n²)
```

改成 `ans += a[i];`，或先 `ans.reserve(总长度)`。n = 1e5、每段不短时，这个二次方会从「能过样例」变成 TLE。现场赛有人用 `+` 拼出一篇作文，最后两分钟对着 TLE 发呆——算法是对的。

### 5. 查找与切片：`find` / `npos` / `substr`

```cpp
string s = "banana";
auto p = s.find("ana");
if (p == string::npos) {
    // 没有
} else {
    // p == 1，第一次出现的下标
}
```

**`npos` 不是 -1 的文学比喻，它就是 `string::npos`。** 写成 `if (p == -1)` 有时能混过（无符号绕一圈），有的编译警告会骂你。用 `npos`。

`find` 第二个参数是起始下标，适合「找下一个」：

```cpp
int cnt = 0;
for (size_t p = s.find("ana"); p != string::npos; p = s.find("ana", p + 1)) {
    cnt++;
}
```

注意 `p + 1` 还是 `p + 模式长度`：前者允许重叠（`ana` 在 `anana` 里出现两次），后者不允许。题面没说重叠，先问自己，再写。

切片：

```cpp
s.substr(pos);       // 从 pos 到末尾
s.substr(pos, len);  // 最多 len 个字符
```

`pos` 超出 `s.size()` 会抛异常（RE）。先判断再切。`len` 超出剩余长度则切到末尾，这个倒是宽容。

### 6. 和数字互相转：`stoi` / `stoll` / `to_string`

```cpp
int x = stoi("12");
long long y = stoll("1234567890123");
string t = to_string(2026);  // "2026"
```

`stoi` 会吃掉前导空白，读到第一个不合法字符为止：`stoi("  12abc")` 是 12。空串、`"abc"` 会抛异常。竞赛里数据一般合法；你自己造数据时别坑自己。

高精、超长数字不要 `stoi`，那是字符串题，该按位处理。

**拼数题的味道**（洛谷 [P1012 拼数](https://www.luogu.com.cn/problem/P1012)）：若干正整数，排一个顺序使拼起来最大。这不是按数值 `sort`，是按字符串拼接比较：

```cpp
sort(a.begin(), a.end(), [](const string& x, const string& y) {
    return x + y > y + x;
});
```

`"9"` 和 `"90"`：`990` vs `909`，所以 `"9"` 应该排前面。默认字典序会把 `"90"` 排在 `"9"` 前（因为 `'0' < 结束` 的规则其实是更短的……等等，默认比较是字典序，`"90"` 和 `"9"` 比的是第一个字符 `'9'=='9'`，然后 `"90"` 还有 `'0'`，**更长的更大**？不对：字典序是逐位比，较短的是前缀则较短的更小。`"9" < "90"`。拼数必须用 `x+y` vs `y+x`，不要相信默认顺序。这是字符串题里最经典的「看起来像数字、其实是字符串」的一课。

### 7. `stringstream`：一行里又是数又是词

有的题「一行未知个数的整数」。`cin` 会跨行，不好停。先 `getline` 再拆：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string line;
    getline(cin, line);
    stringstream ss(line);
    vector<int> a;
    int x;
    while (ss >> x) a.push_back(x);
    // a 里是这一行的全部整数
    return 0;
}
```

`stringstream` 和 `cin` 一样用 `>>`，只是数据源是字符串。拆单词同理：`while (ss >> word)`。

不要每读一个字符就 `new` 一个 `stringstream`（有人真写过）。一行建一次，用完即可。

### 8. 排序、翻转、当字符数组扫

```cpp
string s = "cba";
sort(s.begin(), s.end());     // "abc"
reverse(s.begin(), s.end());  // "cba" 再反过来
```

`vector<string>` 的 `sort` 是字典序，和 [vector 与 sort](vector与sort.md) 里排整数是同一套迭代器。字典序不是拼音序，也不是数值序。

判断回文：

```cpp
bool pal(const string& s) {
    int i = 0, j = (int)s.size() - 1;
    while (i < j) {
        if (s[i] != s[j]) return false;
        i++;
        j--;
    }
    return true;
}
```

或 `string t = s; reverse(t.begin(), t.end()); return s == t;` 多一份拷贝，n 大时前者更好。

### 9. 读入加速那两行，对 `getline` 同样生效

```cpp
ios::sync_with_stdio(false);
cin.tie(nullptr);
```

关掉同步之后，**不要再混 `scanf` / `gets` / `getchar` 和 `cin`。** `gets` 已经从 C++ 里除名，缓冲区溢出是 CVE 博物馆藏品，竞赛里也不要写。

`cin >> s` 读超长字符串（1e6）通常没问题。自己用 `s += getchar()` 循环去「优化」，除非你真的量过，否则不要在入门期玩。

### 10. 一张对照表，贴在显示器边上

| 题面怎么说 | 怎么读 |
| --- | --- |
| 一个不含空格的字符串 | `cin >> s` |
| n 个单词，空白分隔 | `for (...) cin >> s` |
| 一整行，含空格 | `getline(cin, s)` |
| 第一行 n，下面 n 行（行内有空格） | `cin >> n;` 然后 `ignore` 掉换行，再 `getline` n 次 |
| 一行里个数不定的整数 | `getline` + `stringstream` |
| 多组，每组一个单词，到 EOF | `while (cin >> s)` |
| 多组，每组一行，到 EOF | `while (getline(cin, s))`，注意可能读到空行，按题意跳过或处理 |

空行：`getline` 会得到 `s.empty()==true` 或 `s == ""`。`>>` 会跳过空行去找下一个单词。题面写「空行表示结束」时必须 `getline`。

## 常见坑 / 注意事项

- **`>>` 之后立刻 `getline`，第一行是空的。** 见第 2.3 节。这是本篇要消灭的头号敌人。
- **`cin >> s` 读带空格的姓名。** 后半截留给下一题当种子，造成「玄学 WA」。
- **`if (s.find(t))` 当布尔用。** `find` 找到开头时返回 0，0 在 `if` 里是假。必须和 `npos` 比。
- **循环里 `s = s + t`。** 改 `+=`。
- **`s[s.size()]` 当哨兵。** C++ 的 `string` 在 `s[s.size()]` 处保证是 `'\0'`（这点像 C 字符串），但写 `s.at(s.size())` 会抛异常。不要依赖下标越界。遍历用 `i < s.size()`。
- **比较 `char` 和 `"a"`。** 用 `'a'`。
- **中文按下标改大小写。** ASCII 题用下标；汉字当整体，不要切片切到半个 UTF-8 里去。
- **`stoi("")` / 前导加号。** 合法数据一般没有；自己测时会炸。
- **输出末尾多空格、多换行。** 字符串题比 A+B 更爱卡格式。样例有几行就几行。
- **`string dummy; cin >> dummy;` 想吞掉换行。** `>>` 会跳过换行去读下一个单词，越帮越忙。吞换行用 `ignore` 或 `getline`。

## 趣味课间：实验室经典翻车集

- **「样例是 Alice，我输出了 Alice，为什么 WA？」** 因为你 `getline` 到了空串，Alice 进了第二组，评测数据有 3 行，你的错位在样例里看不出来——样例第一行如果碰巧没有空格姓名，更看不出来。加一行 `cerr << "[" << name[i] << "]\n";` 本地看方括号里有没有空的。提交前删掉。
- **「我用了 `gets`，CE。」** 现代评测机的 C++ 没有它。当文物即可。
- **「`find` 返回 0 我当成没找到。」** 单词出现在句首是光荣的 0，不是失败。
- **「Python 的 `input()` 多好。」** 是挺好。ICPC 现场默认 C++。在 C++ 里把 `getline` 当 `input()` 用，把 `>>` 当 `split()` 用，世界就清静了。
- **Windows 记事本保存的 `in.txt` 带 BOM。** 第一行字符串开头多三个看不见的字节，`stoi` 或比对样例失败。用 VS Code / Cursor 存 UTF-8 无 BOM。这和 [第 0 周](../00-团队指南/第0周-Windows下g++与A+B交题.md) 是同一条河。

## 小结

字符串题一半是读入，一半才是 `find` / 排序 / 模拟。读入只问一句：空白要不要当数据？不要，就 `>>`；要，就 `getline`；两种刀换手，先把换行扔掉。

建议练习：

- 洛谷 [P5733 自动修正](https://www.luogu.com.cn/problem/P5733)：大小写，练下标扫字符
- 洛谷 [P1308 统计单词数](https://www.luogu.com.cn/problem/P1308)：整行、查找、大小写，几乎是本篇模拟考
- 洛谷 [P1012 拼数](https://www.luogu.com.cn/problem/P1012)：`string` 比较，不是整数比较
- 自己造一份「`n` + 带空格姓名」的 `in.txt`，确认 `ignore` 前后 `name[0]` 是不是空的——比交十次签到题值

下一篇：[前缀和与差分](../03-算法竞赛/前缀和与差分.md)。语言线见 [map 与 set](map与set.md)。

## 参考资料

- [多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md)
- [vector 与 sort](vector与sort.md)
- [cppreference：std::string](https://en.cppreference.com/w/cpp/string/basic_string)
- [cppreference：std::getline](https://en.cppreference.com/w/cpp/string/basic_string/getline)
- [OI Wiki：字符串基础](https://oi-wiki.org/string/basic/)
