# song gift 规划综合建议（summary-1）

> 本文是在 `song-gift-plan.md`（原方案）+ 三份 AI 建议（`song-gift-plan-suggest-1/2/3.md`）+ `poc-checklist.md` + 两份成本调研（`service-cost-survey.md`、`service-cost-survey-2.md`）基础上的综合判断、分歧裁决、补充建议与分阶段规划。引用均标注源文件与行号，方便核对。

---

## 一、总体判断

原方案方向是**对的**（免费引流 → AI 商业化 → `/gift/[id]` 裂变飞轮），三份 AI 建议各有侧重且互补：

| 维度 | 状态 |
|---|---|
| 产品定位 | 需从 "AI Song Generator" 重定位为 **Personalized Song Gift**（`song-gift-plan-suggest-1.md:668-696`） |
| 商业模型 | 毛利空间巨大（单首交付成本 ~$0.05，售价 $4.99–19.99，`service-cost-survey.md:340-348`），**但前提是重生成被控住** |
| 技术栈 | Cloudflare 全家桶 + Next.js，**成本几乎可忽略**（R2/Queue 都是免费额度内，`service-cost-survey-2.md:213-272, 697-784`），唯一风险是 Next.js on Workers 兼容性 |
| 最大盲点 | **音乐质量方差**（`song-gift-plan-suggest-3.md:18-26`）——三份建议都没把它当产品定义级风险 |
| MVP 复杂度 | 偏大，需要再砍 |

**一句话结论**：商业模式站得住、基础设施成本不是问题，**唯一的不确定性集中在「AI 音乐能否稳定生成拿得出手的歌」**。所有动作都应围绕这个不确定性展开。

---

## 二、关键分歧裁决

### 分歧 1：工具入口（单入口 vs 三工具）

- AI-1：收敛成 mixed-gift-maker 单入口
- AI-2：保留三工具
- **裁决：采用 AI-3 的折中（`song-gift-plan-suggest-3.md:61`）** —— 用户侧单一入口（Builder 心智），但保留三个独立 SEO 落地页 URL 抓不同搜索意图，底层是同一组件。
- 理由：单入口降低用户认知负担，独立 URL 服务 SEO，两者不矛盾。原方案「free/AI/mixed」三个并列工具名对用户偏开发者视角。

### 分歧 2：歌词先出 vs 音乐先出

- AI-1：合并，先音乐后看词（减少步骤、惊喜感）
- AI-2：保留歌词前置
- **裁决：歌词前置（同 AI-3，`song-gift-plan-suggest-3.md:62`）**，但 UI 上只做轻量预览，不让用户在词上反复纠结。
- 理由：歌词生成成本几乎为 0（`service-cost-survey.md:110-121`），但音乐生成每首 $0.03+。**先让用户校验"名字/故事"是否进词，是免费的纠错步骤，能显著降低音乐层的无效重生成成本**。这与成本控制的核心逻辑一致。

### 分歧 3：积分扣费时机（生成时 vs 解锁时）

- AI-2：生成音乐时**预扣**（成本前置，`song-gift-plan-suggest-2.md:13, 203`）
- 成本调研：每次 generation 都扣（`service-cost-survey.md:350-356, 1379-1433`）
- **裁决：按"每次生成即扣"**，与成本调研一致。
- 理由：成本在 generation 那一刻就发生了，不是在 unlock。原方案"两版预览→解锁才扣"会让站点承担全部试听成本（`song-gift-plan-suggest-2.md:9-16` 的亏损风险）。具体：
  - 用户看到的价格是 "Create Song = N credits"（含一次生成）
  - 生成 v1 扣 N credits
  - "Create another version" 再扣 N credits（不要叫 regenerate，`service-cost-survey.md:1281-1302`）
  - 失败/无音频自动退还（这条保护用户）

### 分歧 4：MVP 范围

- AI-1：大幅削减（砍 PDF/上传/下载/草稿/后台）
- AI-2：基本保留
- **裁决：基本采用 AI-1，但保留两点**（同 AI-3，`song-gift-plan-suggest-3.md:63`）：
  - ✅ **保留收礼页下载**（礼物价值感关键，砍掉削弱分享动机）
  - ✅ **保留最小草稿机制**（异步生成必需，否则用户离场即丢现场）
  - ❌ 砍掉：PDF 歌词、图片上传、后台统计页、复杂多场景

---

## 三、几个被低估/遗漏的风险

文档已经覆盖得很全，本节**只补几个仍被低估的**。

### 1. 质量方差 > 平均质量（最关键）

`song-gift-plan-suggest-3.md:18-26` 和 `poc-checklist.md:57-75` 说得对。礼物是**一次性高情感投入**，用户不会接受"这次明显比上次差"。POC 必须做**同输入 N=10 次盲评方差测试**，验收线 σ ≤ 1.0、"可送礼度 ≥3 占比 ≥60%"、**零灾难性输出**。

### 2. pixabay 音乐授权（免费入口的隐形炸弹）

`song-gift-plan-suggest-3.md:36-39` 点出了但被低估。原方案"采集可商用音乐到 R2"（`song-gift-plan.md:64-65`），但 pixabay 授权条款对**二次分发**并不一定允许，且条款会变。建议：
- **逐首核实并留存授权快照**（截图 + URL + 日期）
- 优先选 Pixabay Content Creator License 明确允许再分发的
- 备份方案：用 AI 生成一批纯音乐 BGM 存 R2，规避版权

### 3. 礼物链接生命周期 & 存储成本累积

三份文档都没说 `/gift/[id]` 永久还是定期失效（`song-gift-plan-suggest-3.md:40-44`）。建议：
- **链接永久有效**（礼物有多年后回看的情感诉求，失效会反噬品牌）
- 已交付歌曲走 R2 标准 storage（成本可忽略，`service-cost-survey-2.md:213-272`）
- **临时版本（未选中的 v2/v3）7 天后删除**（`service-cost-survey-2.md:322-400`）

### 4. IM 内置浏览器音频限制（微信/WhatsApp/iMessage）

`song-gift-plan-suggest-3.md:43-45` 提到但值得放大。收礼人多在 IM 内打开链接，这些环境**禁止自动播放**，需用户手势。`/gift/[id]` 首屏必须设计成"点击拆礼物"式的手势触发，不能依赖 autoplay。iOS 锁屏/后台播放行为也要单独测。

### 5. 时间敏感性 + 异步 = 错过日期 = 差评

`song-gift-plan-suggest-3.md:47-50`：生日/纪念日是固定日期，异步生成 + 队列可能让临时创作错过。**必须在产品里显式提示"建议提前 X 小时"**，并在高峰期排队时给明确预期。这条不做就是差评源头。

### 6. 反薅羊毛不只是每日登录

`song-gift-plan-suggest-2.md:73` 提了每日登录需设备指纹。再加一条：**新用户赠送积分需绑定"完成首个动作才到账"**（比如赠 20 分，但生成歌词后才解锁 10 分、分享后才解锁 10 分），避免账号农场批量注册即提。

---

## 四、补充建议（文档未明显涉及的）

### A. Prompt Pipeline 是真正的护城河

成本调研最后一节（`service-cost-survey.md:1722-2198`）建议的 **Prompt Transformation Pipeline** 是最有价值的部分，但被埋在文末：

```
用户结构化输入 → Song Blueprint (JSON) → Lyrics → Music Prompt → MiniMax
```

不要把用户故事直接丢给音乐模型。中间的「歌曲策划层」是你**区别于 Suno/通用工具的核心资产**，也是**降低首次重生成率**最有效的手段。建议把场景 Prompt 模板库作为长期资产维护（每个 SEO 场景页对应一套模板）。

### B. 第二首采用 lazy generation

成本调研（`service-cost-survey.md:1092-1175`）：原方案"一次生成两版"成本翻倍且大量浪费。改为：
- 生成 v1 → 试听 → "Love it" 或 "Create another version"
- v2 **按需生成**，成本降 30-50%
- 这是技术与体验双赢的设计

### C. AI 自检评分门控

`service-cost-survey.md:2114-2135`：歌词生成后让 GPT 评分（个性化/情感/故事/可唱性），<7 分自动重生（不收费、不展示给用户）。**在用户看不到的层做质量过滤**，比开放无限重生成便宜得多。

### D. A/B 测试框架

既然 Prompt 是核心资产，就要能持续优化。早期不一定需要复杂框架，但至少把每次生成的 prompt 模板版本号 + 用户满意度反馈（❤️🙂😐）记下来，形成迭代闭环。

### E. 失败/止损预案（`song-gift-plan-suggest-3.md:77-86`、`poc-checklist.md:156-164`）

POC 前就要定好"不通过怎么办"，不要等结论出来再慌：
- 音乐质量不过 → free-gift-maker 单独上线（固定音乐 + 贺卡）
- 或降级为"歌词朗诵 + 固定 BGM"（TTS + pixabay）
- Next.js on Workers 不过 → 切 Cloudflare Pages + Functions，或 Vercel

---

## 五、项目规划（分阶段）

拆成 5 个阶段，**前两个阶段是 POC，跑不过就停下来决策**。

### 阶段 0：POC（2-3 周，决定生死）

按 `poc-checklist.md:8-22` 的依赖关系，**音乐 POC 和 Next.js POC 并行**，unit economics 等音乐数据出来再做。

| POC | 验证什么 | 验收线 | 不过怎么办 |
|---|---|---|---|
| 音乐生成（1-2 周） | 带词人声 + **方差** + 成本 + 商用授权 | `poc-checklist.md:42-94` | 换模型 / 降级（仅歌词+BGM） |
| Next.js on Workers（1 周） | Google OAuth / Creem webhook / D1 事务 / R2 Range / Queues / Cron | `poc-checklist.md:99-110` | 换部署方案 |
| Unit economics（2-3 天） | 单首交付总成本含重生成系数 | 成本/售价 ≤ 0.7 | 调积分模型/赠送额度 |

**强制产出两份文档**（`poc-checklist.md:147-152`）：《模型选型与质量基线报告》《Unit economics 基准表 v1》。后者是后续所有促销活动的约束边界。

### 阶段 1：MVP 核心闭环（4-6 周）

只做验证飞轮所需的最小集：

```
Create Song Gift (单一入口)
   ├─ 选场景（仅 Birthday + Anniversary 2 个深度页，song-gift-plan-suggest-3.md:32）
   ├─ 故事 + 歌词生成（Prompt Pipeline 最小版）
   ├─ 音乐生成（v1 + lazy v2）
   ├─ 基础封面（默认 + AI 二选一，不上传）
   └─ 分享链接
/gift/[id]（打磨到位，移动端优先，IM 兼容，含下载）
积分系统（credit_grants + credit_transactions，FIFO，预扣）
Google 登录 + Creem 支付 + 价格页
异步任务（Queues + D1 任务表 + 邮件通知）
```

**显式砍掉**：mixed-gift-maker 的免费分支单独入口、PDF 歌词、图片上传、后台统计、多语言 UI、复杂场景页矩阵。

### 阶段 2：传播与转化优化（3-4 周）

- free-gift-maker 独立上线（免费入口引流）
- mixed-gift-maker 作为统一入口（融合 free + AI）
- 收礼页 CTA 优化（`song-gift-plan-suggest-1.md:425-450` 的情感化 CTA）
- 积分阶梯赠送（前 1k 用户 3n → 2n → n）
- 数据分析接入（PostHog/Plausible）+ Sentry

### 阶段 3：SEO 矩阵 + i18n（4-6 周）

- 每个场景页**独立真实文案 + 真实示例音频 + 真实案例**（`song-gift-plan-suggest-3.md:30-34`，避免 thin page / doorway）
- URL 用具体意图（`/birthday-song-gift` 而非 `/birthday`，`song-gift-plan-suggest-1.md:515-556`）
- 站点 i18n（呼应多语种音乐）
- 与 DigiFlower 联动（`song-gift-plan-suggest-1.md:453-512` 的 Digital Gift Network）

### 阶段 4：商业化深化（持续）

- Premium 档（MiniMax 2.6 等高级模型，$9.99-19.99）
- 订阅制（独立 source 类型的 subscription credits，`song-gift-plan-suggest-1.md:971-1009`）
- PDF 歌词、图片上传、歌词精修等增值功能回归
- 后台统计与运营工具

---

## 六、立即行动项（建议本周开始）

1. **建测试集**：5 场景 × 中英 2 语言 = 10 条固定 prompt（`poc-checklist.md:44`）
2. **跑 MiniMax Music 1.5 主测** + Suno API / Music 2.6 对照，重点是**方差测试**（每条跑 10 次）
3. **写 Next.js on Workers 最小骨架**，验 Google OAuth + Creem webhook + Queues
4. **建 unit economics 表模板**（等音乐数据填入）
5. **核实 pixabay 音乐再分发授权**（逐首）

> **建议：先不要写任何业务代码**。POC 三件套跑完、产出两份基线文档后再进入阶段 1。这能避免最大的风险——做了一半发现音乐质量不稳或 Workers 不兼容。
