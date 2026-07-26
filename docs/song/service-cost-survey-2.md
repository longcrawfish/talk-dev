# Cloudflare R2 
结合你现在设计的 **Song Gift**，Cloudflare R2 非常适合作为音乐文件存储。它的计费逻辑比较简单：

> **存储空间 + API 操作次数**
>
> 不是按流量收费（Egress 免费）。([Cloudflare Docs][1])

---

# 1. R2 主要收费项目

Cloudflare R2 Standard Storage：

| 项目          |              价格 |
| ----------- | --------------: |
| 存储          | $0.015 / GB / 月 |
| 写操作 Class A |     $4.50 / 百万次 |
| 读操作 Class B |     $0.36 / 百万次 |
| 公网下载流量      |              免费 |

免费额度：

| 项目        |      免费额度/月 |
| --------- | ----------: |
| 存储        | 10 GB-month |
| Class A 写 |       100万次 |
| Class B 读 |      1000万次 |

([Cloudflare Docs][1])

---

# 2. 放到 Song Gift 场景计算

你的流程：

```
用户创建歌曲

↓

MiniMax生成 mp3

↓

上传 R2

↓

gift page 播放歌曲

↓

用户分享链接
```

主要产生：

## A. 上传歌曲

一次：

```
PUT object
```

属于：

Class A

成本：

```
$4.5 / 1,000,000

≈ $0.0000045 / 次
```

几乎忽略。

---

## B. 用户播放歌曲

浏览器：

```
GET song.mp3
```

属于：

Class B

成本：

```
$0.36 / 1,000,000
```

也非常低。

---

# 3. 假设你的 Song Gift 数据量

假设：

每天：

```
100 个用户购买
```

每个生成：

```
2 首歌曲
```

一年：

```
100 × 365 × 2

=73,000 首歌曲
```

---

假设：

每首 MP3：

```
5 MB
```

存储：

```
73,000 × 5MB

=365GB
```

R2：

```
365 × $0.015

=$5.48/月
```

---

## 播放访问

假设：

每首歌曲：

100 次播放

一年：

```
73,000 ×100

=7,300,000 次播放
```

Class B:

免费：

10,000,000 次

所以：

```
$0
```

---

## 上传

73,000 次：

免费：

1,000,000

所以：

```
$0
```

---

### 总成本：

约：

```
$5~6/月
```

---

# 4. 和 AI 生成成本相比

你的 Song Gift：

单首：

MiniMax 1.5：

```
≈ $0.03
```

R2：

存一年：

```
≈ $0.00075/首/月
```

计算：

5MB：

```
0.005GB × $0.015

=$0.000075/月
```

一年：

```
$0.0009
```

---

也就是说：

## 成本比例：

```
AI生成音乐
        |
        |
        v
    $0.03


R2存储一年

    $0.0009
```

AI 成本约：

> 30倍以上

---

# 5. Song Gift 推荐 R2 存储结构

建议：

```
r2://song-gift/

├── songs/
│
│   ├── 2026/
│   │
│   │   ├── gift_id/
│   │   │
│   │   │   ├── version-a.mp3
│   │   │   └── version-b.mp3
│
├── covers/
│
│   └── gift_id.webp
│
└── previews/
    └── gift_id-preview.mp3
```

---

数据库只保存：

例如 D1：

```sql
songs

id
gift_id
user_id

audio_url
duration
style

created_at
```

不要把 MP3 放数据库。

---

# 6. 一个重要设计：不要永久保存所有版本

因为你前面讨论：

> 用户可能多次 regenerate

例如：

用户：

```
生成 A
生成 B
生成 C
生成 D
```

最后选择：

```
B
```

不要保存：

```
A
B
C
D
```

永久。

建议：

## 临时版本

```
/temp-song/

保存 7天
```

用户选择：

```
B
```

之后：

移动：

```
/songs/
```

其他删除。

---

例如：

```
temp/

gift123/
    v1.mp3
    v2.mp3
    v3.mp3


final/

gift123/
    final.mp3
```

---

# 7. 是否需要 Cloudflare CDN？

需要。

架构：

```
User

 ↓

Cloudflare CDN

 ↓

R2
```

优势：

* 全球访问快
* 降低 R2 GET 请求
* 分享页面体验好

---

# 8. 对你的项目，我建议

Song Gift：

```
Next.js
    |
    |
Cloudflare D1
(metadata/user/order)
    |
    |
Cloudflare R2
(audio/image)
    |
    |
MiniMax/Suno API
```

职责：

| 服务                | 用途              |
| ----------------- | --------------- |
| D1                | 用户、积分、订单、gift记录 |
| R2                | mp3、封面、动画资源     |
| Replicate/MiniMax | AI生成            |
| Cloudflare CDN    | 分发              |

---

# 9. 一个实际成本模型

假设：

每月：

1000 个付费用户

每人：

生成 2 首

```
2000 songs
```

音乐：

```
2000 × $0.03

=$60
```

R2：

歌曲：

```
2000×5MB

=10GB
```

存储：

```
≈$0.15/月
```

播放：

假设：

20万次：

免费额度覆盖。

所以：

你的主要成本：

> **AI 音乐生成，而不是文件存储。**

---

对于你的 Song Gift ：

**Cloudflare R2 + D1**

这个架构非常匹配：

* 音频文件天然适合对象存储
* 分享页全球访问
* 成本几乎可以忽略
* 和 Cloudflare Workers 集成方便

R2 在这个项目里不会成为成本瓶颈。真正需要重点优化的是前面讨论的 **AI generation retry 控制 + prompt 提升首次成功率**。

[1]: https://developers.cloudflare.com/r2/pricing/ "Pricing · Cloudflare R2 docs"

---

# 异步任务 Cloudflare Workers Queues

结合你的 **Song Gift** 项目，Cloudflare Workers Queues 非常适合用来处理：

```
用户点击 Generate Song

↓

创建任务

↓

Queue

↓

Worker Consumer

↓

调用 MiniMax / Suno API

↓

等待生成

↓

上传 R2

↓

更新 D1 状态

↓

通知用户
```

它的成本其实非常低，尤其你的场景（每个任务几十 KB 消息）。

Cloudflare Queues 按 **operation（操作次数）** 计费，而不是按任务时间计费。一个 operation 指 **64KB 数据的写入、读取或删除**。([Cloudflare Docs][1])

---

# 1. Cloudflare Queues 计费规则

## Workers Free

包含：

```
10,000 operations / day
```

消息保留：

```
24小时
```

([Cloudflare Docs][2])

---

## Workers Paid ($5/月)

包含：

```
1,000,000 operations / month
```

超出：

```
$0.40 / million operations
```

消息默认保留：

```
4天
```

最长可配置：

```
14天
```

([Cloudflare Docs][1])

---

# 2. 一个 Queue 消息实际消耗多少？

例如：

你发送：

```json
{
  "jobId": "abc123",
  "userId": "user001",
  "songId": "gift001",
  "style": "birthday"
}
```

大小：

可能：

```
200 bytes
```

远小于：

```
64KB
```

所以：

一次消息 = 1 operation

---

一个完整生命周期：

```
Producer Worker

写入 Queue
      |
      | 1 operation
      ↓

Consumer Worker

读取消息
      |
      | 1 operation
      ↓

删除消息
      |
      | 1 operation
```

总：

```
≈3 operations / task
```

([Cloudflare Docs][3])

---

# 3. 放到 Song Gift 计算

假设：

## 每天 1000 个歌曲生成任务

每天：

```
1000 messages
```

Queue：

```
1000 × 3

=3000 operations/day
```

一个月：

```
3000 × 30

=90,000 operations
```

---

Workers Free：

额度：

```
10,000/day
```

你的：

```
3000/day
```

完全免费。

---

# 4. 如果增长到 10万用户/月？

假设：

每月：

```
100,000 songs
```

每首：

一个生成任务。

Queue：

```
100,000 ×3

=300,000 operations
```

Paid：

包含：

```
1,000,000 operations
```

仍然：

```
$0
```

---

# 5. 百万级歌曲生成

假设：

每月：

```
1,000,000 songs
```

Queue：

```
3,000,000 operations
```

超过：

```
1,000,000 free
```

收费：

```
2,000,000 × $0.40 / million

=$0.8
```

非常便宜。

---

# 6. 真正需要注意的是 Retry

这里比正常任务更重要。

例如：

MiniMax API 临时失败：

```
Song Job

第一次失败
↓
retry

第二次失败
↓
retry

第三次成功
```

每次 retry：

增加 read operation。

([Cloudflare Docs][3])

所以：

设计：

```text
max retries = 3
```

足够。

---

# 7. Song Gift 推荐 Queue 设计

不要只有一个 Queue。

建议：

```
Queues

song-generation
        |
        |
        ↓

song-processing
        |
        |
        ↓

song-cleanup
```

---

## Queue 1：song-generation

用户任务：

```json
{
 type:"generate_song",
 giftId,
 userId,
 lyrics,
 style
}
```

Consumer：

调用：

```
MiniMax
```

---

## Queue 2：song-processing

生成完成：

```
MiniMax webhook

↓

Queue

↓

Worker

↓

download mp3

↓

R2
```

---

## Queue 3：cleanup

处理：

临时文件：

```
/temp/song-a.mp3

7天删除
```

---

# 8. 和 Next.js 对比

你的架构：

```
Next.js

    |
    |
Cloudflare Queue

    |
    |
Worker

    |
    |
MiniMax API

    |
    |
   R2
```

比：

```
Next.js API Route

↓

等待5分钟

↓

生成完成
```

好很多。

原因：

AI音乐生成：

可能：

30秒~几分钟。

HTTP request 不适合。

---

# 9. 结合你的积分系统

Queue 还能帮你做：

## 生成前

D1：

```
check credits

↓

deduct credits

↓

create job

↓

send queue
```

状态：

```
pending
processing
completed
failed
```

---

例如：

song_jobs 表：

```sql
id

user_id

gift_id

status

provider

model

cost

created_at
```

---

# 10. 推荐你的最终架构

结合前面讨论：

```
                 User

                  |
                  v

             Next.js

                  |
                  v

             D1
       (订单/积分/job)

                  |
                  v

        Cloudflare Queue

                  |
                  v

        Workers Consumer

                  |
        ----------------
        |              |
        v              v

   MiniMax API       R2

        |
        v

    Song Ready

        |
        v

   Gift Page
```

---

## 成本估算（早期）

假设：

1000 首/月：

Queue:

```
3000 operations/day
```

≈ 免费

Worker:

免费额度覆盖

R2:

几 MB

≈ 几分钱

真正成本：

```
MiniMax Music
1000 × $0.03

=$30
```

---

所以对于你的 Song Gift：

**Cloudflare Workers Queues 几乎可以忽略成本，但会显著提高系统可靠性。**

尤其你的场景有：

* AI 音乐生成（慢任务）
* Retry
* 积分扣费
* 状态追踪
* R2 上传

我认为 Queue 是非常值得加入的。

[1]: https://developers.cloudflare.com/workers/platform/pricing/ "Pricing · Cloudflare Workers docs"
[2]: https://developers.cloudflare.com/changelog/post/2026-02-04-queues-free-plan/ "Cloudflare Queues now available on Workers Free plan · Changelog"
[3]: https://developers.cloudflare.com/queues/platform/pricing/ "Cloudflare Queues - Pricing · Cloudflare Queues docs"
