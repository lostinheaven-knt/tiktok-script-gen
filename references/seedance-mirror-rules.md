# Seedance Mirror Rules — tiktok-script-gen v1

> Mirror of video_prompt_gen's anti-pattern self-check rules R1, R3, R4, R6.
> Applied in Step 5 of the reasoning workflow when generating `seedance_segments[].prompt_draft`.
> These rules ensure prompt_draft outputs pass video_prompt_gen's self-check on first submission,
> so the user can use "fast track" mode without triggering downstream rewrites.
>
> **sync_source**: `~/.claude/skills/video_prompt_gen/references/principles.md` R1, R3, R4, R6
> **version**: v1
> **last_synced**: 2026-06-02

---

## Why Mirror Rules?

video_prompt_gen applies R1/R3/R4/R6 self-checks on all generated prompts and auto-rewrites
once on failure. If tiktok-script-gen produces a prompt_draft that violates these rules, the downstream
user experience degrades: video_prompt_gen will rewrite the carefully composed prompt, potentially
losing TikTok-specific context.

By applying the same rules before output, tiktok-script-gen ensures:
1. prompt_draft passes video_prompt_gen's self-check on arrival.
2. User can use "直接给" / "fast track" without modification.
3. If the auto-rewrite still fails here, `requires_manual_review = true` gives the user a heads-up.

---

## R1: 14-Word Abstract Adjective Blacklist

### Rule

The prompt_draft MUST NOT contain any of the following 14 abstract adjectives.
These words are banned because they are non-actionable for AI video models
(instead of "震撼画面", describe the actual visual: "红色口红从画面右侧快速推入").

### Blacklist

| # | Chinese | English Equivalent | Why Banned |
|---|---|---|---|
| 1 | 史诗 | epic | Non-visual — what specifically makes it "epic"? |
| 2 | 震撼 | shocking/stunning | Subjective — model cannot render "shock" |
| 3 | 绝美 | breathtakingly beautiful | Subjective — describe specific beauty elements |
| 4 | 电影感 | cinematic | Vague — specify aspect ratio, lighting, color grade |
| 5 | 大师之作 | masterpiece | Subjective — cannot be rendered |
| 6 | 完美 | perfect | Absolute — describe what makes it good |
| 7 | 高质量 | high quality | Redundant — model always tries for quality |
| 8 | 超高清 | ultra HD / 8K | Resolution is model-dependent, not prompt-controlled |
| 9 | 8K | 8K | Same as above — resolution parameter |
| 10 | 极致 | ultimate / extreme | Subjective superlative |
| 11 | 惊艳 | amazing / stunning | Subjective — describe what creates amazement |
| 12 | 史无前例 | unprecedented | Unverifiable claim |
| 13 | 无与伦比 | unparalleled | Unverifiable claim |
| 14 | 终极 | ultimate | Subjective superlative |

### Edge Cases

- Partial matches: "完美" should NOT match "完美主义" (perfectionism). Use whole-word matching for Chinese.
- English variants: "epic" should match "epicness" / "epically". Use substring matching for English.
- Mixed language: if a prompt contains both Chinese and English, check against both blacklists.

### Safe Alternatives

| Instead of... | Use... |
|---|---|
| "震撼的画面" | "高对比度画面，前景物体快速推入镜头" |
| "绝美的风景" | "暖色调日落，金色光线穿透云层，远处山脉轮廓清晰" |
| "电影感" | "21:9 宽银幕比例，柔光滤镜，暖色调调色" |
| "完美构图" | "三分法构图，主体位于画面右下交叉点" |
| "超高清画质" | "锐利对焦，纹理清晰可见" |
| "极致光影" | "左上方 45 度柔光，主体边缘有轻微轮廓光" |

---

## R3: Single Action Constraint

### Rule

Each camera shot (镜头) in `prompt_draft` MUST describe exactly **one main action**.
If the prompt describes a multi-shot segment (max 3 shots), each "镜头N：" block must have one action.

### Violations (examples)

| Violation | Why |
|---|---|
| "模特奔跑然后跳跃然后旋转" | 3 actions (跑 + 跳 + 转) in one shot |
| "产品从左侧滑入同时旋转并弹跳" | 3 actions (滑 + 旋转 + 弹跳) in one shot |
| "口红涂抹后晕开再叠加" | 2 sequential actions (涂抹 + 晕开 + 叠加) |

### Corrections

| Violation | Correction |
|---|---|
| "模特奔跑然后跳跃" | Split into 2 shots: "镜头1：模特从画面左侧向右奔跑" / "镜头2：模特原地向上跳跃，双臂展开" |
| "产品旋转并弹跳" | "产品绕自身垂直轴匀速旋转，约 5 秒一圈" (pick ONE action: rotation) |
| Multi-step process | Split across scenes: scene 2 = 涂抹, scene 3 = 叠加 |

### Exceptions

- **Natural consequence**: "液体倒入杯中，杯中液面上升" — pouring causes rising, one causal chain.
- **Simultaneous sub-actions**: "模特微笑并点头" — facial expression + head movement treated as one composite action.

---

## R4: Single Camera Movement Constraint

### Rule

Each camera shot (镜头) in `prompt_draft` MUST describe exactly **one camera movement**.
Static/fixed camera is acceptable (described as "镜头静止" or "固定镜头").

### Violations (examples)

| Violation | Why |
|---|---|
| "镜头推近同时向左平移" | 2 moves (dolly-in + pan-left) |
| "镜头从下往上摇同时拉远" | 2 moves (tilt-up + dolly-out) |
| "快速摇镜并推近到产品特写" | 2 moves (pan + dolly-in) |

### Allowed Camera Movements (choose ONE)

| Type | Chinese Description | English |
|---|---|---|
| Static | 镜头静止 / 固定镜头 | static / locked-off |
| Dolly in | 推近 / dolly-in | dolly in / push in |
| Dolly out | 拉远 / dolly-out | dolly out / pull back |
| Pan left/right | 向左/右平移 | pan left/right |
| Tilt up/down | 向上/下摇镜 | tilt up/down |
| Orbit | 环绕运镜 | orbit shot |
| Tracking | 跟拍 | tracking shot |
| Zoom | 变焦推拉 | zoom in/out |

### Multi-Shot Segments

If `prompt_draft` contains multiple shots ( 镜头1 / 镜头2 / 镜头3 ),
each shot may have its own camera movement (one per shot).

---

## R6: Length and Format Constraint

### Rule 6a: Word Count

`prompt_draft` MUST be **50-150 Chinese characters** (or 50-150 words for English-targeting models).
This maps to Seedance 2.0's optimal prompt length (approximately one Weibo post worth of text).

- Under 50 chars: Too vague, model lacks sufficient direction.
- Over 150 chars: Too detailed, model may ignore later instructions or introduce artifacts.

### Rule 6b: No Negative Prompts

`prompt_draft` MUST NOT contain negative prompt syntax (Seedance 2.0 does not support negative prompts).
Instead of describing what NOT to do, describe what you WANT in positive terms.

| Instead of... | Use... |
|---|---|
| "不要模糊" | "锐利对焦，纹理清晰" |
| "不要抖动" | "画面平滑稳定" |
| "没有杂乱背景" | "背景干净整洁，纯色墙面" |
| "不要太暗" | "充足柔光，画面明亮" |
| "不要快速切换" | "缓慢平稳过渡" |

### Rule 6c: Continuity Constraint

For all non-first segments (`segment_id > 1`), append this sentence to the end of `prompt_draft`:

```
保持与上一段画面的光照、色调、主体位置连续性，确保拼接后无明显跳变。
```

This sentence does NOT count toward the 50-150 character limit.

### Rule 6d: Multi-Shot Limit

A single `prompt_draft` may contain at most 3 shots (镜头1 / 镜头2 / 镜头3).
If a segment spans more than 3 scenes, split it into multiple segments.

---

## Auto-Rewrite Strategy

When any of R1/R3/R4/R6 is violated, apply the following flow:

```
1. DETECT: Identify which rule(s) are violated and the specific violating text span.
2. REWRITE ONCE: Modify prompt_draft to resolve the violation:
   - R1: Replace abstract adjective with specific visual description.
   - R3: Split multi-action into separate shots (or pick one action).
   - R4: Reduce to single camera movement per shot.
   - R6a: Trim or expand to 50-150 characters.
   - R6b: Replace negative phrasing with positive equivalent.
3. RE-CHECK: Run the modified prompt_draft through R1/R3/R4/R6 again.
4. DECIDE:
   - PASS → output the rewritten prompt_draft, no flag.
   - STILL FAILING → keep the best version, set requires_manual_review = true.
```

### Rewrite Priority

If multiple rules are violated simultaneously, fix in order: R6b > R1 > R3 > R4 > R6a.
Rationale: negative prompts and abstract adjectives are the most harmful to generation quality.

---

## requires_manual_review Flag Rules

Set `seedance_segments[i].requires_manual_review = true` when:

1. The prompt_draft still violates R1/R3/R4/R6 after 1 auto-rewrite attempt.
2. The prompt_draft contains a `@引用` syntax (image/video/audio reference) that cannot be auto-validated.
3. The segment's `continuity_hint` contains a warning about potential visual mismatch (e.g., "光照变化较大，建议人工检查").
4. The segment spans a duration boundary where a scene's voiceover sentence was forcibly split (AC16 edge case).

Set `requires_manual_review = false` in all other cases (default).

---

## Integration with video_prompt_gen

When the user feeds a `prompt_draft` that passed these mirror rules into video_prompt_gen:

1. video_prompt_gen will run its own R1-R8 self-check.
2. Since our mirror rules are aligned, the prompt_draft should pass R1/R3/R4/R6 without triggering rewrites.
3. video_prompt_gen may still apply R2/R5/R7/R8 advisories (non-rewriting rules) — these are acceptable.
4. If video_prompt_gen's rules have been updated (e.g., new words added to R1 blacklist) since our last sync,
   a drift may occur. The CI `drift_check` in template-registry.md partially covers this; a full sync requires
   manual comparison of `seedance-mirror-rules.md` vs `~/.claude/skills/video_prompt_gen/references/principles.md`.

### Drift Risk Mitigation

- This file stores a local copy of the 14-word blacklist. If video_prompt_gen adds words, CI should flag.
- The R3/R4/R6 constraints are structural (not list-based) and less likely to drift.
- Recommended: quarterly manual sync of this file against video_prompt_gen's principles.md.
