# Sitemap 新增页面追踪工具：第一版需求分析与实施方案

本文基于 `docs/gf-sitemap-tracker.md` 的方法论，定义一个可以快速开发和本地运行的第一版 SEO 工具。

第一版不做完整的关键词机会分析平台，只验证最核心的业务闭环：

```text
添加网站
→ 自动发现或人工添加 sitemap
→ 首次扫描建立 URL 基线
→ 在控制面板手动检测
→ 找出新增 URL
→ 抓取 title / H1 / meta description
→ 在 Dashboard 查看结果
```

系统定位：

> 一个本地运行的轻量 Sitemap 新增页面监控工具。用户添加网站，系统发现 sitemap 并建立 URL 基线；用户手动发起检测后，系统识别新增 URL、抓取页面 SEO 元信息，并在 Dashboard 中展示结果和扫描历史。

## 1. 需求结论

这个需求边界明确，适合作为第一版。

它能够优先验证两个关键问题：

1. 监控成熟网站的 sitemap，是否能持续发现有价值的新页面？
2. 哪些网站和页面类型最值得后续接入关键词提取、Google Trends 和 SERP 分析？

第一版应保持轻量，不提前建设任务队列、自动调度、AI 分析和复杂评分系统。工具只负责可靠地发现和保存新增页面，机会判断暂时由人完成。

## 2. 第一版目标与边界

### 2.1 必须实现

- 添加和删除监控网站；
- 根据网站域名自动发现 sitemap；
- 允许人工添加和删除非标准 sitemap 地址；
- 支持普通 sitemap、sitemap index 和 `.xml.gz`；
- 在控制面板手动检测单个网站；
- 在控制面板手动检测全部网站；
- 首次扫描只建立 URL 基线，不把历史页面误报为新增；
- 后续扫描识别新增 URL；
- 抓取新增页面的 title、第一个 H1 和 meta description；
- 在 Dashboard 展示网站、扫描状态、新增页面和历史记录；
- 使用本地 SQLite 持久化数据；
- 程序关闭并重新启动后数据仍然存在。

### 2.2 第一版不实现

- 后台定时扫描；
- Google Trends 验证；
- 搜索量、关键词难度和 CPC；
- SERP 竞争分析；
- AI 关键词提取和机会评分；
- 邮件、Telegram、Discord 等外部告警；
- 用户注册、登录和多人权限；
- 云端部署和多实例运行；
- 自动注册域名、建站或发布内容；
- 页面正文抓取和内容相似度分析；
- 完整的 URL 删除和恢复追踪。

这些功能可以在第一版积累真实数据、证明方法有效后再逐步加入。

## 3. 技术方案

### 3.1 总体架构

参考 Twitter-Trend-Radar 的轻量结构：

```text
浏览器
└── index.html
    ├── 网站管理
    ├── 手动启动扫描
    ├── 扫描进度
    └── 新增 URL Dashboard
             │
             ▼ HTTP / JSON
Python 本地服务
├── 提供 index.html
├── 提供本地 API
├── 发现和解析 sitemap
├── 对比历史 URL
├── 抓取页面 SEO 元信息
└── 读写 SQLite
             │
             ▼
tracker.db
```

系统只监听本机地址：

```text
http://127.0.0.1:8787
```

第一版不默认监听 `0.0.0.0`，避免把允许访问任意网址的本地接口暴露到公网。

### 3.2 推荐技术栈

```text
后台语言       Python 3.11+
Web 服务       Flask
HTTP 请求      requests
HTML 解析      BeautifulSoup4
XML 解析       Python ElementTree
数据库         SQLite
前端           单个 index.html
前端样式       原生 CSS
前端交互       原生 JavaScript + fetch
```

不使用：

```text
React / Vue / Node.js / npm
Redis / Celery
PostgreSQL
Docker（第一版非必须）
```

Flask、requests 和 BeautifulSoup4 会增加少量依赖，但能显著简化 API、HTTP 错误处理和不规范 HTML 的解析。它仍然属于轻量方案。

依赖文件可以保持为：

```text
Flask
requests
beautifulsoup4
```

### 3.3 建议目录结构

```text
sitemap-tracker/
├── server.py              # 启动入口和 API
├── index.html             # 唯一的控制面板页面
├── requirements.txt
├── data/
│   └── tracker.db         # 运行后创建，不提交 Git
└── app/
    ├── __init__.py
    ├── db.py              # 建表、查询和事务
    ├── sitemap.py         # sitemap 发现和解析
    ├── scanner.py         # 基线、差异检测和扫描流程
    └── page_parser.py     # title/H1/meta 抓取
```

对使用者来说，仍然只有一个 Python 启动入口和一个 HTML 页面；后端内部适当拆分，避免所有逻辑堆积在 `server.py`。

## 4. 核心业务流程

### 4.1 添加网站

用户在控制面板输入：

```text
https://example.com
```

后端执行：

1. 校验 URL，只接受 `http` 或 `https`；
2. 规范化为站点根地址；
3. 检查该网站是否已经存在；
4. 保存网站记录；
5. 自动发现 sitemap；
6. 将发现结果返回控制面板；
7. 提示用户确认 sitemap 并建立首次基线。

第一版建议使用“两步式”交互：

```text
添加网站并发现 sitemap
→ 用户检查自动发现结果
→ 点击“建立基线”
```

这样用户可以在扫描大量 URL 之前补充非标准 sitemap，也能避免错误地址立刻触发全站抓取。

网站字段：

- 网站名称，可选；
- 网站根地址，必填；
- 创建时间；
- 是否已经建立基线；
- 最近扫描时间；
- 最近扫描状态；
- 已记录 URL 数量。

### 4.2 自动发现 Sitemap

发现顺序：

1. 请求网站的 `/robots.txt`；
2. 提取所有 `Sitemap:` 声明，不区分大小写；
3. 尝试 `/sitemap.xml`；
4. 尝试 `/sitemap_index.xml`；
5. 尝试 `/sitemap-index.xml`；
6. 对返回内容进行基本格式验证；
7. 去重后保存有效地址。

如果没有发现 sitemap，控制面板提示：

> 未自动发现 Sitemap，请人工输入 Sitemap 地址。

自动发现只在以下情况执行：

- 添加网站时；
- 用户点击“重新发现 Sitemap”时。

每次普通检测不需要重新执行全部自动发现逻辑。

### 4.3 人工管理 Sitemap

每个网站提供 sitemap 管理区域，支持：

- 查看自动发现的地址；
- 输入完整的非标准 sitemap 地址；
- 启用或停用某个 sitemap；
- 删除人工添加的 sitemap；
- 查看最近一次读取状态和错误；
- 点击“重新发现 Sitemap”。

需要标记来源：

```text
auto      自动发现
manual    人工添加
```

人工 sitemap 地址不要求位于网站根目录，但第一版默认要求它与目标网站属于同一主机或子域名。跨域 sitemap 如确有需求，后续可以增加人工确认开关。

### 4.4 首次扫描：建立基线

这是第一版最重要的业务规则。

网站第一次成功读取 sitemap 时，可能得到几千或几万个历史 URL。这些 URL 不能被标记为新增页面。

正确逻辑：

```text
if 网站尚未建立基线:
    保存本次读取到的全部 URL
    新增 URL 数量 = 0
    将网站标记为“基线已建立”
else:
    将当前 URL 与数据库中的历史 URL 对比
    只把从未出现过的 URL 标记为新增
```

基线完成后显示：

> 基线已建立，共记录 12,351 个 URL。下次检测开始识别新增页面。

如果首次扫描只成功读取了部分 sitemap，则不能建立基线。必须确保所有已启用 sitemap 均成功读取，或者由用户明确排除失败的 sitemap 后重新扫描。

### 4.5 手动检测新增 URL

控制面板提供：

- 单个网站的“检测新增 URL”；
- 顶部的“检测全部网站”；
- 扫描中状态；
- 简单进度信息；
- 扫描完成结果。

一次检测流程：

```text
创建扫描批次
→ 读取所有启用的 sitemap
→ 递归解析 sitemap index
→ 汇总并规范化 URL
→ 与历史 URL 对比
→ 立即保存新增 URL
→ 扫描批次标记为“URL 检测完成”
→ 抓取新增页面的 SEO 元信息
→ 更新成功数、失败数和最终状态
```

差异检测和页面抓取应分成两个阶段。即使某个新增页面无法抓取，新增 URL 也已经安全保存，不会因页面请求失败而丢失。

### 4.6 后台执行与进度

虽然扫描由用户手动启动，但不能让一个 HTTP 请求一直等待扫描完成。

建议流程：

1. 用户点击检测；
2. API 创建 `scan_runs` 记录；
3. Python 后台线程执行扫描；
4. API 立即返回 `scan_id`；
5. 前端每 1～2 秒查询扫描状态；
6. 完成后刷新统计和新增页面列表。

同一个网站同时只允许存在一个运行中的扫描任务。重复点击时返回当前扫描状态，不创建重复任务。

“检测全部网站”可以依次执行网站扫描，第一版不需要同时并发扫描所有网站。

## 5. Sitemap 解析要求

### 5.1 必须支持

- 标准 `<urlset>`；
- 标准 `<sitemapindex>`；
- sitemap index 中的子 sitemap；
- 两到三层递归 sitemap；
- `.xml.gz` 压缩响应；
- XML namespace；
- `<loc>` 和可选 `<lastmod>`；
- 同一个 URL 出现在多个 sitemap 时去重；
- HTTP 跳转；
- UTF-8 及响应头声明的常见编码。

### 5.2 安全限制

为防止错误或恶意 sitemap 导致程序失控，应设置：

- sitemap 请求超时：15～30 秒；
- 最大响应体：例如 50 MB；
- 最大递归深度：例如 3 层；
- 单站最大 sitemap 数：例如 500；
- 单次扫描最大 URL 数：可配置，默认 500,000；
- 拒绝非 HTTP/HTTPS 地址；
- 避免循环引用；
- 记录被截断或超限的原因。

任何超限扫描都不能覆盖网站的有效历史状态。

### 5.3 不依赖 lastmod 判断新增

`<lastmod>` 经常缺失、不准确，或者每次生成 sitemap 时全部刷新。

第一版判断新增的唯一依据应为：

> 规范化后的 URL 是否曾经成功保存到该网站的历史 URL 表中。

`lastmod` 只作为展示和辅助信息保存。

## 6. URL 规范化与去重

第一版至少执行：

- 协议和主机名转小写；
- 移除 `#fragment`；
- 移除默认端口；
- 移除 `utm_source`、`utm_medium`、`utm_campaign`、`fbclid` 等常见跟踪参数；
- 对保留下来的查询参数排序；
- 保留原始 URL；
- 为规范化 URL 生成 SHA-256 哈希；
- 使用 `site_id + url_hash` 唯一约束去重。

第一版不应擅自删除所有查询参数，因为部分站点使用参数区分真实内容页面。

尾斜杠是否等价也不能一刀切。默认保留原样，后续可以按网站增加规则。

## 7. 页面 SEO 元信息抓取

### 7.1 抓取范围

仅抓取本次发现的新增 URL，不重新抓取全部历史页面。

第一版保存：

- 原始 URL；
- HTTP 状态码；
- 最终跳转 URL；
- `<title>` 文本；
- 页面中第一个非空 `<h1>`；
- `<meta name="description" content="...">`；
- Content-Type；
- 抓取时间；
- 抓取状态；
- 失败原因。

这里的 `meta` 明确定义为 `meta description`，不是保存页面的全部 meta 标签。

### 7.2 抓取规则

- 单页面超时 10～15 秒；
- 使用明确的 User-Agent；
- 跟随有限次数的 HTTP 跳转；
- 只解析 HTML；
- 限制响应体大小，例如 5 MB；
- title、H1 和 description 去除多余空白；
- 字段过长时截断显示，但数据库可以保存合理长度的完整文本；
- 403、404、429、5xx、超时和解析失败都要记录；
- 429 时尊重 `Retry-After`，不持续高频重试；
- 第一版最多自动重试一次。

### 7.3 并发建议

页面抓取可以使用一个小型线程池：

```text
全局并发：5
单域名并发：3～5
```

由于同一次扫描通常只抓取一个网站的新增页面，默认并发 3 已经足够。并发数应支持配置，避免对目标站造成压力。

### 7.4 动态渲染页面

第一版只解析服务器返回的 HTML，不使用浏览器或 Playwright 执行 JavaScript。

如果某个页面的 title、H1 或 description 只在前端渲染，Dashboard 可以显示为空。这属于已知限制，不应在第一版引入浏览器自动化来解决。

## 8. Dashboard 需求

控制面板使用单个 `index.html`，原生 JavaScript 调用本地 JSON API，不需要前端构建步骤。

### 8.1 顶部统计卡片

显示：

- 网站总数；
- 已建立基线的网站数；
- 已记录 URL 总数；
- 最近一次检测的新增 URL 数；
- 页面抓取失败数。

### 8.2 添加网站区域

包含：

- 网站名称，可选；
- 网站地址，必填；
- “添加并发现 Sitemap”按钮；
- 自动发现结果；
- 人工 sitemap 输入框；
- “建立基线”按钮。

### 8.3 网站列表

每个网站显示：

- 名称和域名；
- sitemap 数量；
- 历史 URL 数量；
- 是否已经建立基线；
- 最近扫描时间；
- 最近新增数量；
- 当前扫描状态；
- 最近错误摘要。

操作按钮：

- 检测新增 URL；
- 管理 Sitemap；
- 重新发现 Sitemap；
- 删除网站。

删除网站属于高影响操作，必须二次确认。第一版可以级联删除该网站的 sitemap、URL、页面和扫描记录，但界面必须明确说明。

### 8.4 新增 URL 列表

建议字段：

| 字段 | 说明 |
|---|---|
| 发现时间 | URL 第一次被系统发现的时间 |
| 网站 | 所属网站 |
| URL | 可在新标签页打开 |
| Title | 页面 title |
| H1 | 第一个 H1 |
| Meta Description | 页面描述 |
| HTTP | 最后一次抓取状态码 |
| 抓取状态 | pending / success / failed / skipped |
| 来源 Sitemap | 首次发现该 URL 的 sitemap |
| 扫描批次 | 发现它的扫描记录 |

支持：

- 按网站筛选；
- 按发现日期筛选；
- 按抓取状态筛选；
- 搜索 URL、title、H1 和 description；
- 分页；
- 默认按发现时间倒序排列。

### 8.5 扫描历史

每个扫描批次显示：

- 网站；
- 开始和结束时间；
- 扫描状态；
- sitemap 成功数和失败数；
- 本次读取到的 URL 总数；
- 新增 URL 数；
- 页面抓取成功数和失败数；
- 错误摘要。

### 8.6 状态与错误提示

前端需要明确区分：

```text
idle             未运行
queued           等待运行
fetching_sitemap 正在读取 Sitemap
diffing          正在对比 URL
fetching_pages   正在抓取新增页面
completed        已完成
partial          部分成功
failed           失败
```

不能只显示“失败”。用户应能看到是 sitemap 请求、XML 解析、页面抓取还是数据库写入失败。

## 9. SQLite 数据模型

第一版建议使用五张核心表。

### 9.1 sites

```sql
CREATE TABLE sites (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    name            TEXT,
    base_url        TEXT NOT NULL UNIQUE,
    baseline_ready  INTEGER NOT NULL DEFAULT 0,
    status          TEXT NOT NULL DEFAULT 'idle',
    last_scanned_at TEXT,
    last_error      TEXT,
    created_at      TEXT NOT NULL,
    updated_at      TEXT NOT NULL
);
```

### 9.2 sitemaps

```sql
CREATE TABLE sitemaps (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    site_id         INTEGER NOT NULL,
    url             TEXT NOT NULL,
    source          TEXT NOT NULL,       -- auto / manual
    enabled         INTEGER NOT NULL DEFAULT 1,
    last_http_status INTEGER,
    last_checked_at TEXT,
    last_error      TEXT,
    created_at      TEXT NOT NULL,
    UNIQUE(site_id, url),
    FOREIGN KEY(site_id) REFERENCES sites(id) ON DELETE CASCADE
);
```

### 9.3 scan_runs

```sql
CREATE TABLE scan_runs (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    site_id         INTEGER NOT NULL,
    scan_type       TEXT NOT NULL,       -- baseline / detect
    status          TEXT NOT NULL,
    stage           TEXT,
    total_sitemaps  INTEGER NOT NULL DEFAULT 0,
    failed_sitemaps INTEGER NOT NULL DEFAULT 0,
    total_urls      INTEGER NOT NULL DEFAULT 0,
    new_urls        INTEGER NOT NULL DEFAULT 0,
    pages_succeeded INTEGER NOT NULL DEFAULT 0,
    pages_failed    INTEGER NOT NULL DEFAULT 0,
    error           TEXT,
    started_at      TEXT NOT NULL,
    finished_at     TEXT,
    FOREIGN KEY(site_id) REFERENCES sites(id) ON DELETE CASCADE
);
```

### 9.4 urls

```sql
CREATE TABLE urls (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    site_id         INTEGER NOT NULL,
    sitemap_id      INTEGER,
    first_scan_id   INTEGER NOT NULL,
    raw_url         TEXT NOT NULL,
    normalized_url  TEXT NOT NULL,
    url_hash        TEXT NOT NULL,
    sitemap_lastmod TEXT,
    first_seen_at   TEXT NOT NULL,
    last_seen_at    TEXT NOT NULL,
    is_new          INTEGER NOT NULL DEFAULT 0,
    UNIQUE(site_id, url_hash),
    FOREIGN KEY(site_id) REFERENCES sites(id) ON DELETE CASCADE,
    FOREIGN KEY(sitemap_id) REFERENCES sitemaps(id) ON DELETE SET NULL,
    FOREIGN KEY(first_scan_id) REFERENCES scan_runs(id) ON DELETE CASCADE
);
```

基线扫描插入的 URL：

```text
is_new = 0
```

后续扫描首次发现的 URL：

```text
is_new = 1
```

### 9.5 pages

```sql
CREATE TABLE pages (
    id               INTEGER PRIMARY KEY AUTOINCREMENT,
    url_id            INTEGER NOT NULL UNIQUE,
    http_status       INTEGER,
    final_url         TEXT,
    content_type      TEXT,
    title             TEXT,
    h1                TEXT,
    meta_description  TEXT,
    fetch_status      TEXT NOT NULL DEFAULT 'pending',
    fetch_error       TEXT,
    fetched_at        TEXT,
    FOREIGN KEY(url_id) REFERENCES urls(id) ON DELETE CASCADE
);
```

### 9.6 索引

```sql
CREATE INDEX idx_urls_first_seen
ON urls(first_seen_at);

CREATE INDEX idx_urls_site_new
ON urls(site_id, is_new, first_seen_at);

CREATE INDEX idx_scan_runs_site_started
ON scan_runs(site_id, started_at);

CREATE INDEX idx_pages_fetch_status
ON pages(fetch_status);
```

SQLite 连接需要启用：

```sql
PRAGMA foreign_keys = ON;
PRAGMA journal_mode = WAL;
```

WAL 模式可以减少 Dashboard 查询与后台扫描写入之间的阻塞。

## 10. 本地 API 设计

### 10.1 网站

```text
GET    /api/sites
POST   /api/sites
GET    /api/sites/{site_id}
DELETE /api/sites/{site_id}
```

添加网站：

```json
POST /api/sites
{
  "name": "Example",
  "base_url": "https://example.com"
}
```

### 10.2 Sitemap

```text
GET    /api/sites/{site_id}/sitemaps
POST   /api/sites/{site_id}/sitemaps
DELETE /api/sitemaps/{sitemap_id}
PATCH  /api/sitemaps/{sitemap_id}
POST   /api/sites/{site_id}/discover-sitemaps
```

### 10.3 扫描

```text
POST /api/sites/{site_id}/baseline
POST /api/sites/{site_id}/scan
POST /api/scans/all
GET  /api/scans
GET  /api/scans/{scan_id}
```

启动扫描后立即返回：

```json
{
  "scan_id": 42,
  "status": "queued"
}
```

### 10.4 Dashboard

```text
GET /api/stats
GET /api/pages
GET /api/pages?site_id=1&status=failed&q=game&page=1
```

所有 API 错误使用统一格式：

```json
{
  "error": {
    "code": "SITEMAP_FETCH_FAILED",
    "message": "无法读取 Sitemap",
    "details": "HTTP 403"
  }
}
```

## 11. 数据一致性与异常保护

### 11.1 不完整扫描不能更新有效状态

如果本次扫描出现以下情况：

- 某个已启用 sitemap 请求失败；
- XML 解析失败；
- 超过系统限制而被截断；
- 子 sitemap 只读取到一部分；
- 数据库写入中断；

则扫描状态应为 `partial` 或 `failed`，不能将其当作一次完整快照。

第一版只检测“从未见过的 URL”，因此部分扫描不会制造删除误报；但首次基线必须要求完整成功。

### 11.2 数据库事务

同一轮 URL 差异检测应使用数据库事务：

```text
开始事务
→ 批量插入或更新 URL
→ 创建待抓取页面记录
→ 更新扫描统计
→ 提交事务
```

如果中途失败则整体回滚，避免只保存部分新增 URL。

页面元信息抓取可以逐条提交，因为单个页面失败不应回滚其他页面。

### 11.3 幂等

- `sites.base_url` 唯一；
- `sitemaps(site_id, url)` 唯一；
- `urls(site_id, url_hash)` 唯一；
- `pages.url_id` 唯一；
- 重复扫描不会重复创建 URL 或页面记录；
- 重复点击扫描不会同时运行两个相同站点任务。

## 12. 安全与抓取约束

### 12.1 本地绑定

默认只监听：

```text
127.0.0.1
```

第一版没有用户登录和权限系统，因此不应直接部署到公网。

### 12.2 URL 安全

由于用户可以输入网址，后端必须：

- 只接受 HTTP/HTTPS；
- 拒绝包含用户名和密码的 URL；
- 拒绝 `file://`、`ftp://` 等协议；
- 拒绝访问 localhost、环回地址和私有网络地址；
- DNS 解析后再次检查实际目标 IP；
- HTTP 跳转后重新进行地址安全检查；
- 限制响应大小、请求时间和跳转次数。

即使是本地工具，也应防止意外访问路由器、云元数据服务或本机管理接口。

### 12.3 抓取礼仪

- 使用可识别的 User-Agent；
- 控制并发和请求频率；
- 尊重 robots.txt 和目标网站规则；
- 遇到 429 主动退避；
- 不绕过登录、验证码、付费墙和访问限制；
- 不执行目标页面中的 JavaScript；
- 不把网页内容当作程序指令执行。

## 13. 启动和使用方式

安装：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

启动：

```bash
python3 server.py
```

终端输出：

```text
Sitemap Tracker is running at http://127.0.0.1:8787
Database: ./data/tracker.db
```

用户在浏览器打开地址即可使用，不需要安装前端依赖。

## 14. 开发阶段

### 阶段一：数据层和网站管理

- 建立目录结构；
- 初始化 SQLite；
- 完成网站 CRUD；
- 完成 sitemap 人工管理；
- 完成基础 Dashboard 框架。

验收：添加网站后重启程序，数据仍然存在。

### 阶段二：Sitemap 发现和解析

- robots.txt 发现；
- 常见 sitemap 路径发现；
- urlset 和 sitemap index 解析；
- gzip 和递归支持；
- URL 规范化和去重；
- 错误与限制保护。

验收：使用多种真实 sitemap 样本得到正确 URL 集合。

### 阶段三：基线和新增检测

- 首次基线流程；
- 后续差异检测；
- 扫描批次和状态；
- 后台线程和前端轮询；
- 单站扫描锁；
- 检测全部网站。

验收：第一次扫描新增数为零；修改测试 sitemap 后，第二次只发现新加入的 URL。

### 阶段四：页面信息抓取

- HTTP 状态和跳转；
- title、H1 和 meta description；
- 超时、限流和错误记录；
- 小型并发线程池；
- Dashboard 新增页面列表。

验收：页面可抓取、不可抓取和非 HTML 三类情况都得到正确状态。

### 阶段五：测试和体验修复

- 补充单元测试和集成测试；
- 验证大型 sitemap；
- 验证重复扫描；
- 验证程序中断后的数据一致性；
- 优化加载、空状态、错误提示和移动端基本显示。

## 15. 测试方案

### 15.1 单元测试

- robots.txt 中 Sitemap 声明提取；
- urlset 解析；
- sitemap index 递归解析；
- gzip 解压；
- XML namespace；
- URL 规范化和哈希；
- 新旧 URL 集合差异；
- title、H1 和 meta description 提取；
- 私有地址和非法协议拦截。

### 15.2 集成测试

- 添加网站并自动发现 sitemap；
- 自动发现失败后人工添加 sitemap；
- 第一次扫描只建立基线；
- 第二次扫描发现新增 URL；
- 同一 URL 位于多个 sitemap 时只保存一次；
- 子 sitemap 失败时基线不成立；
- 页面抓取失败不丢失新增 URL；
- 重复点击不会启动重复扫描；
- 重启服务后 Dashboard 数据完整；
- 删除网站后相关数据正确清理。

### 15.3 测试样本

测试目录中应保存本地 fixture：

```text
robots-with-sitemaps.txt
urlset.xml
sitemap-index.xml
nested-sitemap-index.xml
urlset.xml.gz
invalid.xml
sample-page.html
sample-page-no-h1.html
```

测试不应完全依赖公网网站，否则结果会受到网络和目标站变化影响。

## 16. 验收标准

第一版完成必须满足：

1. 能添加、查看和删除网站；
2. 能从 robots.txt 和常见路径自动发现 sitemap；
3. 能人工添加、停用和删除非标准 sitemap；
4. 能解析 urlset、sitemap index 和 `.xml.gz`；
5. 第一次扫描正确建立基线，不误报历史 URL；
6. 后续扫描只把从未出现过的 URL 标记为新增；
7. 能抓取并展示 title、H1 和 meta description；
8. Sitemap 或页面抓取失败时有明确状态和错误；
9. 失败扫描不会破坏有效历史数据；
10. 重复扫描不会重复创建 URL；
11. Dashboard 支持网站、日期、状态筛选和文本搜索；
12. SQLite 数据在程序重启后保持完整；
13. 服务默认只监听 `127.0.0.1`；
14. 非法协议和私有网络目标受到拦截；
15. 核心单元测试和集成测试通过。

## 17. 复杂度和工期评估

| 模块 | 复杂度 | 说明 |
|---|---|---|
| SQLite 数据层 | 低 | 数据规模和关系较简单 |
| 网站管理 | 低 | 常规 CRUD |
| Sitemap 自动发现 | 中 | robots、常见路径和异常处理 |
| Sitemap 递归与 gzip | 中 | 需要防循环和超限 |
| URL 差异检测 | 低 | 依赖唯一索引即可可靠实现 |
| 页面元信息抓取 | 中 | 超时、跳转、编码和异常 HTML |
| 后台扫描状态 | 中 | 需要线程安全和单站任务锁 |
| 单 HTML Dashboard | 中 | 包含表格、筛选、分页和轮询 |
| 测试与边界修复 | 中 | 真实网站情况差异较多 |

参考工期：

- 可运行原型：2～3 个开发日；
- 功能完整 MVP：5～8 个开发日；
- 加入真实网站测试和异常修复：约 1～2 周。

## 18. 第一版之后的迭代方向

第一版运行 2～4 周后，先分析真实数据，再决定增加哪些功能。

推荐顺序：

```text
V1.1  定时扫描和简单桌面通知
V1.2  CSV 导出、URL 标记和人工备注
V1.3  从 title/H1 自动提取候选关键词
V1.4  Google Trends 人工触发验证
V1.5  SERP、搜索量和机会评分
```

是否增加复杂功能，应由以下数据决定：

- 哪类网站最常产生有效新增页面；
- 每次检测的平均新增数量；
- 新增 URL 中有价值页面的比例；
- title/H1 是否足以支持人工判断；
- 人工最终采纳了哪些页面和关键词；
- 自动扫描和外部告警是否真的必要。

## 19. 最终建议

第一版应优先保证三件事：

1. **基线准确**：第一次扫描绝不能把全站历史页面当作新增；
2. **数据可靠**：失败、重复执行或程序重启不能破坏历史数据；
3. **操作简单**：启动一个 Python 服务，打开一个 HTML 控制面板即可完成全部操作。

只要“添加网站 → 建立基线 → 手动检测 → 查看新增页面”这条链路稳定可用，第一版就已经达成目标。关键词提取、趋势验证和机会评分应等真实监控数据证明价值之后再建设，避免在尚未验证核心方法之前把工具做重。
