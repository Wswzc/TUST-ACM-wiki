---
title: 从 OJ 到 Web 的最小后端：一个端口、一份 JSON
author: TUST-ACM 算法实验室
date: 2026-09-19
category: 02-编程方向
tags: [后端, HTTP, Python, JSON, 工程]
level: 入门
status: 已完成
---

# 从 OJ 到 Web 的最小后端：一个端口、一份 JSON

> 读完你应该能：在本机用 Python 标准库拉起一个只回答 JSON 的 HTTP 服务；用浏览器或 `curl` 打通 GET / POST；对照 [后端实习词汇表](后端实习词汇表.md) 指出路径、状态码、请求体分别在哪；知道这和 Spring / 数据库无关，只是把「标准输入输出」换成「一次请求一次响应」。
>
> 一句口号：**OJ 从 stdin 读、往 stdout 写；最小后端从套接字读、往套接字写，身体里仍是解析和分支。**

## 背景 / 为什么要读这篇

[后端实习词汇表](后端实习词汇表.md) 和 [前端实习词汇表](前端实习词汇表.md) 把词钉过了。词会了，手还停在 `cin`。实习仓库里的 Controller 看起来像魔法，其实第一层就是：进来一个 HTTP 请求，出去一个状态码加一段 JSON。

TUST-ACM 算法实验室不需要第一周上生产。需要的是：你能指着屏幕说「这就是接口」。本篇用 **Python 3 标准库**，不 `pip install`，Windows 能跑。框架（Flask / Spring）是以后的衣服；衣服下面是同一具身体。

## 正文

### 1. 和 A+B 对照

| OJ | 最小后端 |
| --- | --- |
| 评测机调你的进程 | 浏览器或 `curl` 调你的端口 |
| 一组输入 | 一次请求（方法 + 路径 + 头 + 体） |
| 一组输出 | 一次响应（状态码 + 头 + 体） |
| `WA` | 4xx / 5xx 或 JSON 字段不对 |
| 多组 EOF | 服务一直循环等下一单 |

[多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md) 里「进程不退出、继续读」的直觉，在这里变成「进程不退出、继续 accept」。差别是协议多了几行 ASCII：`GET /api/add HTTP/1.1` 这类。

### 2. 能跑的最小服务

把下面存成 `server.py`（路径不要中文空格更好）。需要本机已有 Python 3：

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
from urllib.parse import urlparse, parse_qs


class Handler(BaseHTTPRequestHandler):
    def _json(self, code, obj):
        body = json.dumps(obj, ensure_ascii=False).encode("utf-8")
        self.send_response(code)
        self.send_header("Content-Type", "application/json; charset=utf-8")
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)

    def do_GET(self):
        u = urlparse(self.path)
        if u.path == "/api/hello":
            self._json(200, {"msg": "hello from TUST-ACM"})
            return
        if u.path == "/api/add":
            q = parse_qs(u.query)
            try:
                a = int(q.get("a", ["0"])[0])
                b = int(q.get("b", ["0"])[0])
            except ValueError:
                self._json(400, {"error": "a and b must be int"})
                return
            self._json(200, {"sum": a + b})
            return
        self._json(404, {"error": "not found"})

    def do_POST(self):
        u = urlparse(self.path)
        if u.path != "/api/echo":
            self._json(404, {"error": "not found"})
            return
        n = int(self.headers.get("Content-Length", "0"))
        raw = self.rfile.read(n)
        try:
            data = json.loads(raw.decode("utf-8") or "{}")
        except json.JSONDecodeError:
            self._json(400, {"error": "invalid json"})
            return
        self._json(200, {"you_sent": data})


if __name__ == "__main__":
    HTTPServer(("127.0.0.1", 8000), Handler).serve_forever()
```

启动：

```bash
python server.py
```

另开一个终端（PowerShell 亦可）：

```bash
curl http://127.0.0.1:8000/api/hello
curl "http://127.0.0.1:8000/api/add?a=3&b=5"
curl -X POST http://127.0.0.1:8000/api/echo -H "Content-Type: application/json" -d "{\"x\":1}"
```

浏览器打开 `http://127.0.0.1:8000/api/hello` 也能看见 JSON。再按 F12 → Network，对照词汇表里的方法、状态码、Body。

`127.0.0.1` 是本机。不要把这个玩具绑到 `0.0.0.0` 然后在宿舍网裸奔。没有鉴权、没有 HTTPS、没有数据库。

### 3. 代码在干什么（对应词汇）

| 片段 | 词汇 |
| --- | --- |
| `do_GET` / `do_POST` | HTTP 方法 |
| `u.path == "/api/add"` | 路由 / 接口路径 |
| `parse_qs` / `json.loads` | 查询参数、请求体 |
| `send_response(400)` | 状态码：你这请求有问题 |
| `Content-Type: application/json` | 头 |
| `serve_forever` | 进程常驻，不像 OJ 跑完就退 |

业务就是 `a+b`。竞赛选手熟悉的部分：解析、分支、错误时别崩溃。不熟悉的部分：必须先写对协议，对方才理你。漏 `Content-Length`、JSON 没 `encode`，浏览器会显示一坨乱象，那是协议 WA，不是算法 WA。

### 4. 离实习仓库还差什么（只点名）

真仓库会在这具身体外套上：

- 框架：少写手搓 Header  
- 路由表、中间件（日志、鉴权）  
- 数据库：见词汇表的 SQL，本篇故意没有  
- 配置与环境：开发 / 测试端口不同  
- 前端：[前端词汇表](前端实习词汇表.md) 里的页面来打这些接口；本机还会碰到 CORS——玩具若只给 `curl` 用，可以暂时不理

下一步若想「像实习一点」：同一套 `/api/add` 用 Flask 再写一遍，对比少了哪些样板代码。不要第一步就上微服务。

## 常见坑 / 注意事项

- **端口被占用。** 换 8001，或关掉上一个 `python server.py`。  
- **路径写成 `/api/add/` 多一个斜杠。** 对不上就 404。  
- **POST 不读 `Content-Length`，`rfile.read()` 卡死。**  
- **把玩具部署到公网。**  
- **简历写「精通后端」只交了这篇的 `a+b`。** 见 [简历上的竞赛经历](../04-行业认知/简历上的竞赛经历.md)：写「完成过一个本地 JSON API 并演示 GET/POST」可以，精通不行。  
- **Windows 上敲的是 `python` 还是 `py`。** 以 `py -3 --version` 能出号为准。

## 趣味课间：实验室经典翻车集

- **「后端不就是写算法题输出 JSON 吗。」** 差协议、差状态码、差长期挂着。算法是 handler 里面那几行。  
- **用 C++ 写了套接字，卡在 HTTP 头解析。** 入门用脚本语言把路径跑通，C++ 留给评测机。  
- **curl 过了，前端同学一接 CORS。** 不是前端玄学，是浏览器安全策略。先让 `curl` 当标准输入。  
- **服务写在 `while(cin)` 里等评测机。** 没有评测机来喂你。是你自己 `serve_forever`。

## 小结

最小后端 = 常驻进程 + 按路径分支 + JSON 进出。先用标准库打通 GET/POST，再谈框架和数据库。OJ 练的是 handler；Web 多出来的是协议和生命周期。

建议接着：

- 给 `/api/add` 加一种错误：缺参数返回 400，用 curl 看 JSON  
- 在 F12 里把同一条请求和 [后端词汇表](后端实习词汇表.md) 对一遍  
- 简历条目写可演示的事实，见 [简历上的竞赛经历](../04-行业认知/简历上的竞赛经历.md)

## 参考资料

- [后端实习词汇表](后端实习词汇表.md)
- [前端实习词汇表](前端实习词汇表.md)
- [多组输入与 EOF](../03-算法竞赛/多组输入与EOF.md)
- Python `http.server` 文档（搜 Python BaseHTTPRequestHandler）
