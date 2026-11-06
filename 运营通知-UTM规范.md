# 📢 关于统一推广链接 UTM 参数规范的通知

**收件人**：运营部、市场部、设计部、品牌部
**发件人**：技术部
**日期**：2026-11-06
**重要程度**：🔴 紧急（2 周内必须全部落地）

---

## 一、问题背景（为什么要做这件事？）

### 我们花了广告费，但不知道用户从哪来

GA4 过去 28 天数据显示：

| 事实 | 数字 | 说明 |
|---|---|---|
| 总用户数 | 68,204 人 | — |
| **被归为 Direct（来源不明）** | **52,174 人（76.5%）** | 🔥 极度过高 |
| Paid Social（确认的 FB/Ins 付费） | 8,340 人（12.2%） | 明显偏低 |
| Email（Mailchimp 邮件） | 369 人（0.5%） | 明显偏低 |
| Organic Social（自然社交媒体） | 64 人（0.1%） | 几乎为 0 |

### 根因：推广链接没加 UTM 参数

GA4 只有看到 `?utm_source=xxx&utm_medium=xxx&utm_campaign=xxx` 这些参数，才能知道用户从哪来。我们运营发的推广链接**大部分没加**，GA4 只能把用户一律记成 Direct。

### 这意味着什么？

> 我们在 Facebook、Instagram、Google、Mailchimp 花了真金白银投广告，但 GA4 告诉我们"3/4 的用户不知道从哪来"。
> 
> → 我们不知道哪个渠道带的用户多
> → 不知道哪个素材（视频 vs 图片）转化率高
> → 不知道哪个受众（新客 vs 老客）ROI 好
> → 预算分配全凭感觉，无法数据驱动

---

## 二、解决方案：统一加 UTM 参数

### 一句话理解 UTM

在每个推广链接**后面加 3 个参数**，让 GA4 能识别：

```
原始链接：
https://121shoes.co.uk/products/hoff-bridge-mkii

加 UTM 后：
https://121shoes.co.uk/products/hoff-bridge-mkii
  ?utm_source=facebook        ← 从哪个平台来的
  &utm_medium=cpc             ← 付费广告还是自然流量
  &utm_campaign=xmas_boots_2026  ← 哪个具体活动
```

GA4 看到这个链接就知道：**这个用户来自 Facebook 的圣诞靴子付费广告系列**。

### UTM 规范字典（必须严格遵守，不许自己发明）

#### 🟡 utm_source（平台名，**5 个固定值**）

| utm_source | 适用场景 | 你发链接时这样填 |
|---|---|---|
| `facebook` | Facebook **所有**（付费 + 自然帖子） | ✅ |
| `instagram` | Instagram **所有**（付费 + 自然） | ✅ |
| `google_ads` | Google 搜索/展示/购物**付费广告** | ✅ |
| `mailchimp` | Mailchimp **邮件群发** | ✅ |
| `tiktok` | TikTok **付费 + 自然** | ✅ |

**不需要手动加**（GA4 自动识别）：
- 自然搜索 → 自动归为 Organic Search
- 直接输入网址 → 自动归为 Direct
- 其他网站跳转 → 自动归为 Referral

#### 🟡 utm_medium（渠道类型，**6 个固定值**）

| utm_medium | 适用场景 | 什么时候用 |
|---|---|---|
| `cpc` | 付费点击广告 | Facebook 广告、Google 广告、TikTok 广告 |
| `social` | 社交媒体自然流量 | 免费帖子、Story、Reels（非付费） |
| `email` | 邮件群发 | Mailchimp 营销邮件 |
| `video` | 视频广告 | YouTube / TikTok 付费视频 |
| `display` | 展示 banner | 网站侧边栏、弹窗广告 |
| `guest_post` | 博客合作 | 行业网站推荐我们的产品 |

#### 🟡 utm_campaign（活动名，**格式规范**）

**规则**：全部**英文小写 + 下划线**，不许中文、不许空格

| 场景 | ✅ 正确写法 | ❌ 错误写法 |
|---|---|---|
| 圣诞靴子推广 | `xmas_boots_2026` | 圣诞靴子 / xmas-boots-2026 |
| 冬季新品发布 | `launch_winter_2026` | Winter Launch / winter_new_arrivals |
| 黑色星期五 | `sale_bf_2026` | bf2026 / Black Friday |
| 会员日邮件 | `member_day_nov` | Member Day November / vip_day |
| Google 品牌词 | `brand_terms` | Brand Search / 品牌词 |
| 通用再营销 | `remarketing_general` | 再营销 / retargeting |

#### 🟢 utm_content（可选，素材区分）

同一个活动里要区分不同版本的广告时加：

| 区分什么 | 示例 |
|---|---|
| 新客 vs 再营销 | `audience_new` / `audience_retargeting` |
| 视频 vs 图片 | `creative_video_v1` / `creative_image_v2` |
| Feed vs Reels vs Story | `placement_feed` / `placement_reels` / `placement_story` |

### 完整示例（照着抄就行）

| 场景 | 完整 URL |
|---|---|
| Facebook Feed 广告 - 圣诞靴子 | `https://121shoes.co.uk/products/xxx?utm_source=facebook&utm_medium=cpc&utm_campaign=xmas_boots_2026&utm_content=feed_video_v1` |
| Instagram Reels 免费帖子 | `https://121shoes.co.uk/products/xxx?utm_source=instagram&utm_medium=social&utm_campaign=organic_daily_post` |
| Google Shopping 广告 | `https://121shoes.co.uk/products/xxx?utm_source=google_ads&utm_medium=cpc&utm_campaign=shopping_xmas_2026` |
| Mailchimp 会员日邮件 | `https://121shoes.co.uk/products/xxx?utm_source=mailchimp&utm_medium=email&utm_campaign=member_day_nov&utm_content=subject_early` |
| TikTok 付费视频广告 | `https://121shoes.co.uk/products/xxx?utm_source=tiktok&utm_medium=video&utm_campaign=launch_winter_2026` |

---

## 三、操作流程（超简单，30 秒搞定）

### 用 Google 官方 UTM 生成器（复制粘贴就行）

**地址**：https://ga-dev-tools.google.com/utm-campaign-builder/

### 操作步骤图解

```
┌─────────────────────────────────────────────────────────┐
│  Step 1：打开生成器页面                                  │
│  https://ga-dev-tools.google.com/utm-campaign-builder/  │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  Step 2：填表（3 个必填 + 1 个可选）                     │
│                                                         │
│  ✅ Website URL（必填）                                 │
│  填：https://121shoes.co.uk/products/hoff-bridge-mkii   │
│                                                         │
│  ✅ Campaign Source（必填）                             │
│  从字典选：facebook                                     │
│                                                         │
│  ✅ Campaign Medium（必填）                             │
│  从字典选：cpc                                          │
│                                                         │
│  ✅ Campaign Name（必填）                               │
│  按规范写：xmas_boots_2026                              │
│                                                         │
│  ⚪ Campaign Content（可选）                            │
│  素材区分：feed_video_v1                                │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  Step 3：复制生成好的完整 URL → 粘贴到推广平台          │
│                                                         │
│  https://121shoes.co.uk/products/hoff-bridge-mkii       │
│    ?utm_source=facebook&utm_medium=cpc                  │
│    &utm_campaign=xmas_boots_2026&utm_content=feed_video_v1│
│                                                         │
│  直接粘贴到：FB 广告后台 / Mailchimp / 飞书 / WhatsApp   │
└─────────────────────────────────────────────────────────┘
```

---

## 四、禁止事项（违者 GA4 数据还是会乱）

| ❌ 禁止 | 为什么 |
|---|---|
| 直接用不带 UTM 的裸链接 | GA4 记成 Direct，来源丢失 |
| source 写成 `FB` / `facebook.com` / `Meta` | GA4 识别不出来，归到 Unassigned |
| campaign 写成中文或带空格 | GA4 识别不出来 |
| 每个人发明不同的 source 写法 | 有人写 facebook，有人写 FB，有人写 Meta → GA4 出现三行"不同渠道"其实都是 Facebook |
| Google Ads 自动加的 UTM 不管 | Google Ads 后台可以自动加，但**运营在其他平台发的必须手动加** |

---

## 五、验收标准

| 指标 | 当前 | 2 周后目标 | 谁来查 |
|---|---|---|---|
| Direct 占比 | 76.5% | **降到 40% 以下** | 技术 |
| Paid Social 占比 | 12.2% | 上升到 20-30% | 技术 |
| Email 占比 | 0.5% | 上升到 2-5% | 技术 |
| Unassigned 人数 | 526 | 降到 100 以下 | 技术 |

**技术部会在 2 周后复查 GA4 数据，把结果同步给大家。**

---

## 六、常见问题 FAQ

### Q：我发 Facebook 广告时，Google Ads 那边说会自动加 UTM，我还要再加吗？
A：**要**。Google Ads 自动加的是 Google 自己的参数，Facebook 广告、Mailchimp 邮件、飞书分享这些不会自动加。我们要求**所有推广渠道的所有链接**都加。

### Q：太长的 UTM 参数会不会影响打开速度？
A：完全不会。URL 多 100 个字符对加载速度零影响。

### Q：同一个产品页，我给 Facebook 和 Instagram 推，campaign 要一样还是不一样？
A：**campaign 可以一样**（比如 `xmas_boots_2026`），source 不一样（一个 `facebook` 一个 `instagram`）。GA4 会按 source 区分。

### Q：我要推首页而不是产品页，可以加吗？
A：**可以**。UTM 加在任何 URL 后面都行，首页 `/`、集合页 `/collections/xxx`、产品页 `/products/xxx` 都支持。

### Q：已经发出去的推广链接没加 UTM，要重发吗？
A：**不用重发**。老链接的用户已经被记成 Direct 了，改不了。**从今天起新发的链接必须加**。

---

## 七、快速参考卡（保存到桌面）

```
╔══════════════════════════════════════════════════╗
║  121Shoes UTM 快速参考卡                          ║
╠══════════════════════════════════════════════════╣
║                                                    ║
║  生成器：ga-dev-tools.google.com/utm-campaign-builder ║
║                                                    ║
║  source 固定 5 个：                                ║
║    facebook | instagram | google_ads | mailchimp | tiktok ║
║                                                    ║
║  medium 固定 6 个：                                ║
║    cpc(付费) | social(自然) | email | video | display | guest_post ║
║                                                    ║
║  campaign 格式：全小写 + 下划线                     ║
║    例：xmas_boots_2026 | launch_winter_2026        ║
║                                                    ║
║  完整示例：                                        ║
║    ?utm_source=facebook&utm_medium=cpc             ║
║    &utm_campaign=xmas_boots_2026                   ║
║                                                    ║
╚══════════════════════════════════════════════════╝
```

---

**有问题直接在群里问技术部，随时解答。**

感谢配合！数据准确了才能知道每一分广告费花在了哪里 💪
