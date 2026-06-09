# Monetization Playbooks -- tiktok-script-gen v1

> **version**: v1
> **purpose**: Given `monetization_goal`, determine CTA type, copy style, video rhythm strategy, and compliance boundaries. Each playbook defines the complete content strategy for one monetization path.
> **used_in**: SKILL.md Step 4 (Copy Generation)
> **references**: tiktok-video-gen-report.md §5 (monetization paths), §10 (conclusions)

---

## Playbook Selection (Decision Tree)

```
User monetization_goal?
├── crp         → §1. CRP Playbook (Creator Rewards Program)
├── shop        → §2. Shop Playbook (TikTok Shop / E-commerce)
├── brand_deal  → §3. Brand Deal Playbook (Sponsorship)
├── live_funnel → §4. Live Funnel Playbook (Livestream Traffic)
└── awareness   → §5. Awareness Playbook (Follower Growth / Brand Building)
```

---

## 1. CRP Playbook (Creator Rewards Program)

### Goal
Maximize **completion rate** + **interaction rate** (likes, comments, shares). These two signals directly drive FYP recommendation weight and RPM.

### Content Tone
- **Native, personal, authentic**: feels like the creator is talking to a friend, not selling.
- **No hard sell**: no price mentions, no "buy now", no discount codes.
- **Value-first**: the content itself is the reward. CTA is a natural extension, not an interruption.
- Creator's personality is the differentiator.

### CTA Type
`comment_question` / `like_prompt` / `share_bait`

The CTA must feel organic -- an invitation to participate, not a demand.

### CTA Copy Templates

#### English (3 templates)

| Template ID | CTA Text | Best Context | Timing |
|---|---|---|---|
| crp-en-01 | "Which one would you pick? Tell me in the comments!" | Comparison/ranking content, product roundups | Last 5s |
| crp-en-02 | "Drop a 🔥 if you agree -- and follow for part 2!" | Opinion/controversial take, series content | Last 8s (composite CTA: like + follow) |
| crp-en-03 | "Tag someone who needs to hear this 👇" | Advice/educational content, relatable humor | Last 5s |

#### Chinese (3 templates)

| Template ID | CTA Text | Best Context | Timing |
|---|---|---|---|
| crp-zh-01 | "你觉得呢？评论区告诉我！" | 观点/测评内容，引导讨论 | 倒数 5s |
| crp-zh-02 | "觉得有用的点个赞，下期继续讲！" | 教程/知识类，系列内容 | 倒数 8s (复合：点赞+关注) |
| crp-zh-03 | "@一个需要的朋友，这期干货很多！" | 技巧分享/省钱攻略，转发性强 | 倒数 5s |

### Video Rhythm Strategy

```
Timeline (for 75s video, proportion %):
 0-3s   (0-4%):   Hook -- attention grab, no CTA
 3-15s  (4-20%):  Value Show -- "Did you know...?" type teaser engagement point at 40-50%
 15-35s (20-47%): Body -- main content delivery; mid-roll engagement prompt at 55-65%
 35-55s (47-73%): Deep Demo -- detailed explanation; peak content value
 55-68s (73-91%): Climax/Twist -- emotional peak or reveal; build to CTA
 68-75s (91-100%): CTA -- soft, natural, inviting
```

**Key rhythm rules**:
- **First soft engagement prompt** at ~40-50% of total duration. Example: "Did you know [surprising stat]?" or "Wait till you see what happens at the end." This primes the viewer to engage later without disrupting flow.
- **No CTA before 80%** of video duration. Premature CTA triggers early drop-off.
- **CTA is the last element**, not "don't forget to like and subscribe, bye!" filler. The content ends, CTA appears, video ends. Clean.
- Leave the hook's question unanswered until the climax -- this drives completion.

### Typical Example

Topic: "3 lipstick shades for pale skin"
Niche: beauty | Audience: 18-24, US | Duration: 75s | Language: en

Rhythm: 0-3s (hook: red lipstick slams onto marble) → 3-15s ("I've tested 47 shades -- here are the 3 that actually work.") → 15-35s (shade 1 demo with half-face) → 35-55s (shades 2+3, side-by-side comparison) → 55-68s (the surprise: cheap shade outperforms luxury) → 68-75s (CTA: "Which shade should I try next? Comment below!")

---

## 2. Shop Playbook (TikTok Shop / E-commerce)

### Goal
Maximize **conversion rate** (click-through to product page, add-to-cart, purchase). Views and interactions are secondary; revenue per view is primary.

### Content Tone
- **Pain → Solution → Urgency**: classic direct-response structure adapted for short-form video.
- **Credible, not pushy**: demonstrate value with evidence (close-ups, tests, comparisons), not superlatives.
- Product is the hero; creator is the trusted recommender.
- **Transparency**: disclose sponsorship/commercial intent per platform rules.

### CTA Type
`shop_cta`

The CTA must include a **clear action instruction** + **urgency driver** (time/quantity limit) + **visual cue** (where to tap/click).

### CTA Copy Templates

#### English (3 templates)

| Template ID | CTA Text | Best Context | Timing |
|---|---|---|---|
| shop-en-01 | "Link in my bio -- 50% off for the next 24 hours!" | Time-limited flash sale, new launch | Last 8s |
| shop-en-02 | "Tap the yellow basket to grab yours -- only 100 left!" | Scarcity-based, limited stock | Last 5s |
| shop-en-03 | "I've linked everything below. The [specific item] is the one I use daily." | Multi-product recommendation, authenticity focus | Last 8s |

#### Chinese (3 templates)

| Template ID | CTA Text | Best Context | Timing |
|---|---|---|---|
| shop-zh-01 | "链接在主页，24 小时限时 5 折！" | 限时促销，新品首发 | 倒数 8s |
| shop-zh-02 | "点下方小黄车，库存只剩 100 件！" | 限量稀缺，爆款返场 | 倒数 5s |
| shop-zh-03 | "全部链接放下面了，[产品名] 是我每天在用的那一款。" | 多品推荐，真实体验背书 | 倒数 8s |

### Video Rhythm Strategy

```
Timeline (for 60s video, proportion %):
 0-3s   (0-5%):   Hook -- pain-point with product visible (product MUST appear here)
 3-12s  (5-20%):  Pain Amplification -- deepen the problem; viewer self-identifies
 12-27s (20-45%): Solution Intro -- "This is what I found..."; first benefit demo
 27-42s (45-70%): Product Demo -- close-ups, tests, comparisons; key selling points
 42-55s (70-92%): Social Proof / Urgency -- reviews, results, stock warning
 55-60s (92-100%): CTA -- clear action + urgency driver
```

**Key rhythm rules**:
- **Product MUST appear within first 10s** (mobile users scroll fast; if they haven't seen the product by 10s, they're gone).
- Pain amplification should be **specific and relatable**, not generic ("tired of messy kitchen drawers" > "tired of kitchen problems").
- Product demo segment (27-42s) is the **conversion zone**: this is where purchase intent forms. Spend the most time here.
- Only ONE product per 60s video. Multi-product videos dilute conversion focus and confuse the CTA.
- Never mention price in the hook; price anchoring happens during the demo segment.

### Compliance & Legal Notes (TikTok Shop Specific)

| Category | Prohibited | Allowed |
|---|---|---|
| **Price claims** | "全网最低价" / "cheapest anywhere" | "今天特价" / "today's deal" (factual, time-bound) |
| **Efficacy claims** | "100% 有效" / "guaranteed results" / "永久去除" | "很多用户反馈有效" / "many users report improvement" / visible before/after with disclaimer |
| **Urgency** | "最后一件" / "never available again" (if untrue) | "本次库存有限" / "limited stock this batch" (if factually limited) |
| **Medical claims** | "治愈" / "治疗" / "heals" / "treats" (for non-medical products) | "帮助改善" / "helps with" / describe mechanism, not medical outcome |
| **Competitor comparison** | "比 X 品牌好 10 倍" / "10x better than [brand]" | "我用了 3 款，这款最适合我" / "I tried 3, this one works best for me" (subjective) |
| **Endorsement** | Implying official certification without evidence | "我个人的推荐" / "my personal recommendation" |

If any prohibited phrasing is detected, auto-rewrite using the allowed alternative closest in meaning. Record in `compliance_check.banned_words_hit[]`.

### Typical Example

Topic: "Kitchen organizer that saves counter space"
Niche: shop | Audience: 25-34, US | Duration: 60s | Language: en

Rhythm: 0-3s (hook: messy spice drawer, can't close) → 3-12s ("I cook every day and this drawer has been driving me crazy for 6 months.") → 12-27s ("Then I found this expandable organizer..." show unboxing) → 27-42s (organize drawer live, split-screen before/after, measure saved space) → 42-55s ("Over 2000 reviews, 4.8 stars, and it fits ANY drawer size.") → 55-60s (CTA: "Link in bio -- 30% off right now!")

---

## 3. Brand Deal Playbook (Sponsorship)

### Goal
Maximize **brand exposure** + **brand favorability** (positive association, not hard conversion). The audience should feel "this brand is cool/trustworthy" not "I'm being sold to."

### Content Tone
- **Natural integration**: the brand is part of the story, not the story itself.
- **Authentic endorsement**: real usage, real opinion. Creator credibility is the brand's asset.
- **Soft sell**: no urgency drivers, no limited-time offers (unless specified by brand_constraint).
- Respect the audience's intelligence -- they know it's a sponsorship. Transparency builds trust.

### CTA Type
`composite` (soft brand mention + mild engagement ask)

The CTA is a composite of brand acknowledgment + natural engagement prompt. Never "go buy this now." The CTA is the **softest** of all playbooks.

### CTA / Voiceover Templates

#### Opening Integration (within first 15s, natural brand intro)

| Template ID | Voiceover Text | Best Context |
|---|---|---|
| brand-intro-en-01 | "Thanks to [Brand] for sponsoring this video -- they sent me their new [product] and I've been testing it for 3 weeks." | Honest review format; product testing content |
| brand-intro-en-02 | "I've been using [Brand]'s [product] for months now, and they asked me to show you how I actually use it." | Routine/lifestyle integration; "day in the life" format |
| brand-intro-en-03 | "[Brand] challenged me to [task] using only their [product]. Here's what happened." | Challenge format; high engagement |

#### Chinese Opening Integration

| Template ID | Voiceover Text | Best Context |
|---|---|---|
| brand-intro-zh-01 | "感谢 [品牌] 赞助本期视频，他们寄来了新款 [产品]，我用了三周了。" | 真实测评/体验分享 |
| brand-intro-zh-02 | "我用了 [品牌] 的 [产品] 好几个月，他们让我给大家看看我的真实用法。" | 日常融入/生活方式 |
| brand-intro-zh-03 | "[品牌] 给我一个挑战：只用他们的 [产品] 完成 [任务]。来看看结果。" | 挑战/实验形式 |

#### Closing CTA (last 8s, soft brand + engagement)

| Template ID | CTA Text | Best Context |
|---|---|---|
| brand-cta-en-01 | "If you want to try [Brand]'s [product], I've linked it below. And let me know in the comments -- [engagement question]?" | Balanced: soft conversion + community |
| brand-cta-en-02 | "Check out [Brand] at the link in my bio. And follow for more honest reviews like this." | Conversion-lite + series building |
| brand-cta-en-03 | "Big thanks to [Brand] for making this video possible. What should I test next? 👇" | Gratitude-dominant; community-first |

#### Chinese Closing CTA

| Template ID | CTA Text | Best Context |
|---|---|---|
| brand-cta-zh-01 | "想试试 [品牌] 的 [产品]，链接放下面了。评论区告诉我 -- [互动问题]？" | 软转化 + 社区互动 |
| brand-cta-zh-02 | "[品牌] 的链接在主页。关注我，下期继续真实测评！" | 轻转化 + 系列预告 |
| brand-cta-zh-03 | "感谢 [品牌] 支持这期内容。你们还想看我测什么？评论区告诉我👇" | 感恩为主 + 社区驱动 |

### Logo Appearance Rules

| Rule | Specification |
|---|---|
| **First appearance** | Not before 5s (audience needs to trust the content first). Not in hook (0-3s). |
| **Position** | Product body (label, engraving) -- NOT as standalone logo overlay/watermark. |
| **Duration** | Logo visible naturally within product shots. No dedicated "logo screen." |
| **Size** | ≤ 15% of frame area. Logo should be discovered, not announced. |
| **Frequency** | Appears naturally in product demo shots (2-4 times through video). Not in every scene. |
| **CTA proximity** | Logo should NOT be the final frame. The CTA should be the last visual element. |

### Video Rhythm Strategy

```
Timeline (for 90s video, proportion %):
 0-3s   (0-3%):  Hook -- content-driven, no brand mention
 3-18s  (3-20%): Value Show -- brand intro at ~8-12s (natural, not forced)
 18-40s (20-44%): Body -- main content; brand product integrated into the story
 40-65s (44-72%): Deep Demo -- detailed product usage; authentic experience sharing
 65-82s (72-91%): Climax/Twist -- results/reveal; genuine reaction
 82-90s (91-100%): CTA -- soft, brand acknowledgment + community ask
```

**Key rhythm rules**:
- Brand intro at ~8-12s: after the hook but before the deep body. This is the "ad break" transparency moment -- get it over with early so the rest feels authentic.
- Product usage should be **demonstrable and specific** (actual features, actual results), not generic ("it's great quality").
- `brand_constraint.must_say` phrases must appear in either the brand intro voiceover or the closing CTA.
- If `brand_constraint.logo_rules` exist, apply them to all visual descriptions where the product appears.
- The video should still work as content **without** the brand mention -- if it doesn't, the integration is too heavy.

### Typical Example

Topic: "My 10-minute morning skincare routine"
Niche: beauty | Audience: 25-34, US | Duration: 90s | Language: en
Brand: fictional "LumiGlow" serum

Rhythm: 0-3s (hook: alarm rings, tired face in mirror, text: "10 min") → 3-18s ("I've cut my morning routine to 10 minutes. Here's how." → at 10s: "Quick thanks to LumiGlow for sponsoring -- I've been using their vitamin C serum for 3 weeks and it's actually changed my routine.") → 18-40s (steps 1-3, LumiGlow appears in step 2) → 40-65s (steps 4-6, close-up of serum texture, application technique) → 65-82s (final result: before/after, genuine "wow, my skin actually looks brighter") → 82-90s (CTA: "LumiGlow link in bio if you want to try it. What's your non-negotiable skincare step? 👇")

---

## 4. Live Funnel Playbook (Livestream Traffic)

### Goal
Maximize **livestream entry rate** (viewers who tap the live notification/badge and enter the stream). The video is a trailer; the livestream is the main event.

### Content Tone
- **Teaser + anticipation**: "This is just the preview -- the full thing happens LIVE."
- **Exclusivity**: "This won't be posted as a regular video. Only in the live."
- **Urgency with FOMO (Fear Of Missing Out)**: time-bound, quantity-bound, access-bound.
- High energy but not desperate -- confident that the live is worth attending.

### CTA Type
`live_cta`

The CTA must combine **what happens in the live** + **when it happens** + **why the viewer should care**.

### CTA Copy Templates

#### English (3 templates)

| Template ID | CTA Text | Technique | Timing |
|---|---|---|---|
| live-en-01 | "I'm going LIVE tonight at 8PM EST -- full tutorial, Q&A, and a giveaway you don't want to miss. Tap to set a reminder! 🔔" | Benefit-stack (tutorial + Q&A + giveaway) + time anchor | Last 10s |
| live-en-02 | "The full unboxing is happening LIVE tomorrow -- and there's a surprise product I'm NOT showing here. Link in bio to get notified." | Exclusivity bait (surprise product only in live) | Last 8s |
| live-en-03 | "Live in 30 minutes! Drop your questions below and I'll answer them on stream 👇" | Interactive bridge (comments feed into live content) | Last 8s |

#### Chinese (3 templates)

| Template ID | CTA Text | Technique | Timing |
|---|---|---|---|
| live-zh-01 | "今晚 8 点直播！完整教程 + 现场答疑 + 抽奖福利。点我主页预约！🔔" | 福利堆叠 + 时间锚点 | 倒数 10s |
| live-zh-02 | "完整开箱明天直播间见——还有个隐藏款这里不剧透。主页链接预约！" | 独家内容悬念 | 倒数 8s |
| live-zh-03 | "半小时后开播！有问题评论区先留言，直播间统一回答👇" | 评论→直播互动桥 | 倒数 8s |

### Three Live Traffic Techniques

#### Technique 1: Suspense/Curiosity (悬念式)

**Pattern**: "I discovered something amazing/terrifying, but I can't show it in a short video. Come to the live."

**Hook design**: Curiosity-question that the short video only partially answers.
**Body content**: 70% of the information -- enough to prove value, not enough to satisfy.
**CTA**: "The remaining 30% -- the part that changes everything -- I'm revealing LIVE tonight."

**Best for**: edu niche, tech niche, investigation-style content, product comparisons with dramatic results.

#### Technique 2: Benefit/Freebie (福利式)

**Pattern**: "I'm giving away [thing] / doing [service] for free -- only for live viewers."

**Hook design**: Visual-shock showing the giveaway item or service result.
**Body content**: Showcase the value, testimonials, "this could be yours."
**CTA**: "This [item/service] is ONLY available during the live. No replay. Be there."

**Best for**: shop niche (product giveaways), beauty niche (makeover/consultation), fitness niche (personalized coaching).

#### Technique 3: Countdown/Deadline (倒计时式)

**Pattern**: "In [time], [event] happens. You need to be there for [reason]."

**Hook design**: Number-promise with time anchor ("In 2 hours, this price is gone.").
**Body content**: Build urgency layer by layer -- first the product, then the deal, then the deadline.
**CTA**: "Timer is running. Link in bio to join the live before it's too late."

**Best for**: shop niche (flash sales), brand_deal (exclusive launch), limited-edition drops.

### Video Rhythm Strategy

```
Timeline (for 60s live funnel video, proportion %):
 0-3s   (0-5%):   Hook -- tease the big reveal/result (curiosity-question is universal)
 3-12s  (5-20%):  Value Preview -- show a TASTE of what the live contains
 12-27s (20-45%): Body -- deliver genuine value (the short video must still be good)
 27-42s (45-70%): Anticipation Build -- hint at what's EXCLUDED from this video
 42-55s (70-92%): FOMO Peak -- "This is only the beginning..."
 55-60s (92-100%): CTA -- live time, platform, action, reason
```

**Key rhythm rules**:
- The video must be valuable on its own (if the live doesn't happen, viewers shouldn't feel scammed).
- Never give away more than 70% of the full content -- leave a clear "need to see more" gap.
- Time anchor must be specific: "8PM EST tonight" not "later today."
- Platform-specific: TikTok Live notification requires the viewer to follow the creator. If awareness is the primary goal, include a "follow to get notified" sub-CTA.

### Typical Example

Topic: "Testing 5 viral kitchen gadgets -- which actually work?"
Niche: shop | Audience: 18-24, US | Duration: 60s | Language: en

Rhythm: 0-3s (hook: gadget #1 fails spectacularly, egg everywhere) → 3-12s ("I bought the 5 most viral kitchen gadgets on TikTok to test them. Here are the first 3...") → 12-27s (gadgets #1-3 results: 1 fail, 2 works) → 27-42s (gadget #4 quick demo, solid results) → 42-55s ("Gadget #5... you won't believe this one. I'm saving the full test for my LIVE tonight. Plus I'm giving away all 5 gadgets to one live viewer.") → 55-60s (CTA: "8PM EST. Link in bio to get notified. Be there!")

---

## 5. Awareness Playbook (Follower Growth / Brand Building)

### Goal
Maximize **follow rate** and **save/bookmark rate**. The video should make viewers think "I need more content from this creator."

### Content Tone
- **Value-first authority**: the content is so useful or entertaining that following is a no-brainer.
- **Series mindset**: each video is part of an ongoing value stream. Following = subscribing to the series.
- **Clean, confident, structured**: no begging for follows. The content earns it.
- "If you liked this, you'll love what's coming" energy.

### CTA Type
`follow_cta`

The CTA is a natural invitation to continue the journey, anchored to a specific future value promise.

### CTA Copy Templates

#### English (3 templates)

| Template ID | CTA Text | Technique | Timing |
|---|---|---|---|
| aware-en-01 | "Follow for more [niche] tips like this -- part 2 drops Thursday 👀" | Series teaser + value preview | Last 6s |
| aware-en-02 | "Save this video for later -- and follow because I post [content type] every week." | Save + follow combo; rhythm promise | Last 6s |
| aware-en-03 | "This is part 1 of my [topic] series. Follow so you don't miss the rest. 🔖" | Content series + FOMO | Last 8s |

#### Chinese (3 templates)

| Template ID | CTA Text | Technique | Timing |
|---|---|---|---|
| aware-zh-01 | "关注我，更多 [品类] 干货每周更新。下期讲 [下期主题]！" | 系列预告 + 价值承诺 | 倒数 6s |
| aware-zh-02 | "先收藏，再关注——我每周分享 [内容类型]。" | 收藏 + 关注组合；频率承诺 | 倒数 6s |
| aware-zh-03 | "这是 [主题] 系列的第一期，关注不错过后续更新🔖" | 内容系列 + 错失恐惧 | 倒数 8s |

### Series Teaser Templates

For awareness playbooks, the hook or the CTA should include a **series teaser** that gives the viewer a concrete reason to follow. The teaser must name the NEXT topic specifically.

| Pattern | Example |
|---|---|
| "This is how I [achieved X]. Next video: how I [achieved Y]." | "This is how I hit 10K followers in 30 days. Next video: how I monetized at 5K." |
| "Part 1: [topic A]. Part 2: [topic B]. Follow for part 2." | "Part 1: the products I use. Part 2: the routine that makes them work. Follow for part 2." |
| "I've tested [X number] [items]. Here's #1. Follow for #2-#5 this week." | "I've tested 10 moisturizers. Here's my #1. Follow for #2-#10 all week." |

### Video Rhythm Strategy

```
Timeline (for 90s video, proportion %):
 0-3s   (0-3%):   Hook -- number-promise or visual-shock establishing authority
 3-18s  (3-20%):  Value Show -- deliver first piece of value immediately (prove you're worthy)
 18-40s (20-44%): Body -- structured, dense value delivery (list format works well)
 40-65s (44-72%): Deep Demo -- the best/most surprising point; the "aha" moment
 65-82s (72-91%): Value Recap / Bridge -- summarize what they learned; bridge to what's coming
 82-90s (91-100%): CTA -- follow + series teaser
```

**Key rhythm rules**:
- First value point must land within 15s -- awareness audiences judge credibility fast.
- Use structured formats (numbered lists, step-by-step, tier lists) -- they perform better for follow conversion because viewers mentally bookmark "this creator is organized and useful."
- The "Bridge" segment (65-82s) is unique to awareness: explicitly connect THIS video's value to FUTURE content. "Now that you know X, next time I'll show you Y which is even more important."
- Avoid follower-begging language ("please follow me", "I need followers") -- it signals low authority.

### Typical Example

Topic: "3 editing tricks that make your TikToks look professional"
Niche: edu | Audience: 18-24, US | Duration: 90s | Language: en

Rhythm: 0-3s (hook: split screen -- amateur edit vs. pro edit of same footage, text: "3 tricks") → 3-18s ("These 3 editing tricks took my videos from 200 views to 200K. Trick #1: beat matching...") → 18-40s (trick #1 demo + trick #2 setup) → 40-65s (trick #2 demo + trick #3 -- the most impactful one) → 65-82s ("These 3 tricks work on CapCut, which is free. Now you know HOW to structure your edits. Next video: I'm breaking down the 4 hook patterns that actually stop the scroll.") → 82-90s (CTA: "Follow for part 2 on hooks. Save this so you don't forget trick #3.")

---

## 6. CTA Timing by Video Duration

CTA timing scales with video duration. The CTA must always be in the **final segment** and never interrupt the content flow.

| Duration (sec) | CTA Timing Window | Segment Type |
|---|---|---|
| 60 | Last 5-8s (52-60s) | Final segment |
| 75 | Last 7-10s (65-75s) | Final segment |
| 90 | Last 8-10s (80-90s) | Final segment |
| 120 | Last 8-12s (108-120s) | Final segment |
| 180 | Last 10-15s (165-180s) | Final segment |

**Universal rule**: CTA must be in the **last seedance_segment** (the segment with the latest end_sec). Never place a CTA mid-video (except for CRP mid-roll engagement prompts, which are not CTAs -- they are retention hooks).

---

## 7. Playbook Combination Rules

Some monetization_goal scenarios have secondary effects. Apply these combination rules:

| Primary Goal | Secondary Effect | Combination Rule |
|---|---|---|
| `shop` | Awareness (new product launch) | Use shop CTA but add a "follow for more deals" sub-CTA. Product still dominates. |
| `brand_deal` | Shop (brand wants conversion) | If `brand_constraint` includes a link/promo code, CTA shifts to soft-conversion. Logos follow shop rules (appear faster). |
| `live_funnel` | Awareness (new creator building audience) | Add "follow to get notified" to the live CTA. Series teaser in hook. |
| `awareness` | CRP (follower growth drives RPM) | Use awareness CTA + add a mid-roll "did you know?" engagement prompt at 45-55%. |
| `crp` | Awareness (follower growth from CRP content) | Keep CRP CTA but use awareness series teaser in the bridge/closing segment. |

---

## 8. Language Unification Rule (AC3)

All CTA copy, voiceover text, on_screen_text, title, and hashtag rationale MUST be in the language specified by the `language` parameter. If `language=en`, use English templates. If `language=zh`, use Chinese templates. For other languages, translate the English templates and adapt culturally -- do NOT output mixed-language copy.

Exception: `music_suggestion.tiktok_library_keywords` is always in English (TikTok sound library search language).

---

## 9. brand_constraint Integration

| brand_constraint field | Playbook behavior |
|---|---|
| `must_say` (string[]) | Each phrase must appear in at least one scene's `voiceover` or the `cta.text`. For `brand_deal`, place in brand intro voiceover (first occurrence) + closing CTA (reinforcement). |
| `banned` (string[]) | Merge with built-in banned word blacklist. Apply in pre-check (Step 6). Words here override playbook template defaults if conflict exists. |
| `logo_rules` (string, optional) | Apply to `visual_description` of all scenes where the brand product appears. For `brand_deal`, follow §3 logo appearance rules. For `shop`, logo may appear earlier (after 5s). |

---

## 10. Execution Checklist (for Step 4)

When Step 4 executes, the LLM should verify:

- [ ] Correct playbook loaded for `monetization_goal`.
- [ ] CTA type matches the playbook's assigned type.
- [ ] CTA copy uses a template from this file (EN or ZH based on `language`).
- [ ] CTA timing falls within the correct window per video duration (last segment).
- [ ] Voiceover generation follows DAG dependency: hook (Step 2) → voiceover (per scene) → on_screen_text (per scene).
- [ ] `brand_constraint.must_say` phrases appear in voiceover or CTA.
- [ ] All copy fields are in the same language (AC3).
- [ ] `trending_inject.hashtags` values appear in `hashtags.trending` (AC11).
- [ ] `trending_inject.sounds` values appear in `music_suggestion.tiktok_library_keywords` (AC11).
- [ ] For `shop` playbook: no absolute price/efficacy claims in CTA or voiceover.
- [ ] For `brand_deal`: brand intro at 8-12s, not in hook (0-3s).
- [ ] For `live_funnel`: CTA includes specific time anchor and platform action.
- [ ] For `awareness`: CTA includes series teaser with next topic name.
