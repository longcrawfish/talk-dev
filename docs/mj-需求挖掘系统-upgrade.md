# MJ 需求挖掘系统升级方案

本文是对 `docs/mj-需求挖掘系统.md` 的讨论总结。原文可以看作一篇“方法论 + 系统设计”文章：它不是单纯讲关键词工具，而是在描述一套自动化机会发现流程。

核心结论：

> 这套方法论大方向靠谱，适合用来批量发现和初筛机会；但它不能直接替代人的商业判断。要真正完善，需要从“关键词机会筛选系统”升级成“商业机会研究系统”。

## 1. 对原方法论的判断

原文提出的主链路是：

```text
种子词
→ 新词发现
→ 老词筛选
→ 候选词合并
→ HN / GitHub / Tavily 社区扫描
→ 社区词回流到 Trends 验证
→ SERP 竞争分析
→ 最终分级 shortlist
```

这套方法论的基本思路是成立的。

它的价值在于把“找方向”这件事从随机刷网页，变成一个可重复的研究流程。系统负责收集、清洗、验证和初筛，人负责判断商业价值和最终取舍。

比较靠谱的设计点：

- 同时使用搜索需求和社区信号，避免只看搜索量或只追社区热点。
- 社区词不直接进入 shortlist，而是回流到 Trends 或关键词数据中重新验证。
- SERP 竞争分析作为最后关口，判断小站是否有机会进入前排。
- 用统一判断链处理不同来源的候选词，减少拍脑袋决策。

但它也有明显边界：

- Google Trends 只能反映相对趋势，不等于真实搜索量。
- KD、SERP 大站比例等指标都有经验主义成分，不能机械套用。
- HN、GitHub、Reddit 更适合技术类和互联网类产品，对其他行业未必适用。
- 搜索机会不等于商业机会，有流量不代表能赚钱。

更准确地说，这套系统适合作为“需求雷达”，不适合作为“自动决策器”。

## 2. 原方案最需要改进的地方

原方案已经能做关键词发现和 SEO 机会筛选，但如果要真正辅助独立开发者或小团队选方向，需要补上四层能力：

```text
商业价值
产品可行性
痛点证据
回测校准
```

### 2.1 增加商业价值评分

原方案主要回答：

```text
有没有人搜？
趋势是不是在涨？
SERP 能不能切进去？
```

但还需要回答：

```text
用户愿不愿意付费？
有没有订阅或复购空间？
竞品是否已经证明有人买单？
这个词对应的是免费流量，还是商业需求？
```

建议新增字段：

```text
buyer_intent        购买意图
monetization_path   变现路径
arpu_potential      客单价潜力
repeat_usage        复用频率
urgency             痛点紧急程度
cpc                 点击价格
paid_competition    广告竞争程度
ads_count           广告数量
```

### 2.2 SERP 分析不只看大站比例

原方案用“大站 / niche 站”判断 SERP 是否可切入，这是有价值的，但不够。

更重要的是分析前排页面类型：

```text
工具页
文章页
目录页
论坛讨论
产品官网
模板页
视频
图片结果
购物结果
```

如果前排大多是文章，而用户真实需求是完成一个任务，那么做交互式工具可能有机会。反过来，如果前排全是成熟免费工具，即使 niche 站很多，也未必值得进入。

### 2.3 增加产品可做性和风险评分

有些关键词 SEO 指标很好，但产品上很难做，或者风险很高。

建议新增：

```text
build_complexity    开发复杂度
data_dependency     是否依赖专有数据
model_dependency    是否依赖特定模型能力
legal_risk          法律风险
copyright_risk      版权风险
platform_risk       平台依赖风险
quality_bar         用户对结果质量的要求
time_to_mvp         MVP 时间
defensibility       是否容易被复制
```

这一步可以过滤掉很多“看起来有流量，但不适合做产品”的方向。

### 2.4 社区信号要抽取痛点，而不只是统计热度

HN、GitHub、Reddit 的价值不只是提及次数，而是用户表达问题的方式。

应该重点识别这些句式：

```text
I wish there was...
Is there a tool for...
How do I...
Any alternative to...
This is too expensive
I hate that...
Looking for...
```

社区层应该输出结构化痛点，例如：

```json
{
  "pain_points": [
    "existing tools are too expensive",
    "users need batch processing",
    "results are not accurate enough"
  ],
  "urgency": "medium",
  "willingness_to_pay_signal": "low"
}
```

### 2.5 增加反验证

发现“低竞争 + 高搜索”的词时，不应该马上判断为机会，而要先问：

```text
为什么这个词没人做大？
是不是用户不付费？
是不是一次性需求？
是不是结果质量很难做好？
是不是版权或合规风险高？
是不是流量国家购买力低？
是不是 Google SERP 已经直接满足需求？
是不是会被平台功能吃掉？
```

很多机会看起来诱人，本质上是陷阱。反验证能减少误判。

### 2.6 建立回测和反馈机制

如果没有回测，评分系统很容易只是“看起来科学”。

建议记录每个机会后续表现：

```text
是否做了 landing page
是否做了工具页
收录速度
排名变化
点击率
注册率
付费率
留存率
人工判断是否正确
```

系统应该根据真实结果反向校准权重。

## 3. 复杂度判断

真正实现这套系统并不算特别复杂，但也不是几个脚本就能长期稳定运行。

可以分三档理解。

### 3.1 MVP 版本：不复杂

目标：每天跑一次，输出 20-50 个候选机会。

包含：

```text
种子词
关键词扩展
Google Trends 验证
DataForSEO 搜索量 / KD / CPC
SERP Top 10 分析
简单社区搜索
最终评分表
```

技术上可以是一组 Python 脚本、一个定时任务、一份 Markdown / CSV 报告。熟练开发者大约 3-7 天可以做出第一版。

### 3.2 可长期运行版本：中等复杂

目标：每天自动跑，结果稳定，可追踪历史变化。

需要增加：

```text
任务编排
失败重试
缓存
历史数据存储
关键词归一化
品牌词 / 噪音词库
SERP 域名分类
趋势变化追踪
日报 / 周报
人工标注反馈
```

这时建议引入清晰的数据模型，而不是只依赖零散 JSON 文件。一个人认真做，大约 2-4 周可以做出可靠内部工具。

### 3.3 完善版本：复杂

目标：不仅发现关键词，还判断商业机会。

需要做：

```text
搜索需求评分
SERP 可进入性评分
社区痛点抽取
竞品定价分析
CPC / 广告竞争分析
产品可做性评分
法律 / 版权 / 平台风险评分
人工反馈校准
回测系统
多市场 / 多语言支持
可视化 dashboard
```

这已经不是 keyword research pipeline，而是 opportunity intelligence system。一个人可以做，但要做扎实，通常需要 1-3 个月持续迭代。

真正难的不是代码，而是：

- 数据源不稳定。
- 关键词归一化难。
- 评分权重容易失真。
- 商业判断很难完全自动化。

## 4. 推荐系统设计

如果重新设计，我会把它设计成一个“机会研究流水线”，而不是关键词工具。

系统目标：

```text
输入：领域、种子词、目标市场
输出：结构化机会卡片
用途：帮助人快速判断一个方向是否值得继续验证
```

核心原则：

> 机器负责收集、归一化、初筛、打分；人负责最终判断和反馈校准。

### 4.1 系统分层

```text
1. Seed Layer          种子词和领域配置
2. Discovery Layer     候选机会发现
3. Evidence Layer      多源证据采集
4. Scoring Layer       机会评分
5. Review Layer        人工复核和反馈
6. Reporting Layer     报告、通知、历史追踪
```

### 4.2 输入不只是关键词，而是研究项目

示例：

```json
{
  "project": "AI tools",
  "markets": ["US", "UK", "CA"],
  "language": "en",
  "seed_keywords": [
    "ai generator",
    "ai detector",
    "ai editor",
    "ai image tool"
  ],
  "exclude_brands": [
    "chatgpt",
    "midjourney",
    "deepseek",
    "canva"
  ],
  "business_model": ["subscription", "tool", "affiliate"],
  "risk_tolerance": "medium"
}
```

同一个关键词，对 SaaS、工具站、联盟站、内容站的价值不同。系统需要知道研究上下文。

### 4.3 候选发现层

来源可以分三类：

```text
搜索侧：
- Google Trends
- DataForSEO / Ahrefs / Semrush 类关键词库
- Google Suggest
- People Also Ask
- SERP related searches

社区侧：
- Reddit
- Hacker News
- GitHub
- Product Hunt
- X / YouTube / TikTok，可选

竞品侧：
- 竞品排名关键词
- 竞品功能页
- 竞品定价页
- alternative / vs / best 类查询
```

候选词不是最终对象。更好的对象是 Opportunity Candidate：

```json
{
  "canonical_query": "ai tattoo generator",
  "variants": [
    "tattoo ai generator",
    "free ai tattoo generator",
    "ai tattoo design generator"
  ],
  "source": ["google_trends", "github", "reddit"],
  "first_seen_at": "2026-03-01"
}
```

关键点是归一化。`ai tattoo generator` 和 `tattoo ai generator` 大概率是同一个机会，不应该分开打分。

### 4.4 证据采集层

每个机会采集五类证据。

需求证据：

```text
search_volume
trend_12m
trend_3m
trend_slope
growth_rate
seasonality
query_variants_count
```

竞争证据：

```text
kd
serp_top10_domains
big_site_ratio
niche_site_ratio
weak_pages_count
content_type_distribution
domain_authority_estimate
```

商业证据：

```text
cpc
paid_ads_count
advertiser_domains
competitor_pricing
has_subscription
has_free_plan
affiliate_presence
b2b_or_b2c
buyer_intent
```

痛点证据：

```text
pain_points
complaints
desired_features
current_workarounds
alternative_requests
price_complaints
quality_complaints
```

可做性和风险证据：

```text
mvp_complexity
data_dependency
model_dependency
legal_risk
copyright_risk
platform_risk
quality_bar
time_to_mvp
defensibility
```

### 4.5 评分模型

第一版不需要机器学习，建议使用透明规则评分，方便调试和校准。

```text
Opportunity Score =
  Demand Score
+ Access Score
+ Pain Score
+ Monetization Score
+ Buildability Score
- Risk Score
```

每项可以按 0-20 分计算，总分 100。系统不应该只输出分数，还必须输出理由。

示例：

```text
Demand Score        18/20  搜索量不错，趋势上升
Access Score        14/20  SERP 有 niche 站，但工具页竞争存在
Pain Score          12/20  社区有明确吐槽，但不算强烈
Monetization Score   8/20  CPC 低，付费意愿一般
Buildability Score  16/20  MVP 容易做
Risk Score          -6     版权和生成质量有风险

Total: 62/100
Decision: Observe / Test Landing Page
```

### 4.6 决策输出

最终不要只分“值得继续 / 可观察 / 不值得”。更好的方式是给出下一步动作：

```text
A. Build MVP        证据强，SERP 可进，商业价值明确
B. Build Landing    需求可能存在，但商业证据不足
C. Content Test     适合先做 SEO 页面验证
D. Watchlist        有早期信号，继续观察
E. Reject           竞争、风险或商业价值不成立
```

### 4.7 机会卡片格式

最终报告应该从关键词列表升级为机会卡片：

```text
Opportunity: AI Tattoo Generator

Decision:
Content Test

Why:
- Search demand is growing across 4 related variants.
- SERP has several niche tools, so entry is possible.
- Community pain exists around style quality and stencil-ready output.
- Monetization signal is weak: low CPC, many free tools.
- MVP is easy, but differentiation needs style-specific workflows.

Key Metrics:
- Search volume: 8,100/month
- KD: 15
- CPC: $0.42
- Trend: +28% over 3 months
- SERP: 7 niche / 3 big
- Ads: 1 active advertiser

Suggested MVP:
- Tattoo style generator
- Placement preview
- Stencil-friendly black line output
- Prompt presets by style

Risks:
- Low willingness to pay
- Generic AI image output is easy to copy
- Copyright / style quality issues

Next Action:
Create one SEO landing page and test conversion before building full product.
```

## 5. 数据模型建议

第一版可以用 SQLite，后续迁移到 Postgres。

核心表：

```text
projects
seed_keywords
keyword_candidates
keyword_variants
daily_keyword_metrics
serp_snapshots
serp_results
community_mentions
competitors
competitor_pricing
opportunity_scores
opportunity_reviews
experiments
```

最重要的是保留历史数据。不要每天覆盖结果，否则无法判断一个词是在持续变好，还是只是一次性波动。

## 6. 任务编排建议

第一版不需要 Airflow。可以用：

```text
cron / GitHub Actions / Cloudflare Workers Cron
        ↓
pipeline runner
        ↓
step jobs
        ↓
database
        ↓
report generator
        ↓
Telegram / Slack / Email
```

每个步骤都应该可重跑、可缓存、可失败恢复。

任务拆分：

```text
discover_candidates
normalize_candidates
fetch_keyword_metrics
fetch_trends
fetch_serp
classify_serp
scan_community
extract_pain_points
analyze_competitors
score_opportunities
generate_report
```

每个 job 记录：

```text
status
started_at
finished_at
input_hash
error_message
api_cost
```

## 7. 人工反馈机制

这是系统能不能变准的关键。

每个机会卡片后面应该允许人工标记：

```text
Approve
Watch
Reject
```

同时选择原因：

```text
商业价值弱
太难做
SERP 太强
风险太高
不符合当前能力
值得测试
```

这些反馈进入下一轮评分校准。

例如，如果人工连续拒绝“低 CPC + 娱乐类 + 一次性使用”的机会，系统以后应该自动降低这类方向的权重。

## 8. 推荐实现路线

不要一开始做完整系统。建议按四个版本推进。

### V0：脚本版，3-5 天

```text
输入 seed_keywords.json
拉 DataForSEO 关键词数据
拉 SERP Top 10
做简单 big / niche 分类
输出 CSV + Markdown
```

目标：先跑通闭环。

### V1：可用版，1-2 周

```text
加入 Trends
加入 CPC
加入社区搜索
加入机会评分
加入历史存储
日报推送
```

目标：每天能产出可读报告。

### V2：研究助手版，3-4 周

```text
加入 LLM 痛点抽取
加入竞品定价分析
加入 SERP 内容类型分析
加入人工 review
加入机会卡片
```

目标：从关键词表升级成机会卡。

### V3：决策系统版，1-3 个月

```text
加入回测
加入实验记录
加入多市场
加入 dashboard
加入评分权重校准
```

目标：知道系统过去判断得准不准。

## 9. 第一版推荐技术栈

```text
Python
SQLite / Postgres
DataForSEO API
pytrends
Tavily / SerpAPI / Brave Search API
GitHub Search API
OpenAI-compatible LLM API
cron / GitHub Actions / simple worker
Markdown + CSV 报告
Telegram / Slack / Email 推送
```

第一版不建议先做复杂前端。真正有价值的是稳定的机会卡片和可追踪的历史记录。

## 10. 最终判断

原文的方法论是一个好的起点。它已经具备“需求雷达”的雏形：

```text
搜索发现
社区补词
趋势验证
SERP 判断
最终分级
```

但如果要把它做成真正可用的系统，需要升级成：

```text
以关键词为入口
以多源证据为基础
以机会卡片为输出
以人工反馈和回测持续校准
```

最重要的设计取舍是：

> 不要让系统自动决定做什么产品。让系统持续产出高质量候选机会，并解释为什么值得测试、为什么应该放弃。

这样的系统不是替代判断，而是提高判断效率。

