服务成本调研
# AI Music generation API
## 1. sunoapi
我看了一下 [SunoAPI.org](https://sunoapi.org/) 的定价和文档。结合你现在规划的 **personalized song gift（AI 定制歌曲礼物）**，重点关注「一首歌成本」。

## 1. SunoAPI.org credits 价格

目前它的 credits 包：

| 套餐         |    价格 | Credits | 单 Credit 成本 |
| ---------- | ----: | ------: | ----------: |
| Basic      |    $5 |   1,000 |      $0.005 |
| Standard   |   $50 |  10,000 |      $0.005 |
| Premium    |  $500 | 105,000 |   ≈$0.00476 |
| Enterprise | $1250 | 275,000 |   ≈$0.00455 |

([Suno API][1])

也就是：

> 1 credit ≈ $0.005

---

## 2. 生成一首歌曲需要多少 credits？

这里需要区分 SunoAPI 的具体接口模式。

SunoAPI 文档说明：

* 调用生成音乐接口 `/api/v1/generate`
* credits 会在生成音乐、歌词或其他处理功能时消耗。([Suno API][2])

但官网没有明确公开「一次生成扣多少 credits」。

不过根据它的套餐说明：

> Sonic 模型：1 generation = 10 credits = 2 songs
> Producer 模型：1 generation = 10 credits = 1 song

([AI Music API][3])

因此可以估算：

### Sonic 模型

一次 API generation:

```
10 credits
生成 2 首歌曲
```

成本：

```
10 × $0.005 = $0.05

单首：
$0.05 / 2 = $0.025
```

≈ **2.5 美分/首**

---

### Producer 模型

一次：

```
10 credits
= 1 song
```

成本：

```
10 × $0.005

= $0.05 / 首
```

≈ **5 美分/首**

---

# 3. 对你的 Song Gift 产品意味着什么？

你之前设计流程：

```
选择场景
 ↓
AI生成
 ↓
填写故事
 ↓
生成歌词
 ↓
生成音乐
 ↓
分享链接
```

实际上用户一次生成成功，后台成本不只是音乐。

应该拆：

## 成本 A：歌词生成

例如 GPT：

几十到几百 token

成本：

≈ $0.001 以下

基本忽略。

---

## 成本 B：Suno 音乐生成

假设：

### 乐观：

一次生成：

```
10 credits
2 songs
```

成本：

```
$0.025/song
```

---

### 实际产品流程

用户不会一次成功。

通常：

第一次：

```
生成版本 A
```

用户觉得：

"不错，但想换风格"

再次：

```
生成版本 B
```

甚至：

```
3~5 次
```

所以真实成本：

| 生成次数 |          成本 |
| ---- | ----------: |
| 1次   | $0.025~0.05 |
| 2次   |  $0.05~0.10 |
| 3次   | $0.075~0.15 |
| 5次   | $0.125~0.25 |

---

# 4. 如果设计积分系统，我建议这样

你之前设计：

> 新用户送积分
> 每日登录送积分
> 购买积分
> 积分有效期180天

这个方向合理。

但是不要直接：

```
1 credit = API credit
```

建议做自己的虚拟积分。

例如：

## 用户积分

```
1 Song Credit = 100 points
```

---

AI生成：

| 功能       |         消耗 |
| -------- | ---------: |
| 生成歌词     |  10 points |
| 生成歌曲(普通) | 200 points |
| 重新生成     | 200 points |
| 高级音乐模型   | 400 points |

---

你的内部成本：

```
200 points
≈ $0.05成本
```

---

# 5. 定价建议（结合 gift 场景）

你的产品不是音乐工具，而是：

> emotional gift

所以价格可以高很多。

竞品：
Songfinch 这类定制歌曲服务，价格通常远高于 AI 成本。

你的成本：

```
AI成本:
$0.05

售价:
$9.99
```

毛利：

```
99%
```

非常舒服。

---

我会设计：

## Free

```
固定模板歌曲
0积分
```

目的：

SEO + 分享传播。

---

## AI Song Gift

### Basic

$4.99

包含：

```
1首AI歌曲
1个分享页
```

成本：

约：

```
<$0.1
```

---

### Premium

$9.99

包含：

```
3次生成机会
选择音乐风格
高级分享页
背景动画
```

成本：

约：

```
$0.15~0.3
```

---

### Deluxe

$19.99

包含：

```
无限修改24小时
歌词精修
多版本
下载
```

---

总结：

| 项目             |          估算 |
| -------------- | ----------: |
| SunoAPI credit |     ≈$0.005 |
| 普通歌曲成本         | $0.025~0.05 |
| 考虑失败重生成        |      $0.1以内 |
| 推荐售价           | $4.99~19.99 |
| 毛利空间           |        90%+ |

对于你的 personalized song gift，我认为 **AI歌曲生成成本不是瓶颈，真正需要控制的是生成次数（retry）**。

建议积分规则里重点限制：

> "每次 AI generation 消耗积分，而不是最终歌曲数量"

否则用户可能无限试风格，把成本放大。

[1]: https://sunoapi.org/ "Most Stable and Pricing Affordable AI Music API"
[2]: https://docs.sunoapi.org/suno-api/generate-music "Generate Suno AI Music - Suno API"
[3]: https://api.sunoapi.com/en/pricing "AI Music API Pricing: Free Credits, Suno API & Producer AI"

---
## minmiax music
https://replicate.com/minimax/music-2.6
https://replicate.com/minimax/music-1.5
https://replicate.com/minimax/music-01

这几个 MiniMax Music 模型放到你的 **personalized song gift** 场景里，非常值得比较，因为它们和 SunoAPI 的商业模式差异很大。

我先看 Replicate 官方价格：

* Replicate 上的 MiniMax Music 2.6：**$0.15 / 首输出音频文件**。([Replicate][1])
* MiniMax Music 1.5：**$0.03 / 首输出音频文件**。([Replicate][2])
* MiniMax Music-01：**$0.035 / 首输出音频文件**。([Replicate][3])

---

# 1. 成本对比

| 模型                | 最大长度  | Replicate成本 |  单首成本 |
| ----------------- | ----- | ----------: | ----: |
| MiniMax Music-01  | 约1分钟  |      $0.035 | 3.5美分 |
| MiniMax Music 1.5 | 最长4分钟 |       $0.03 |   3美分 |
| MiniMax Music 2.6 | 最长6分钟 |       $0.15 |  15美分 |

([Replicate][3])

---

# 2. 三个模型定位差异

## MiniMax Music-01

定位：

> 快速生成短音乐 demo

特点：

* 支持歌词 + 人声
* 支持参考曲风
* 最大约1分钟

成本：

```
$0.035 / generation
```

适合：

✅ 短祝福片段
✅ preview
✅ 免费用户体验

不适合：

❌ 正式生日歌曲
❌ 情侣纪念歌曲

因为：

1分钟对于：

> "Happy Birthday to Jack..."

这种可以。

但是：

> Verse + Chorus + Bridge

明显不够。

([Replicate][3])

---

# 3. MiniMax Music 1.5

这是我认为最适合你第一版产品的模型。

特点：

* Full-length song
* 最长4分钟
* vocals
* rich instrumentation

成本：

```
$0.03/song
```

([Replicate][2])

你的流程：

```
用户故事
 ↓
GPT生成歌词
 ↓
MiniMax Music 1.5
 ↓
生成完整歌曲
 ↓
gift page
```

成本：

| 步骤     |      成本 |
| ------ | ------: |
| 歌词 GPT | <$0.001 |
| 音乐生成   |   $0.03 |
| 存储/CDN | <$0.001 |

总：

≈ **3~4美分**

---

# 4. MiniMax Music 2.6

这是旗舰。

特点：

* Full length
* 更长（最多6分钟）
* instrumental模式
* BPM/key控制
* 自动歌词
* 更强编曲

([Replicate][4])

成本：

```
$0.15/song
```

([Replicate][1])

相比1.5：

```
0.15 / 0.03

= 5倍成本
```

---

# 5. 放到 Song Gift 产品怎么选？

你的用户不是音乐制作人。

他们购买：

> "给妈妈生日做一首歌"

核心价值：

不是音质提升20%。

而是：

* 情感
* 个性化
* 分享页面
* 仪式感

所以成本敏感。

我会这样设计：

---

## Free 用户

固定歌曲：

```
Music-01
或者预生成歌曲
```

成本：

≈0

目的：

SEO + 分享。

---

## Basic AI Song

$4.99

使用：

```
MiniMax Music 1.5
```

成本：

```
1次生成:

$0.03
```

如果允许：

3次重新生成：

```
3 × 0.03

=$0.09
```

毛利：

```
4.99 - 0.09

≈98%
```

---

## Premium AI Song

$9.99

使用：

```
MiniMax Music 2.6
```

包含：

* 3 generations
* 高质量版本
* 下载
* 更多音乐风格

成本：

```
3 × $0.15

=$0.45
```

毛利：

```
≈95%
```

---

# 6. 一个关键问题：Retry 成本

你的产品最大风险不是单次生成。

而是：

用户：

```
生成
 ↓
不好
 ↓
再生成
 ↓
不好
 ↓
换风格
 ↓
再生成
```

假设：

MiniMax 1.5：

平均：

```
用户生成次数 = 3
```

成本：

```
3 × $0.03

=$0.09
```

仍然非常低。

MiniMax 2.6：

```
3 × $0.15

=$0.45
```

也可以接受。

---

# 7. 和 SunoAPI 对比

|       |    Suno API | MiniMax 1.5 | MiniMax 2.6 |
| ----- | ----------: | ----------: | ----------: |
| 成本    | $0.025~0.05 |       $0.03 |       $0.15 |
| 歌曲长度  |        约几分钟 |         4分钟 |         6分钟 |
| 商业API |         第三方 |   Replicate |   Replicate |
| 稳定性   |        看API |           高 |           高 |
| 成本控制  |          一般 |          优秀 |          一般 |

---

# 8. 我的建议

我会采用：

## MVP

```
GPT-4.1 mini
       |
       |
MiniMax Music 1.5
       |
       |
 gift page
```

原因：

成本：

```
<$0.05/song
```

体验：

已经足够。

---

## 后续 Premium

增加：

```
"Studio Quality Song"

MiniMax Music 2.6
```

收费：

$14.99

---

# 推荐最终架构

```
用户
 |
选择场景
 |
写故事
 |
GPT生成歌词
 |
 ------------------
 |                |
Basic            Premium
 |                |
Music 1.5        Music 2.6
$0.03            $0.15
 |
生成分享页
```

对于你的产品，我认为 **MiniMax Music 1.5 是最佳 MVP 模型**。

它的成本几乎和 Suno 最低档接近，但完整歌曲能力更适合「礼物场景」。MiniMax 2.6 更适合作为后续 $9.99~19.99 的高端升级项。

[1]: https://replicate.com/minimax/music-2.6/api "minimax/music-2.6 | Get started"
[2]: https://replicate.com/minimax/music-1.5 "MiniMax Music 1.5 | AI Music Generation API"
[3]: https://replicate.com/minimax/music-01 "MiniMax Music 01 | AI Music Generation API"
[4]: https://replicate.com/minimax/music-2.6 "MiniMax Music 2.6 | AI Music Generation API"

---
# 在 song gift 工具设计中：每次一套歌词生成 2首歌，给用户作选择；
这种设计，如何使用上述 AI API ？

你的设计：
> **一套歌词 → 生成 2 首不同歌曲 → 用户选择喜欢的一首**

其实非常符合 **song gift** 场景，因为用户购买的不是「音乐生成次数」，而是「最终得到一首满意的礼物」。

但这会影响 AI API 调用策略和成本控制。

---

## 1. 先看三个模型是否支持“一次生成两首”

### MiniMax Music（Replicate）

Replicate 的模型调用逻辑：

```
1次 prediction
=
1个生成任务
=
1个输出音频
```

也就是说：

如果你想得到：

```
Lyrics A

↓


Song 1
Song 2
```

通常需要：

```
API Call #1
歌词 + style prompt
        ↓
Song 1.mp3


API Call #2
歌词 + style prompt
        ↓
Song 2.mp3
```

也就是：

> **两次调用，不是一次调用返回两个歌曲。**

---

## 2. 对你的流程重新设计

推荐：

```
用户填写故事

        ↓

LLM生成歌词

        ↓

用户确认歌词

        ↓

Music API

        ↓

--------------
|            |
Song A       Song B
Version 1    Version 2
--------------
        ↓

用户选择
        ↓

生成 Gift Page
```

---

# 3. 成本计算

假设使用 MiniMax Music 1.5：

单首：

```
$0.03
```

一次生成两个版本：

```
Song A
+
Song B

=2 × $0.03

=$0.06
```

加歌词：

GPT：

≈ $0.001

总：

```
≈ $0.061
```

---

如果使用 Music 2.6：

```
2 × $0.15

=$0.30
```

---

# 4. 积分设计应该围绕“生成套餐”

不要让用户理解：

> 生成一次扣多少积分

用户理解的是：

> 创建一首定制歌曲

建议：

## Song Gift Credit

例如：

```
1 AI Song Gift
=
生成2个歌曲版本
```

内部：

```
1 Song Gift
消耗:

Lyrics generation
+
2 × Music API
```

---

积分：

例如：

### 免费注册

送：

```
100 credits
```

只能：

```
体验歌词生成
```

或者：

```
试听低成本模型
```

---

### AI Song

售价：

$4.99

赠送：

```
1000 credits
```

内部：

```
生成2首歌曲

消耗:

600 credits
```

---

## 5. 更好的成本优化方案

我建议不要一开始直接生成两首完整歌曲。

因为：

用户可能：

```
打开页面
填写故事
生成歌词
生成音乐

然后离开
```

你的成本已经产生。

更好的流程：

---

## Step 1

生成歌词：

成本：

≈0

展示：

```
Your song lyrics:

Verse
Chorus
...
```

用户：

👍满意

继续：

```
Create Music
```

---

## Step 2

生成两个版本

例如：

### Version A

Prompt:

```
Warm acoustic guitar,
emotional,
female vocal
```

### Version B

Prompt:

```
Pop ballad,
cinematic,
male vocal
```

两次调用。

---

## 6. 更进一步：延迟生成第二首

这是我认为最适合你的商业设计。

流程：

第一次：

```
Generate Song
```

后台：

```
生成 Song A
```

展示：

```
Your song is ready ❤️

[Play]

Want another style?

Create another version
```

用户点击：

```
Generate another version
```

才调用第二次。

这样：

假设：

100个用户：

### 方案A：

全部生成2首：

```
100×2×$0.03

=$6
```

---

### 方案B：

70%满意：

```
100×1×$0.03

=$3
```

30%重新生成：

```
30×$0.03

=$0.9
```

总：

```
$3.9
```

节省35%。

---

# 7. 对 Song Gift，我推荐最终方案

结合你的 DigiFlower 联动，我会设计：

```
用户输入故事

↓

AI生成歌词

↓

Preview Lyrics

↓

Create My Song

↓

MiniMax Music 1.5

↓

Song A

↓

用户试听

↓

按钮：

❤️ Love it
🔄 Create another version

↓

需要时生成 Song B

↓

生成 gift page
```

---

## 积分模型：

| 动作     |  积分 |
| ------ | --: |
| 生成歌词   |  免费 |
| 生成第一首歌 | 500 |
| 生成第二首歌 | 500 |
| 下载高清音频 | 200 |
| AI润色歌词 | 100 |

---

内部：

```
500 points
≈ $0.03成本
```

---

## 我的判断

对于你的产品定位：

**不要把“2首歌曲”作为后台固定成本。**

更好的营销文案：

> "Every song comes with a free alternative version"

但技术实现：

> 首歌按需生成，第二首 lazy generation。

这样用户体验接近“两首选择”，但你的 API 成本降低约30~50%。这对早期流量产品非常重要。

---
> 首歌按需生成，第二首 lazy generation —— 这个设计不错。
> 
> 关于假如用户不满意，需要多次重新生成音乐，这一块如何控制？

这个问题非常关键。对于 **AI Song Gift**，真正的成本风险不是第一次生成，而是：

> 用户把它当成「无限试音工具」，不断点击 regenerate。

虽然 MiniMax Music 1.5 单次成本只有 $0.03，但如果 100 个用户中有人反复生成 10 次，成本和队列压力都会快速增加。

建议从 **产品设计 + 积分机制 + 技术限制** 三个层面控制。

---

# 1. 不要叫「重新生成」，叫「创建新版本」

这是心理设计。

不要：

> ❌ Regenerate

用户理解：

> 无限重试

改成：

> ✅ Create another version
> ✅ Try another style
> ✅ Generate a new interpretation

让用户意识：

> 这是创建一个新的作品，需要消耗资源。

---

# 2. 采用「套餐包含生成次数」

不要卖：

> 一首歌

应该卖：

> Song Creation Package

例如：

## Free Preview

```text
生成歌词
试听30秒 Demo
```

成本：

≈0

---

## Basic Song Gift $4.99

包含：

```
1首完整歌曲
+ 1次修改机会
```

后台：

```
第一次生成:
Song A

第二次:
Song B
```

成本：

```
2 × $0.03

=$0.06
```

---

## Premium $9.99

包含：

```
3个歌曲版本
歌词优化
高级音乐风格
下载
```

成本：

```
3 × $0.03

=$0.09
```

---

# 3. 积分体系控制（推荐）

你之前设计：

> 积分有效期180天

这个非常适合。

建议：

## 内部积分

例如：

```
1000 points = $5
```

---

消耗：

| 动作      |  积分 |
| ------- | --: |
| 歌词生成    |  免费 |
| 生成第一版歌曲 | 500 |
| 生成第二版歌曲 | 500 |
| 生成第三版歌曲 | 500 |
| AI修改歌词  | 100 |

---

用户：

购买：

$4.99

获得：

```
2000 points
```

可以：

```
Song A
+
Song B
+
歌词优化
```

---

# 4. 最重要：限制 regenerate 的入口

不要在播放器旁边放：

```
[Regenerate]
```

这样用户会疯狂点。

推荐：

第一次：

```
Your song is ready ❤️


▶ Play


Looks good?
[Create Gift Page]


Want another style?
[Create another version]
```

---

第二次：

```
Version 2 is ready.


Choose your favorite:

○ Version 1
○ Version 2


Need another variation?

Upgrade your song package
```

---

# 5. 设置每日/IP/账号限制

## 登录用户

免费：

```
1 song generation
```

---

## 付费用户

按照积分。

---

# 6. 防止「垃圾生成」的技术策略

## A. 生成前确认

不要用户填写：

```
Jack birthday
```

马上调用音乐 API。

增加：

```
Step 1
Review lyrics

Step 2
Generate music
```

降低误操作。

---

## B. 生成任务队列

不要：

```
POST
 ↓
Replicate
 ↓
等待
```

推荐：

```
User Request

↓

Create Job

↓

Queue

↓

Worker

↓

Replicate API

↓

Result

↓

Notify
```

好处：

以后可以：

* 优先付费用户
* 限流
* 重试控制

---

# 7. 增加「满意度反馈」

生成完成：

不要马上：

```
Regenerate
```

增加：

```
How do you like it?

❤️ Perfect
🙂 Good
😐 Needs changes
```

如果：

❤️

引导：

```
Create Gift Page
```

如果：

😐

问：

```
What should change?

○ More emotional
○ Different voice
○ Faster tempo
○ Different style
```

然后生成 prompt。

---

# 8. Prompt 优化比无限生成更重要

很多“不满意”其实不是模型问题，而是 prompt。

例如用户：

```
给妈妈生日歌曲
```

直接：

```
birthday song for mom
```

效果普通。

优化：

```
A heartfelt acoustic pop ballad.
Warm female vocal.
Theme:
A daughter expressing gratitude to her mother...
```

一次成功率提高。

---

# 9. 一个适合你项目的最终方案

我会这样设计：

```
                User

                 |
          Write memory/story

                 |
          AI Lyrics

                 |
          Confirm lyrics

                 |
          Generate Song A
          (500 points)

                 |
        -----------------
        |               |
      Love it       Create Version 2
        |               |
   Gift Page       (500 points)

                 |
          Select final song
```

套餐：

| 套餐      |     价格 |                 歌曲版本 |
| ------- | -----: | -------------------: |
| Basic   |  $4.99 |           2 versions |
| Premium |  $9.99 |           5 versions |
| Deluxe  | $19.99 | 10 versions + extras |

---

我甚至认为可以进一步做：

> **AI Song Gift = 情感创作，不是音乐生成器**

所以不要突出：

* regenerate
* generations

而突出：

* Create your memory song
* Make another version
* Choose the perfect feeling

这样既控制成本，又符合礼物用户心理。

----

# Prompt 优化方案

对于 **personalized song gift**，Prompt 优化是决定用户满意度的核心。因为用户通常不是音乐专家，他们给出的输入非常简单：

> "给妈妈生日写一首歌"

但 AI 音乐模型需要的是：

* 情绪（emotion）
* 人物关系（relationship）
* 故事细节（story）
* 音乐风格（genre）
* 演唱方式（vocal）
* 编曲结构（arrangement）

所以不要直接把用户输入丢给 MiniMax/Suno，而应该设计一个 **Prompt Transformation Pipeline（提示词转换层）**。

---

# 1. 推荐架构

不要：

```
User Story
    |
    ↓
Music API
```

而是：

```
User Input
    |
    ↓
Story Understanding AI
    |
    ↓
Song Blueprint
    |
    ↓
Lyrics Generation
    |
    ↓
Music Prompt Generation
    |
    ↓
MiniMax Music
```

中间增加一个「歌曲策划层」。

---

# 2. 用户输入设计

不要只让用户：

> Write your story

建议结构化收集。

例如：

## Step 1：关系

```
Who is this song for?

○ Mom
○ Dad
○ Partner
○ Friend
○ Child
○ Other
```

---

## Step 2：场景

```
What's the occasion?

○ Birthday
○ Anniversary
○ Graduation
○ Wedding
○ Thank you
○ Just because
```

---

## Step 3：故事

自由输入：

```
Tell us a special memory
```

例如：

> My mom raised me alone. She always cooked dinner after work. I want to thank her.

---

## Step 4：音乐感觉

不要让普通用户选择：

```
C major
BPM 120
```

而是：

```
What feeling do you want?

○ Warm & emotional
○ Happy & uplifting
○ Romantic
○ Nostalgic
○ Epic & cinematic
```

---

# 3. AI 生成 Song Blueprint

例如用户输入：

```
Recipient:
Mom

Occasion:
Birthday

Story:
My mom raised me alone...
```

GPT 不直接写歌词。

先生成：

```json
{
  "theme": "gratitude and unconditional love",
  "emotion": [
    "warm",
    "nostalgic",
    "heartfelt"
  ],
  "relationship": "daughter to mother",
  "story_points": [
    "mother's sacrifice",
    "childhood memories",
    "thank you"
  ],
  "song_style": {
    "genre": "acoustic pop ballad",
    "tempo": "slow",
    "instrument": [
      "piano",
      "acoustic guitar",
      "strings"
    ],
    "vocal": "warm female vocal"
  }
}
```

这个就是内部「歌曲设计稿」。

---

# 4. Lyrics Prompt

然后生成歌词。

Prompt：

```
You are a professional songwriter.

Write a heartfelt personalized song.

Recipient:
Mother

Occasion:
Birthday

Story:
...

Requirements:
- Include specific memories
- Avoid generic birthday phrases
- Make it emotionally authentic
- Structure:
  Verse 1
  Verse 2
  Chorus
  Bridge
  Final Chorus

Style:
Warm acoustic pop ballad
```

---

重点：

不要：

> write a birthday song

而是：

> write a daughter-to-mother emotional story song

效果差距很大。

---

# 5. Music Prompt

音乐模型需要另外一个 Prompt。

不要把完整歌词+故事全部塞进去。

生成：

```
Style:
Warm acoustic pop ballad

Mood:
Heartfelt, nostalgic, emotional

Vocal:
Female warm vocal

Instrumentation:
Piano, acoustic guitar, soft strings

Tempo:
Slow, 75 BPM

Production:
Modern clean studio recording,
intimate and emotional
```

---

# 6. 两个版本如何设计？

你之前：

> 一套歌词生成两首歌

非常适合做 Prompt Variation。

不要随机生成。

设计：

## Version A：Emotional

```
Style:
Emotional acoustic ballad

Mood:
Warm, intimate, nostalgic

Vocal:
Soft female vocal

Instrument:
Piano + guitar + strings
```

---

## Version B：Cinematic

```
Style:
Cinematic pop ballad

Mood:
Inspirational, uplifting

Vocal:
Powerful vocal

Instrument:
Orchestra + drums + piano
```

用户会感觉：

> 两首都是我的歌，但感觉不同。

---

# 7. 建立场景 Prompt 模板库

这是你的长期资产。

例如：

## Birthday Mom

```
Genre:
Acoustic pop ballad

Emotion:
Love, gratitude, warmth

Instrument:
Piano, guitar, strings

Vocal:
Warm female vocal
```

---

## Long Distance Boyfriend

```
Genre:
Romantic pop

Emotion:
Missing, love, hope

Instrument:
Piano, soft beats

Vocal:
Male emotional vocal
```

---

## Graduation

```
Genre:
Inspirational pop

Emotion:
Pride, achievement, future

Instrument:
Piano, orchestra
```

---

未来你的 SEO 页面：

```
/birthday-song-generator

/anniversary-song-generator

/graduation-song-generator

/wedding-song-generator
```

每个页面其实对应：

> 一个 Prompt 模板。

---

# 8. 建议加入 AI 自检评分

生成歌词后：

让 GPT 评分：

```
Rate this song from 1-10:

- Personalization
- Emotional impact
- Story details
- Singability
```

如果：

<7

自动重新生成。

这样减少用户看到低质量结果。

---

# 9. 对你的 MVP，我建议不要做复杂 Agent

第一版：

```
User Form

↓

GPT-4.1 mini
(generate blueprint)

↓

GPT-4.1 mini
(generate lyrics)

↓

GPT-4.1 mini
(generate music prompt)

↓

MiniMax Music 1.5
```

成本：

GPT：

<$0.005

Music：

$0.03

总：

约 $0.035

---

## 我的建议总结

对于 Song Gift，Prompt 优化优先级：

| 优化项                | 重要性   |
| ------------------ | ----- |
| 结构化收集用户故事          | ★★★★★ |
| Song Blueprint 中间层 | ★★★★★ |
| 场景模板库              | ★★★★★ |
| 两个版本 Prompt 差异化    | ★★★★  |
| 自动质量评分             | ★★★★  |
| 无限 regenerate      | ★★    |

你的产品最大的护城河不会是调用 MiniMax 或 Suno，而是：

> **把普通人的一句话故事，自动转化成一首像专业作词人创作的礼物歌曲。**

这个 Prompt Pipeline 会比换模型带来的提升更明显。
