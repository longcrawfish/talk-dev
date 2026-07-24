我看完了这份 `song-gift-plan.md`。整体来说，这个规划已经比较成熟，尤其是结合 DigiFlower 的经验后，很多地方明显吸取了之前的实践。文档目前的核心方向：

> **免费礼物入口（free gift）获取流量 → AI personalized song 提供商业化 → `/gift/[id]` 负责传播裂变**

这个模型是正确的。下面从产品、SEO、商业化、技术实现几个角度提建议。

---

# 总体评价

我给这个方案：

| 方向              | 评价        |
| --------------- | --------- |
| 产品定位            | ★★★★★     |
| 商业模型            | ★★★★☆     |
| 传播模型            | ★★★★★     |
| MVP复杂度          | ★★★☆☆（偏高） |
| SEO潜力           | ★★★★★     |
| 和 DigiFlower 协同 | ★★★★★     |

最大的问题不是方向，而是：

> 第一版功能范围稍微偏大，需要收缩 MVP。

---

# 1. 网站结构：建议调整

目前：

```
首页
 |
 tools
 |- free-gift-maker
 |- AI-song-maker
 |- mixed-gift-maker
```

这个设计逻辑没问题。

但是我建议：

## 不要把三个工具都作为独立产品入口

原因：

用户不理解：

* free-gift-maker
* AI-song-maker
* mixed-gift-maker

这些名字偏开发者。

用户心智：

不是：

> 我要使用 AI-song-maker

而是：

> 我要制作生日歌曲礼物

---

建议改：

```
/
 |
 Create Song Gift

       |
       |
       v

Gift Builder

       |
       |
       +---- Free Song Gift
       |
       +---- AI Personalized Song
```

也就是：

**一个入口，内部选择模式。**

其实你的 mixed-gift-maker 已经接近这个方向。

---

# 2. 我最推荐保留 mixed-gift-maker

文档里：

> mixed-gift-maker 是两个工具的混合版

我认为它应该成为：

## 主产品

而：

* free-gift-maker
* AI-song-maker

变成内部模块。

原因：

用户第一次访问：

不知道自己需要 AI。

例如：

用户搜索：

```
birthday song gift
```

进入：

```
Create a Birthday Song Gift
```

然后：

第一步：

```
Choose your style

🎵 Personalized AI Song
Free Music Card
```

非常自然。

---

# 3. AI生成流程设计很好，但有一个重大调整

目前：

```
填写故事
 ↓
生成歌词
 ↓
预览
 ↓
生成音乐
 ↓
选择封面
 ↓
贺卡
```

这个流程非常专业。

但是商业产品建议：
 不要过早生成歌词

原因：
用户价值感来自：

> 听到音乐

不是：

> 看歌词。

现在 AI 音乐用户体验：

```
输入故事

↓

生成音乐

↓

播放惊喜

↓

查看歌词
```

更符合礼物场景。

建议：

改：

```
故事输入

↓

AI生成：

Lyrics
+
Music

↓

Preview

↓

修改歌词

↓

确认

```

减少等待和步骤。

---

# 4. 最大的问题：音乐生成等待时间

文档里面：

> 生成音乐等待3～5分钟，可以后台生成，用户继续下一步

这个思路很好。

但是我建议：
不要让用户等待。

采用：

## Async Gift Creation

流程：

```
用户提交

↓

生成任务

↓

立即进入：

"Your song is being created 🎵"

↓

填写：

封面
贺卡

↓

稍后回来

```

甚至：

发送 email：

```
Your song gift is ready!
```

这个非常符合礼物产品。

---

# 5. 积分模型建议调整

目前：

```
新用户积分
每日登录积分
购买积分

赠送积分30天
购买积分180天
```

这个设计比之前讨论的180天全部有效更好。

我建议：

## 三类积分

数据库：

```
credits

type:

welcome
daily
purchase
```

规则：

| 来源   | 有效期  |
| ---- | ---- |
| 注册赠送 | 30天  |
| 每日签到 | 30天  |
| 购买   | 180天 |

不要混合。

否则以后：

用户投诉：

> 为什么我买的积分过期？

---

# 6. AI积分消耗建议

不要一开始：

```
生成歌词扣积分
生成音乐扣积分
生成图片扣积分
```

太复杂。

用户不理解。

建议：

## 一个产品单位：

```
Create AI Song Gift

50 credits
```

内部：

拆：

```
Lyrics:
5

Music:
40

Cover:
5
```

但是用户看到：

```
Create Song
50 credits
```

即可。

---

# 7. `/gift/[id]` 是整个项目最重要页面

文档这里方向正确：

> 设计精美、有仪式感、移动端优先。

我认为：

这个页面价值甚至超过生成器。

因为：

一次创建：

```
1 creator
        |
        |
        v

1 gift page

        |
        |
        v

receiver

        |
        |
        v

new creator
```

这是增长飞轮。

---

建议增加：

## 收礼者 CTA

不要：

```
制作音乐
```

太普通。

改：

```
Create a Song Gift for Someone ❤️
```

或者：

```
Make Your Own Personalized Song 🎵
```

---

# 8. 和 DigiFlower 联动建议升级

你之前提出：

DigiFlower `/gift/[id]`

增加：

Replay a song

我建议进一步：

两个 gift 页面统一。

未来：

DigiFlower:

```
/gift/abc


🌹 Flowers

💌 Message


Create another gift
```

Song Gift:

```
/gift/xyz


🎵 Song

💌 Message


Create another gift
```

统一 CTA：

```
Create another digital gift
```

然后：

选择：

```
🌹 Flower
🎵 Song
💌 Card
```

---

# 9. SEO结构建议

目前场景页：

```
Birthday
Anniversary
Christmas
Friendship
...
```

方向正确。

但是建议 URL：

不要：

```
/birthday
```

太宽。

建议：

```
/birthday-song-gift

/anniversary-song-gift

/birthday-song-for-her

/birthday-song-for-him

/love-song-gift

```

因为搜索意图更明确。

---

# 10. 技术架构建议

当前：

```
Next.js
D1
R2
Workers
Creem
Replicate/OpenRouter
```

这个组合合理。

但是我有一个建议：

## 不要低估音频任务队列

需要：

```
song_jobs

id
user_id
status

pending
generating
completed
failed


provider_job_id

created_at
```

不要直接：
HTTP request → 等结果

一定异步。

---

# 11. MVP 建议缩减

现在文档：

包括：

* AI歌词
* AI音乐
* AI封面
* 上传图片
* PDF歌词
* 下载
* 草稿系统
* 三工具
* 积分
* 后台统计

比较大。

我建议 V1：

## Phase 1

只做：

```
mixed-gift-maker

↓

Free Music Card

↓

AI Song Gift

↓

Gift Page

↓

Payment
```

功能：

✅ Google 登录
✅ 积分
✅ AI音乐
✅ 分享页面
✅ 基础封面

暂时不要：

❌ PDF歌词
❌ 上传图片
❌ 下载音频
❌ 复杂草稿
❌ 后台

---

# 12. 我认为最重要的一点

这个项目不要定位：

> AI Song Generator

市场已经很多。

应该定位：

> **Personalized Song Gift**

关键词：

* gift
* birthday
* anniversary
* love
* surprise

你的竞争对手不是 Suno。

你的竞争对手是：

* 礼物网站
* 贺卡网站
* 花店网站

这也是为什么 DigiFlower 导流逻辑非常正确。

---

## 最终建议版本

目前这份方案已经有 80% 正确，我认为主要优化：

1. **mixed-gift-maker 做唯一入口**
2. **降低 MVP 范围**
3. **强化 `/gift/[id]` 传播**
4. **把 Song Gift 定位为礼物，而不是 AI 工具**
5. **和 DigiFlower 做 Digital Gift Network**

这样会比单独做一个 AI 音乐工具更有商业潜力。

---

# 积分使用：优先使用 有效期最短的积分

这个规则：

> **积分使用：优先使用有效期最短的积分**

我认为是非常正确的，而且应该作为 Song Gift 积分系统的核心规则。

行业里通常称为：

> **Expiry FIFO（First Expire First Out）**
>
> 先消耗即将过期的积分。([CleverTap User Docs][1])

对于你的场景（注册赠送、签到赠送、购买积分、多种有效期），这是最合理的设计。

---

## 1. 为什么不是「先获得先使用」？

假设：

用户积分：

| 来源   | 数量  | 过期   |
| ---- | --- | ---- |
| 注册赠送 | 20  | 30天  |
| 每日签到 | 50  | 30天  |
| 购买积分 | 500 | 180天 |

当前余额：

```
570 points
```

用户生成一首歌：

消耗：

```
50 points
```

---

### 如果采用「购买时间 FIFO」

可能：

```
购买积分 -50
```

结果：

30天后：

```
注册20
签到50

过期70
```

用户损失免费积分。

---

### 如果采用「最短有效期优先」

消耗：

```
注册赠送 -20

签到 -30
```

剩余：

| 来源 | 余额  |
| -- | --- |
| 签到 | 20  |
| 购买 | 500 |

30天后：

```
签到20过期
```

损失最小。

---

# 2. 推荐积分数据库设计

不要只保存：

```sql
users.credit_balance
```

这种方式后期很痛苦。

建议：

## credit_grants（积分批次）

例如：

```sql
credit_grants

id

user_id

amount

remaining

source

expires_at

created_at
```

示例：

| id | source      | amount | remaining | expire     |
| -- | ----------- | ------ | --------- | ---------- |
| 1  | welcome     | 20     | 20        | 2026-08-20 |
| 2  | daily_login | 5      | 5         | 2026-08-25 |
| 3  | purchase    | 500    | 500       | 2027-01-20 |

---

消费时：

排序：

```sql
ORDER BY expires_at ASC
```

例如：

用户生成歌曲：

需要：

```
50 credits
```

查询：

```sql
SELECT *
FROM credit_grants
WHERE user_id=xxx
AND remaining > 0
ORDER BY expires_at ASC
```

然后扣：

```
welcome:
20 -> 0


daily:
5 ->0


purchase:
25 ->475

```

---

# 3. 需要记录 credit_transactions

强烈建议增加：

```sql
credit_transactions
```

不要直接修改余额。

例如：

| type    | amount | description        |
| ------- | ------ | ------------------ |
| grant   | +20    | Welcome bonus      |
| grant   | +5     | Daily login        |
| consume | -50    | AI Song generation |
| expire  | -10    | Expired credits    |

这样以后：

用户：

> 我的积分去哪了？

可以展示：

```
Credit History

+20 Welcome Gift
+5 Daily Login

-50 Create AI Song

10 credits expired
```

---

# 4. UI上要显示即将过期积分

非常建议。

例如：

账户：

```
🎵 Melody Points

Balance:
235

⚠️ 15 points expire in 7 days
```

原因：

它会促进转化。

用户看到：

> 有积分快过期

更可能：

* 做一首歌
* 购买更多积分

---

# 5. 购买积分是否也180天？

你的设计：

> 购买积分180天

合理。

但是建议：

以后增加：

## Subscription credits

如果未来订阅：

例如：

```
Song Gift Pro

100 credits/month
```

不要混进去。

增加：

```sql
source:

welcome
daily
purchase
subscription
promotion
```

---

# 6. 一个小优化：消费顺序可以增加 priority

虽然目前：

```
expires_at ASC
```

够用。

但是未来：

比如：

活动赠送：

```
Christmas bonus
100 credits
7 days expire
```

希望优先消耗。

可以：

```sql
priority
```

字段：

规则：

```
1. priority ASC
2. expires_at ASC
3. created_at ASC
```

类似很多计费系统的做法。([Alguna][2])

---

# 7. 对 Song Gift 特别适合

因为你的积分来源天然分层：

## 免费用户

```
Register
+20

Daily
+2
```

目标：

体验 AI。

---

## 付费用户

```
Buy 500 points
```

目标：

大量制作。

---

## 最终消费：

永远：

```
免费积分
 ↓
活动积分
 ↓
购买积分
```

用户感觉：

> 系统没有浪费我的福利。

同时：

你的成本最低。

---

# 8. 我建议最终规则写入 AGENTS / 产品文档

可以定义：

```md
## Credit Consumption Rule

Credits are consumed using Expiry FIFO.

Priority:
1. Credits with earliest expiration date
2. If same expiration date, older credits first

Examples:
- Welcome credits expire in 30 days
- Daily login credits expire in 30 days
- Purchased credits expire in 180 days

The system always consumes the credits that expire soonest.
```

---

总结：

这个规则我建议直接采用：

✅ **Expiry FIFO（有效期最短优先）**
✅ 每批积分独立记录
✅ 不维护单一余额
✅ credit_transactions 记录流水
✅ UI显示即将过期积分

对于你的 Song Gift + DigiFlower 这种「免费获客 + 积分付费」模式，这是最稳的设计。

[1]: https://docs.clevertap.com/docs/create-loyalty-wallet?utm_source=chatgpt.com "Create Loyalty Wallet"
[2]: https://alguna.com/docs/credits/credit-consumption?utm_source=chatgpt.com "Credit Consumption - Alguna Docs"
