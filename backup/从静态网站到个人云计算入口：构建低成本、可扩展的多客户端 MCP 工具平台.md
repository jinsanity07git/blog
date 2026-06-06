# 从静态网站到个人云计算入口：构建低成本、可扩展的多客户端 MCP 工具平台

## 引言：真正的现代化，是减少必须由自己维护的东西

个人网站最初往往部署在一台 VPS 上：

```text
VPS
├── Linux
├── Nginx / Apache
├── PHP
├── WordPress
├── MySQL
└── 插件与主题
```

这种架构直观、灵活，也曾经非常实用。

但即使网站只是展示几篇文章，每一次访问仍然可能触发：

```text
HTTP 请求
 ↓
PHP 执行
 ↓
数据库查询
 ↓
模板渲染
 ↓
返回 HTML
```

网站所有者不仅需要写内容，还需要承担服务器升级、数据库备份、依赖兼容、安全补丁、证书更新、故障排查和性能优化。

当网站迁移到：

```text
Hugo
+
GitHub Pages
+
Cloudflare
```

变化并不仅仅是“网页加载更快”。

更深层的变化是：

> 把不必要的实时计算提前完成，把稳定内容变成静态文件，把全球分发交给 CDN，把动态能力收敛为少量明确的服务接口。

在此基础上，再加入一台公网 VPS：

```text
Cloudflare
+
GitHub Pages
+
VPS FastAPI
+
Streamable HTTP MCP Server
```

就可以形成一套兼顾静态内容、动态应用、AI 工具调用和未来重型计算扩展能力的个人云计算架构。

这套架构的目标不是堆叠最多的技术，而是：

> 用最低的长期运营成本，构建一个足够简单、可以逐步扩展、不会因为增长而被迫推倒重来的系统。

------

# 一、从三层结构理解云计算架构

任何云计算方案都可以分成三个层次。

| 层级       | 需要解决的问题                             | 典型组成                                              |
| ---------- | ------------------------------------------ | ----------------------------------------------------- |
| 基础设施层 | 算力、网络和存储从哪里来？                 | CDN、VPS、磁盘、对象存储、Workstation                 |
| 平台架构层 | 服务如何部署？请求如何流转？状态存在哪里？ | Hugo、FastAPI、MCP、Gradio、PostgreSQL、Redis、Docker |
| 经济运营层 | 系统是否值得长期维护？扩展是否可控？       | 托管服务、备份、日志、监控、自动部署、维护时间        |

很多架构设计只关注第二层：

```text
使用什么语言？
使用什么框架？
要不要上 Kubernetes？
```

但真正决定长期成本的，通常是第一层和第三层：

```text
哪些计算必须实时执行？
哪些文件必须保存在 VPS？
哪些流量必须经过自己的服务器？
哪些服务必须由自己维护？
```

一套好的架构，不是让每台机器都承担所有任务，而是让每一种资源承担它最擅长的工作。

------

# 二、云计算成本的本质：算力、存储和网络

云计算的基础资源可以简化为三个部分：

```text
Compute   算力
Storage   存储
Network   网络
```

传统 VPS 架构的问题并不是 VPS 本身昂贵，而是它很容易成为一个承担所有责任的单点：

```text
静态文件
动态网页
数据库
日志
用户上传
任务执行
文件下载
定时任务
AI 工具
```

全部集中在一台服务器上。

短期看很方便，长期却会导致：

```text
资源竞争
维护复杂
备份困难
故障影响范围扩大
升级风险上升
扩展边界模糊
```

更合理的方案是对三类资源分别优化。

------

# 三、目标架构：静态优先，动态收敛，算力分层

最终推荐的整体结构如下：

```text
┌──────────────────────────────────────────────┐
│ 第一层：静态内容层                           │
│                                              │
│ Hugo + GitHub Pages                          │
│                                              │
│ 博客、文档、个人主页、静态 Dashboard          │
│ HTML、CSS、JavaScript、Markdown、图片          │
└──────────────────────┬───────────────────────┘
                       │
                       │ HTTPS
                       ▼
┌──────────────────────────────────────────────┐
│ 第二层：网络边缘层                           │
│                                              │
│ Cloudflare                                   │
│                                              │
│ DNS、TLS、CDN、缓存、访问控制、基础防护        │
│ 可选：Workers / Pages Functions              │
└──────────────────────┬───────────────────────┘
                       │
                       │ HTTPS
                       ▼
┌──────────────────────────────────────────────┐
│ 第三层：公网动态服务层                       │
│                                              │
│ VPS                                          │
│                                              │
│ Caddy / Nginx                                │
│ ├── /api/*   → FastAPI                      │
│ ├── /mcp     → Streamable HTTP MCP Server   │
│ ├── /ui/*    → 可选 Gradio                  │
│ └── /health  → 健康检查                     │
│                                              │
│ PostgreSQL：业务状态                         │
│ Redis：可选缓存与任务队列                    │
│ Docker Compose：服务管理                     │
└──────────────────────┬───────────────────────┘
                       │
                       │ 未来按需增加
                       ▼
┌──────────────────────────────────────────────┐
│ 第四层：内网重型计算层                       │
│                                              │
│ Workstation                                  │
│                                              │
│ CPU、GPU、RAM、NVMe                          │
│ Python Worker、GIS、模型、本地 LLM            │
└──────────────────────────────────────────────┘
```

第一阶段不必一次性部署所有组件。

最重要的是建立清晰边界：

```text
静态内容
→ 交给静态托管平台

网络分发
→ 交给 CDN

轻量动态能力
→ 交给边缘函数或 VPS

多客户端工具调用
→ 交给 MCP Server

长时间后台任务
→ 交给 Worker

重型计算
→ 未来交给内网 Workstation
```

------

# 四、算力优势：减少必须实时执行的计算

## 1. 将运行时计算转移到构建时

传统 WordPress 页面通常在用户访问时动态生成：

```text
用户访问
 ↓
运行 PHP
 ↓
查询 MySQL
 ↓
渲染模板
 ↓
返回页面
```

Hugo 的思路完全不同：

```text
Markdown
 ↓
Hugo Build
 ↓
预先生成 HTML、CSS 和 JavaScript
 ↓
用户直接获取静态文件
```

计算发生时间发生了变化：

```text
Run Time Compute
        ↓
Build Time Compute
```

这意味着：

```text
访问量增加
≠
VPS CPU 负载同比增加
```

只要内容没有变化，就没有必要为每一位访客重新生成同一份网页。

这是降低运营成本最有效的方式之一：

> 最便宜的计算不是更低价的计算，而是不需要重复执行的计算。

------

## 2. 将轻量计算留在边缘

部分动态功能不值得占用 VPS：

```text
短链接跳转
联系表单校验
简单访问控制
请求重定向
少量区域化内容
轻量 API 代理
```

这些任务适合交给 Cloudflare Workers 或 Pages Functions。

边缘函数的价值不是替代 VPS，而是：

```text
让短小、无状态、低延迟的逻辑
在靠近用户的位置执行
```

这样可以减少：

```text
VPS 请求数量
跨区域网络往返
后端负载
故障影响范围
```

------

## 3. 将 VPS 保留给真正需要服务器的任务

VPS 适合承担：

```text
身份验证
权限管理
工具调用
任务提交
数据库查询
状态管理
日志索引
轻量脚本
API 聚合
```

VPS 不必承担：

```text
全球静态文件分发
每次访问都重复生成网页
大型文件长期存储
重型 GPU 推理
超长时间模型计算
```

这使 VPS 从一台“包办所有工作的服务器”，转变成：

> 一个低负载、稳定、职责明确的公网协调节点。

------

## 4. 重型计算可以延后接入 Workstation

当任务逐步增加，例如：

```text
交通模型
GIS 空间分析
大型 DataFrame
矩阵运算
批量数据处理
本地 LLM
Ollama
GPU 推理
```

不要急着购买更昂贵的 VPS。

可以保留现有公网入口，在内网增加 Workstation：

```text
手机或桌面 Client
 ↓
VPS
 ↓
任务队列
 ↓
内网 Workstation
 ↓
CPU / GPU / RAM / NVMe
```

VPS 负责稳定性，Workstation 负责算力。

两者职责分离：

```text
VPS
=
门卫、前台、调度中心

Workstation
=
机房、工厂、算力节点
```

------

## 5. 算力成本总结

| 架构方式            | 算力消耗方式                      | 长期成本特征             |
| ------------------- | --------------------------------- | ------------------------ |
| WordPress 动态页面  | 每次访问重复执行 PHP 和数据库查询 | 流量增长会增加服务器压力 |
| Hugo 静态页面       | 发布时构建一次，访问时直接分发    | 访问成本显著降低         |
| Cloudflare 边缘函数 | 轻量逻辑在边缘按需执行            | 减少 VPS 请求和延迟      |
| VPS FastAPI         | 承担必要的业务逻辑                | 保持服务清晰可控         |
| Workstation Worker  | 承担重型任务                      | 按需扩展，不污染公网服务 |

长期原则可以概括为：

```text
可以预计算的，不实时计算；
可以缓存的，不重复计算；
可以异步的，不阻塞请求；
可以分层的，不集中在一台机器。
```

------

# 五、存储优势：让不同数据进入不同仓库

## 1. 传统 VPS 容易变成数据杂物间

一个长期运行的 VPS 很容易逐渐堆积：

```text
数据库
网站源代码
上传文件
日志
缓存
备份
图片
模型结果
临时文件
用户下载
```

这会导致：

```text
磁盘空间难以管理
备份越来越慢
恢复过程复杂
权限边界混乱
服务器迁移困难
```

更合理的方案不是不断扩大磁盘，而是按数据性质分层。

------

## 2. 推荐的存储分层

| 数据类型               | 推荐存储位置                     | 原因                     |
| ---------------------- | -------------------------------- | ------------------------ |
| 博客 Markdown          | Git 仓库                         | 可版本控制、可回滚       |
| Hugo 生成的静态文件    | GitHub Pages 或 Cloudflare Pages | 不占用 VPS 存储          |
| 图片、附件、报告       | 对象存储                         | 便于扩展和下载           |
| 用户、权限、任务元数据 | PostgreSQL                       | 适合结构化查询           |
| 临时缓存               | Redis                            | 可以快速读写，也可以重建 |
| 大型输入文件           | 对象存储或内网 NAS               | 避免 VPS 成为中转瓶颈    |
| GPU 模型和工作数据     | Workstation 本地 NVMe            | 靠近计算节点，减少传输   |
| 审计日志               | 数据库或专用日志目录             | 便于追踪和归档           |

这一原则可以压缩为：

```text
Git
→ 保存内容和配置

数据库
→ 保存业务状态

对象存储
→ 保存大文件

缓存
→ 保存可重建的临时数据

Workstation 本地磁盘
→ 保存靠近算力的数据
```

------

## 3. 大文件不要反复经过 VPS

假设用户提交一个大型任务：

```text
上传 5 GB 数据
 ↓
VPS
 ↓
Workstation 下载
 ↓
生成 3 GB 报告
 ↓
上传回 VPS
 ↓
手机下载
```

此时 VPS 同时承担：

```text
公网入口
中转站
存储节点
带宽瓶颈
```

更合理的方式是：

```text
用户
 ↓
对象存储
 ↓
Workstation

Workstation
 ↓
对象存储
 ↓
用户
```

VPS 只保存：

```text
任务 ID
文件地址
权限
状态
结果索引
```

VPS 不必搬运所有数据。

------

## 4. 存储成本总结

传统思路是：

```text
VPS 磁盘不够
→ 扩容磁盘
```

分层思路是：

```text
先判断数据性质
→ 放到最适合的位置
```

这样做的直接收益包括：

```text
VPS 更轻量
备份更简单
恢复更快
磁盘压力更低
扩展不必停机
数据迁移更容易
```

长期原则是：

> 数据不要因为方便而全部堆在同一台机器上。
> 存储位置应该由数据性质决定，而不是由最初部署位置决定。

------

# 六、网络优势：让数据走最短、最合理的路径

## 1. 静态内容由 CDN 分发

单一 VPS 的网络路径是：

```text
全球用户
 ↓
固定位置的 VPS
```

静态 CDN 的网络路径是：

```text
全球用户
 ↓
附近的边缘节点
 ↓
缓存内容
```

可以把它理解成物流系统：

```text
单一 VPS
=
所有商品都从中央仓库发货

CDN
=
提前把常用商品配送到区域仓库
```

对于：

```text
HTML
CSS
JavaScript
图片
字体
普通附件
```

CDN 比单一 VPS 更合理。

这不仅改善加载速度，也降低 VPS 的带宽消耗。

------

## 2. 动态请求只发送必要数据

静态网页仍然可以成为动态应用入口。

例如：

```text
GitHub Pages 上的 Dashboard
 ↓
JavaScript
 ↓
api.example.com
 ↓
VPS FastAPI
```

页面外壳是静态的，只有真正变化的数据通过 API 请求获取：

```text
任务状态
系统信息
用户权限
日志摘要
报告索引
```

因此：

```text
不变化的内容
→ CDN 缓存

变化的数据
→ API 动态获取
```

这比每次请求都重新下载完整页面更高效。

------

## 3. 重型计算结果使用对象存储直传

网络设计应该避免不必要的中转：

```text
错误方式：
浏览器 → VPS → Workstation → VPS → 浏览器

更合理：
浏览器 → 对象存储
Workstation → 对象存储
VPS 只管理元数据
```

这会明显减少：

```text
VPS 带宽占用
磁盘 I/O
内存压力
传输时间
失败重试成本
```

------

## 4. 内网 Workstation 不直接暴露公网

未来接入 Workstation 时，不建议直接开放：

```text
Workstation:8000
```

因为 Workstation 往往保存：

```text
源代码
模型
数据
SSH 密钥
内部目录
GPU 服务
开发环境
```

更安全的路径是：

```text
公网用户
 ↓
Cloudflare
 ↓
VPS
 ↓
私有网络
 ↓
Workstation
```

VPS 与 Workstation 可以通过：

```text
Tailscale
WireGuard
Cloudflare Tunnel
```

建立私有通信。

如果使用 Cloudflare Tunnel，内网机器可以主动建立向外连接，而不需要开放入站端口。

------

## 5. 网络成本总结

| 流量类型       | 推荐路径                     |
| -------------- | ---------------------------- |
| 博客和静态资源 | CDN → 用户                   |
| 普通 API 请求  | 用户 → VPS                   |
| MCP 工具调用   | Client → VPS `/mcp`          |
| 任务进度       | VPS → Client，优先 SSE       |
| 大文件上传     | 用户 → 对象存储              |
| 大文件下载     | 对象存储 → 用户              |
| 内网重型任务   | VPS → 私有网络 → Workstation |

长期原则是：

> 网络优化不是单纯提高带宽，而是减少不必要的数据移动。

------

# 七、为什么选择 Streamable HTTP MCP Server？

## 1. 从单机工具转向共享工具平台

MCP 的价值是：

> 将工具能力抽象成标准化接口，让不同 Client 可以发现并调用同一组工具。

本地 STDIO 模式通常是：

```text
Claude Code
 ↓
本地子进程
 ↓
本地 MCP Server
```

适合：

```text
单机开发
本地调试
文件系统工具
不希望开放网络的脚本
```

远程 Streamable HTTP 模式则是：

```text
手机 ────────┐
MacBook ─────┼──→ VPS MCP Server
Windows PC ──┤
自动化脚本 ──┤
AI Client ───┘
```

更适合：

```text
多设备
多客户端
集中部署
统一升级
统一认证
统一日志
统一权限
```

------

## 2. MCP Server 是能力出口，不是聊天界面

VPS 可以提供一个统一入口：

```text
https://api.example.com/mcp
```

暴露工具：

```text
get_system_status()
list_reports()
read_report(filename)
submit_job(job_type, parameters)
get_job_status(job_id)
cancel_job(job_id)
generate_report(report_type)
```

也可以暴露资源：

```text
report://latest
jobs://recent
docs://architecture
```

MCP 解决的是：

```text
不同 AI Client
如何用统一方式调用工具
```

它不必替代：

```text
FastAPI
Gradio
网页 Dashboard
后台 Worker
数据库
```

------

## 3. Streamable HTTP 的基本机制

远程 MCP Server 提供统一端点：

```text
POST /mcp
GET  /mcp
```

客户端通过 `POST` 发送 JSON-RPC 请求：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {}
}
```

调用工具：

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "get_system_status",
    "arguments": {}
  }
}
```

对于快速任务：

```text
Client
 ↓ POST /mcp
Server
 ↑ JSON Response
```

对于持续推送：

```text
Client
 ↓ GET /mcp
Server
 ↑ SSE Stream
```

------

# 八、SSE、WebSocket 和异步任务如何分工？

## 1. SSE：优先用于单向进度推送

SSE 适合：

```text
任务进度
日志流
状态更新
通知
流式文本
```

例如：

```text
queued
 ↓
running: 10%
 ↓
running: 55%
 ↓
uploading_result: 90%
 ↓
completed
```

对于手机端尤其合适，因为实现简单，断线后也更容易恢复。

------

## 2. WebSocket：只在真正需要双向实时交互时使用

WebSocket 适合：

```text
交互式终端
远程控制台
在线协作
高频双向通信
持续修改运行参数
```

不要默认引入 WebSocket。

多数个人工具平台只需要：

```text
HTTP API
+
Streamable HTTP MCP
+
SSE
```

------

## 3. 长任务必须从 HTTP 请求中拆出来

不要这样设计：

```text
POST /run-model
 ↓
等待 45 分钟
 ↓
返回结果
```

更合理的方式是：

```text
POST /api/jobs
 ↓
返回 job_id
 ↓
后台执行
 ↓
客户端轮询或订阅 SSE
```

例如：

```json
{
  "job_id": "model-run-20260606-001",
  "status": "queued"
}
```

任务状态写入数据库：

```json
{
  "job_id": "model-run-20260606-001",
  "type": "generate_report",
  "status": "running",
  "progress": 42,
  "message": "正在处理输入文件",
  "result_url": null
}
```

------

# 九、FastAPI、Gradio、MCP 和 Worker 的职责边界

这些组件不是互相替代，而是承担不同角色。

| 组件                   | 核心职责                       |
| ---------------------- | ------------------------------ |
| Hugo                   | 将 Markdown 构建为静态页面     |
| GitHub Pages           | 托管静态站点                   |
| Cloudflare             | CDN、TLS、网络边缘、访问保护   |
| FastAPI                | 普通业务 API、认证、任务管理   |
| Gradio                 | 快速构建手机友好的交互界面     |
| MCP Server             | 向多个 AI Client 暴露标准工具  |
| PostgreSQL             | 保存用户、权限、任务和审计信息 |
| Redis                  | 缓存、任务队列、短期事件       |
| RQ / Celery / Dramatiq | 执行可恢复的异步任务           |
| Docker Compose         | 固化和管理服务环境             |
| Workstation            | 承担 GPU、GIS 和重型计算       |
| Kubernetes             | 未来多节点调度，不是当前必需品 |

整体关系如下：

```text
┌──────────────────────────────┐
│ GitHub Pages                 │
│ Hugo 静态网站                │
└──────────────┬───────────────┘
               │
               │ HTTPS
               ▼
┌──────────────────────────────┐
│ Cloudflare                   │
│ CDN / DNS / TLS              │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ VPS                          │
│                              │
│ Caddy / Nginx                │
│ ├── /ui/*    → Gradio       │
│ ├── /api/*   → FastAPI      │
│ ├── /mcp     → MCP Server   │
│ └── /health  → FastAPI      │
│                              │
│ PostgreSQL                   │
│ Redis                        │
│ Worker                       │
└──────────────┬───────────────┘
               │
               │ 未来按需扩展
               ▼
┌──────────────────────────────┐
│ 内网 Workstation            │
│                              │
│ CPU / GPU / RAM / NVMe      │
│ Worker / Ollama / GIS       │
└──────────────────────────────┘
```

------

# 十、Gradio 在架构中的合理位置

Gradio 很适合快速构建一个手机端工具面板。

例如：

```text
https://tools.example.com/
```

手机浏览器打开后，可以显示：

```text
检查 VPS 状态
提交任务
查看任务列表
查看运行进度
读取日志
下载报告
调用本地 LLM
```

Gradio 可以处理：

```text
请求排队
并发限制
进度条
日志流
文件上传
结果下载
简单聊天页面
```

但 Gradio 不应该成为整个后台系统。

它更适合承担：

```text
交互层
```

而不是：

```text
持久化任务系统
权限中心
数据库
重型调度器
```

合理分工是：

```text
Gradio
=
手机端操作台

FastAPI
=
业务接口

MCP Server
=
AI 工具标准出口

Redis + Worker
=
异步任务执行

PostgreSQL
=
可恢复业务状态
```

------

# 十一、Docker Compose：当前阶段最重要的运维工具

Docker 的价值不是炫技，而是减少环境维护成本。

它解决：

```text
Python 版本不一致
依赖散落
升级难回滚
服务启动方式不同
服务器重装恢复困难
```

对于一台 VPS，推荐使用 Docker Compose 管理：

```text
Caddy
FastAPI
MCP Server
Gradio
PostgreSQL
Redis
Worker
```

目录结构可以保持简单：

```text
personal-cloud/
├── compose.yaml
├── caddy/
│   └── Caddyfile
├── api/
│   ├── Dockerfile
│   └── main.py
├── mcp-server/
│   ├── Dockerfile
│   └── app.py
├── gradio-ui/
│   ├── Dockerfile
│   └── app.py
└── data/
    ├── reports/
    └── results/
```

示例 `compose.yaml`：

```yaml
services:
  api:
    build: ./api
    restart: unless-stopped
    expose:
      - "8001"

  mcp-server:
    build: ./mcp-server
    restart: unless-stopped
    expose:
      - "8000"
    volumes:
      - ./data/reports:/data/reports:ro

  gradio-ui:
    build: ./gradio-ui
    restart: unless-stopped
    expose:
      - "7860"

  postgres:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_DB: personal_cloud
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:8
    restart: unless-stopped

  caddy:
    image: caddy:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./caddy/Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - api
      - mcp-server
      - gradio-ui

volumes:
  postgres_data:
  caddy_data:
  caddy_config:
```

示例 `Caddyfile`：

```text
api.example.com {
    handle /mcp* {
        reverse_proxy mcp-server:8000
    }

    handle /api/* {
        reverse_proxy api:8001
    }

    handle /health {
        reverse_proxy api:8001
    }
}

tools.example.com {
    reverse_proxy gradio-ui:7860
}
```

启动：

```bash
docker compose up -d --build
```

查看日志：

```bash
docker compose logs -f
```

更新：

```bash
git pull
docker compose up -d --build
```

------

# 十二、Kubernetes 什么时候才值得引入？

Docker 和 Kubernetes 解决的问题不同。

可以类比为：

```text
Docker
=
标准化集装箱

Docker Compose
=
管理一艘船上的集装箱

Kubernetes
=
管理港口、船队、仓库和调度系统
```

当前架构如果只有：

```text
1 台 VPS
+
少量 Client
+
少量 Worker
```

Docker Compose 足够。

只有当系统逐步发展为：

```text
多个 VPS
多个 Workstation
多个 GPU 节点
多个 Worker
多个用户
服务高可用
复杂资源调度
```

才需要考虑：

```text
K3s
或
Kubernetes
```

Kubernetes 能解决：

```text
容器运行在哪台机器？
服务需要几份副本？
容器挂了如何自动恢复？
流量如何负载均衡？
GPU 如何调度？
滚动升级如何进行？
```

但它不会自动解决：

```text
业务状态设计
权限边界
任务语义
文件备份
网络安全
运营成本
```

长期原则是：

> 先解决可复现部署，再解决多节点调度。
> 不要为未来可能出现的问题，提前承担今天必然发生的维护成本。

------

# 十三、安全边界：公网工具必须收窄权限

远程 MCP Server 不应该直接暴露万能工具。

不要轻易提供：

```text
run_any_shell_command(command)
execute_any_sql(sql)
read_any_file(path)
delete_any_file(path)
```

这些接口几乎等价于开放远程 Shell。

更安全的方式是提供边界明确的工具：

```text
get_system_status()
list_reports()
read_approved_report(filename)
submit_registered_job(job_type, parameters)
get_job_status(job_id)
cancel_job(job_id)
restart_named_service(service_name)
```

可以按风险分层：

| 风险级别 | 示例                   | 执行策略           |
| -------- | ---------------------- | ------------------ |
| 低风险   | 查询状态、读取公开报告 | 自动执行           |
| 中风险   | 提交任务、生成报告     | 需要确认           |
| 高风险   | 修改配置、重启服务     | 二次确认并记录日志 |
| 极高风险 | 任意 Shell、任意 SQL   | 默认不暴露         |

公网入口至少需要：

```text
HTTPS
身份认证
权限控制
输入校验
Origin 校验
限流
审计日志
超时
任务取消
```

个人使用时可以先从：

```text
Cloudflare Access
+
HTTPS
+
固定 Token
```

开始。

多个用户或多个正式 Client 接入后，再升级到：

```text
OAuth
+
Scope
+
审计日志
```

------

# 十四、业务状态必须外置

系统中的关键状态不要只存在于：

```text
内存
HTTP 连接
SSE 连接
MCP Session
Gradio 进程
Worker 进程
```

正确划分如下：

| 状态           | 推荐位置              |
| -------------- | --------------------- |
| 用户信息       | PostgreSQL            |
| 权限           | PostgreSQL            |
| 任务状态       | PostgreSQL            |
| 任务队列       | Redis                 |
| 临时缓存       | Redis                 |
| 大型结果       | 对象存储              |
| 静态内容       | Git                   |
| 实时连接上下文 | 内存                  |
| 审计日志       | PostgreSQL 或日志系统 |

长期原则是：

> 连接可以断开，服务可以重启，容器可以替换，任务不能丢失。

------

# 十五、渐进式实施路线

不要一次性部署最终形态。

按照真实需求逐步演化。

## 阶段一：静态网站

```text
Hugo
+
GitHub Pages
+
Cloudflare
```

实现：

```text
博客
文档
个人主页
静态 Dashboard
```

运营特点：

```text
几乎没有服务器维护
静态流量不占 VPS
发布流程简单
```

------

## 阶段二：加入轻量 VPS

```text
Caddy
+
FastAPI
+
Docker Compose
```

实现：

```text
健康检查
报告查询
简单表单
任务列表
基础 API
```

运营特点：

```text
只维护一台轻量 VPS
动态能力开始集中管理
```

------

## 阶段三：部署远程 MCP Server

```text
Streamable HTTP
+
/mcp
```

优先实现只读工具：

```text
get_server_time
get_system_status
list_reports
read_report
```

运营特点：

```text
一套工具服务多个 Client
环境只维护一份
工具升级只需部署一次
```

------

## 阶段四：加入手机端 Gradio UI

```text
tools.example.com
 ↓
Gradio
```

实现：

```text
任务提交
状态查看
日志流
文件上传
结果下载
简单聊天
```

运营特点：

```text
快速获得手机端体验
不必开发原生 App
```

------

## 阶段五：加入异步任务系统

```text
FastAPI
+
PostgreSQL
+
Redis
+
RQ / Celery / Dramatiq
```

实现：

```text
submit_job
get_job_status
cancel_job
download_result
```

运营特点：

```text
任务可以跨重启恢复
前端和执行逻辑解耦
```

------

## 阶段六：接入 Workstation

```text
VPS
 ↓
私有网络
 ↓
Workstation Worker
```

实现：

```text
大型 Python 任务
GIS
交通模型
本地 LLM
GPU 推理
```

运营特点：

```text
无需购买昂贵 GPU VPS
公网节点继续保持轻量
Workstation 不直接暴露公网
```

------

## 阶段七：多节点调度

当真实需求增长到：

```text
多台 VPS
多台 Workstation
多 GPU
多个用户
高可用
```

再考虑：

```text
K3s
或
Kubernetes
```

------

# 十六、从运营成本角度总结整套架构

## 1. 算力成本

通过静态化、缓存、边缘执行和后台 Worker：

```text
减少重复计算
降低 VPS CPU 压力
避免过早升级服务器
重型任务按需迁移到 Workstation
```

核心思想：

> 算力不是越多越好，而是不要把昂贵算力浪费在不必要的实时计算上。

------

## 2. 存储成本

通过 Git、数据库、缓存、对象存储和本地 NVMe 分层：

```text
VPS 磁盘保持轻量
大文件不挤占系统盘
备份范围更清楚
恢复过程更简单
数据迁移更容易
```

核心思想：

> 存储不是把所有数据放在同一个地方，而是让不同数据进入不同仓库。

------

## 3. 网络成本

通过 CDN、静态分发、API 收敛、对象存储直传和私有网络：

```text
静态流量不经过 VPS
大文件不经过 VPS 中转
动态请求只发送必要数据
Workstation 不暴露公网
移动端访问路径更清晰
```

核心思想：

> 网络优化不是单纯追求更大带宽，而是减少不必要的数据移动。

------

## 4. 运维成本

通过托管平台、职责分离和 Docker Compose：

```text
减少需要手工维护的服务
降低环境漂移
升级和回滚更简单
故障范围更容易定位
未来扩展不需要推倒重来
```

核心思想：

> 架构现代化的价值，不是技术栈更新，而是把人的时间从重复维护中释放出来。

------

# 十七、十条长期指导原则

## 原则一：静态优先

```text
能在构建时完成，就不要在访问时重复完成。
```

## 原则二：边缘增强，不要边缘滥用

```text
边缘节点适合短小逻辑，不适合重型计算。
```

## 原则三：VPS 是公网协调节点，不是无限算力池

```text
VPS 优先承担认证、调度、状态管理和工具出口。
```

## 原则四：减少数据移动

```text
大文件走对象存储，重型数据靠近计算节点。
```

## 原则五：长任务必须异步化

```text
提交任务后返回 job_id，不要阻塞 HTTP 请求。
```

## 原则六：业务状态必须外置

```text
连接可以断，任务不能丢。
```

## 原则七：工具接口必须收窄

```text
不要默认暴露任意 Shell、任意 SQL 和任意文件访问。
```

## 原则八：MCP 与 FastAPI 各司其职

```text
MCP 负责标准化工具出口，FastAPI 负责普通业务接口。
```

## 原则九：Docker 优先，Kubernetes 延后

```text
先解决可复现部署，再解决多节点调度。
```

## 原则十：复杂度必须随真实需求增长

```text
不要为想象中的规模支付今天的维护成本。
```

------

# 结语：从维护服务器，到管理能力

最初，个人网站可能只是：

```text
一台 VPS
+
一个 WordPress
```

经过逐步演化，它可以成为：

```text
Hugo
+
GitHub Pages
+
Cloudflare
+
VPS FastAPI
+
Streamable HTTP MCP Server
+
Gradio 手机端界面
+
PostgreSQL
+
Redis Worker
+
未来可选的内网 Workstation
```

表面上看，组件数量增加了。

但如果边界划分清楚，维护成本反而下降：

```text
静态内容
→ 交给静态平台

网络分发
→ 交给 CDN

动态业务
→ 交给 FastAPI

AI 工具
→ 交给 MCP Server

手机端交互
→ 交给 Gradio

任务状态
→ 交给数据库

长任务
→ 交给 Worker

重型计算
→ 交给 Workstation

环境固化
→ 交给 Docker Compose

多节点调度
→ 留给真正需要时的 Kubernetes
```

这套架构的最终目标不是构建一个复杂的平台。

而是建立一种长期有效的资源配置方式：

> 将稳定内容静态化，将动态能力服务化，将工具接口标准化，将业务状态持久化，将重型计算隔离化，将网络流量合理分层，将运维复杂度控制在真实需求之内。

当这些边界建立以后，VPS 就不再只是一台需要不断维护的服务器。

它会逐渐成为：

> 一个可以从手机、电脑、自动化脚本和 AI Client 随时访问的个人云计算入口。