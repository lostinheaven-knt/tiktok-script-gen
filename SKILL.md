---
name: tiktok-script-gen
description: "Use this skill when the user wants to generate a high-RPM TikTok video script (60-180s, CRP-eligible) with hook design, scene breakdown, CTA strategy, hashtags, cover, music suggestions, compliance checks, and pre-filled Seedance 2.0 prompt drafts. The skill orchestrates monetization-aware scripts for CRP / TikTok Shop / Brand Deal / Live Funnel / Awareness goals and outputs both Markdown and JSON. Invoke whenever user expresses intent to create a TikTok video for revenue or growth. Common triggers in Chinese: TikTok 脚本 / 抖音脚本 / 短视频脚本 / 高 RPM / 完播率 / 带货脚本 / TikTok 变现 / TikTok Shop / CRP 收益. English: tiktok script / tiktok video / short-form script / high rpm / tiktok shop / crp / creator rewards / hook script."
slash_commands:
  - name: tiktok-script
    description: Generate a high-RPM TikTok video script with full scene breakdown, Seedance 2.0 prompts, and compliance checks.
    args:
      - name: topic
        description: Video topic or one-line theme (e.g., "lipstick review for pale skin")
        required: true
      - name: niche
        description: Content vertical — beauty / tech / edu / food / travel / fitness / finance / comedy / shop / other
        required: true
      - name: target_audience
        description: Audience profile as JSON — {age, region, interests, gender?} (e.g., {"age":"18-24","region":"US","interests":["kbeauty"]})
        required: true
      - name: monetization_goal
        description: Monetization path — crp / shop / brand_deal / live_funnel / awareness
        required: true
      - name: duration_sec
        description: Target video length in seconds (default 60; <60 triggers CRP ineligibility warning)
        required: false
      - name: language
        description: Output language (default en) — en / zh / es / pt / ja / ko / ...
        required: false
      - name: ab_variants
        description: Number of hook variants (default 2, max 5)
        required: false
      - name: output_format
        description: Output format (default both) — markdown / json / both
        required: false
      - name: brand_constraint
        description: Brand rules as JSON — {must_say:[], banned:[], logo_rules?}
        required: false
      - name: platform_region
        description: Platform region (default US) — US / SEA / CN-overseas / global
        required: false
      - name: style_preference
        description: Visual/narrative style — fast-cut / story / asmr / talking-head / faceless
        required: false
      - name: reference_creators
        description: Reference creator handles for style anchoring (not copying) — e.g., ["@creator1","@creator2"]
        required: false
      - name: trending_inject
        description: Manually injected trending data as JSON — {hashtags:[], sounds:[]}
        required: false
---

# tiktok-script-gen

> TikTok 高 RPM 脚本生成 Skill — 上游编排层，产出完整 TikTok 视频脚本（分镜 / 文案 / CTA / 标签 / 合规检查 / Seedance 2.0 prompt 草稿），下游由 video_prompt_gen 或直接由 Seedance 2.0 渲染成片。

---

## 1. When to Activate / When to Exit

### 1.1 Activate

This skill activates when the user message contains **any** of the following triggers (matched semantically by Claude Code against the skill `description` field plus the explicit lists below):

**Chinese triggers**:
TikTok 脚本 / 抖音脚本 / 短视频脚本 / 高 RPM / 完播率 / 带货脚本 / TikTok 变现 / TikTok Shop / CRP 收益 / TikTok 视频脚本 / 帮我写一条 TikTok / 生成 TikTok 脚本 / 抖音带货 / 短视频带货 / TikTok 高收益

**English triggers**:
tiktok script / tiktok video / high rpm / tiktok shop / crp / creator rewards / hook script / short-form script / tiktok monetization / tiktok ad script / tiktok content / viral tiktok script

**Slash command**:
`/tiktok-script` — explicitly invoke with structured parameters.

Activation also occurs when the user expresses a TikTok video creation need without naming a model (e.g., "make me a TikTok about skincare products that earns well").

### 1.2 Exit (Mis-Trigger)

**Exit immediately** when the user's actual intent is **not** TikTok script generation, but rather:

- **Video editing / post-production** (剪辑 / 裁剪 / 加字幕 / 转场 / 调色)
- **Account operations** (账号运营 / 涨粉策略 / 发布时间 / 数据分析)
- **Ad platform backend** (广告投放后台 / TikTok Ads Manager / 竞价策略)
- **Live streaming setup** (直播间搭建 / 直播话术 / 直播带货现场)
- **General social media management** not specific to TikTok script creation

**Exit response template (Chinese)**:
> 看起来你的需求可能是 [推断意图]，而不是生成一条 TikTok 视频脚本。如果是这样，我先让出。如果你确实想要一条 TikTok 脚本，请告诉我你的选题、品类、受众和变现目标，我马上开始。

**Exit response template (English)**:
> It seems your need may be [inferred intent], not generating a TikTok video script. If so, I'll step aside. If you do want a TikTok script, tell me your topic, niche, target audience, and monetization goal and I'll get started.

---

## 2. First Step: Load References

Always read in order when this skill activates:

1. `references/template-registry.md` — template ID domain (18 templates: 13 reused + 5 new)
2. `references/compliance-rules.md` — banned word blacklist + watermark detection + music copyright rules
3. `references/seedance-mirror-rules.md` — R1/R3/R4/R6 mirror rules from video_prompt_gen
4. `references/kpi-table.md` — completion rate & RPM estimation tables

Conditional loads (on-demand):
- `references/hook-strategies.md` — load when entering Step 2 (hook selection)
- `references/monetization-playbooks.md` — load when entering Step 4 (CTA generation)
- `references/templates/tiktok-*.md` — load when a scene references a specific template_id
- `references/output-schema.json` — load in Step 8 when outputting JSON for schema validation

> **Note (Cycle 5 — FINAL)**: All reference files are available and all 8 reasoning steps (Steps 1-8) are fully implemented with complete reasoning flow. This skill is at v1 production readiness. All output sections (§5, §6, §7, §8, §9) are fully populated.

---

## 3. Input Parameters

### 3.1 Parameter Table (13 parameters)

| # | Parameter | Type | Required | Default | Description | Example |
|---|---|---|---|---|---|---|
| 1 | `topic` | string | **Yes** | — | Video topic or one-line theme | `"lipstick review for pale skin"` |
| 2 | `niche` | enum | **Yes** | — | Content vertical. Valid values: `beauty` / `tech` / `edu` / `food` / `travel` / `fitness` / `finance` / `comedy` / `shop` / `other` | `beauty` |
| 3 | `target_audience` | object | **Yes** | — | Audience profile with `age` (string), `region` (string), `interests` (string[]), and optional `gender` | `{"age":"18-24","region":"US","interests":["kbeauty"]}` |
| 4 | `monetization_goal` | enum | **Yes** | — | Monetization path determining script style. Valid values: `crp` / `shop` / `brand_deal` / `live_funnel` / `awareness` | `crp` |
| 5 | `duration_sec` | int | No | `60` | Target video length in seconds. No lower bound, but <60s triggers CRP ineligibility warning (AC2) | `75` |
| 6 | `language` | enum | No | `en` | Output language for all copy fields (hook/voiceover/on_screen_text/cta/title/hashtags). Supported: `en` / `zh` / `es` / `pt` / `ja` / `ko` / `ar` / `fr` / `de` / `id` / `th` / `vi` / `it` / `tr` / `ru` / `hi` | `zh` |
| 7 | `brand_constraint` | object | No | `null` | Brand compliance rules: `must_say` (string[] — phrases that MUST appear), `banned` (string[] — forbidden words), `logo_rules` (string, optional) | `{"must_say":["new arrival"],"banned":["cheapest"]}` |
| 8 | `platform_region` | enum | No | `US` | Platform region affecting compliance word lists and music copyright. Valid: `US` / `SEA` / `CN-overseas` / `global` | `US` |
| 9 | `style_preference` | enum | No | `null` | Visual/narrative style preference. Valid: `fast-cut` / `story` / `asmr` / `talking-head` / `faceless` | `fast-cut` |
| 10 | `reference_creators` | string[] | No | `[]` | Reference creator handles for style anchoring (not copying). Used to infer tone/pace/visual style | `["@glowrecipe","@skincarebyhyram"]` |
| 11 | `ab_variants` | int | No | `2` | Number of hook variants to generate. **Max 5**; values >5 are truncated to 5 (AC5) | `3` |
| 12 | `trending_inject` | object | No | `null` | Manually injected trending data: `hashtags` (string[]) and `sounds` (string[]). Not auto-fetched from TikTok | `{"hashtags":["#summerglow"],"sounds":["upbeat pop summer 2026"]}` |
| 13 | `output_format` | enum | No | `both` | Output format. Valid: `markdown` / `json` / `both` | `both` |

### 3.2 Default Value Population

When a parameter is not provided by the user, apply the following defaults:

| Parameter | Default Value |
|---|---|
| `duration_sec` | `60` |
| `language` | `"en"` |
| `ab_variants` | `2` |
| `output_format` | `"both"` |
| `platform_region` | `"US"` |
| `brand_constraint` | `null` |
| `style_preference` | `null` |
| `reference_creators` | `[]` |
| `trending_inject` | `null` |

### 3.3 Boundary Handling Rules

These rules execute during **Step 1 (Parameter Parsing & Validation)** and affect downstream fields:

| Rule | Condition | Action |
|---|---|---|
| **Short duration warning** | `duration_sec < 60` | Do NOT reject the request. Set `meta.duration_meets_crp = false`. Append to `compliance_check.notes`: `"时长不足 60s（实际 {N}s），不计入 TikTok CRP（Creator Rewards Program）收益。如需 CRP 准入，请将 duration_sec 调整为 ≥60。"` (AC2) |
| **Marginal duration hint** | `60 <= duration_sec < 75` | Append to `compliance_check.notes`: `"时长 {N}s 满足 CRP 准入下限，但 75s+ 通常 RPM 区间更优，建议考虑加长。"` |
| **Long duration warning** | `duration_sec > 180` | Append to `compliance_check.notes`: `"时长 >180s 完播率显著下降，建议拆分为系列短视频。"` |
| **AB variant cap** | `ab_variants > 5` | Truncate to `5`. Add note: `"ab_variants 超过上限 5，已截断为 5。"` (AC5) |
| **Language fallback** | `language` not in supported enum | Fallback to `"en"`. Add note: `"language '{input}' 不在支持列表中，已回退为 en。"` |
| **Niche fallback** | `niche` not in 10-class enum | Fallback to `"other"`. Add note: `"niche '{input}' 未在经验表内，已回退为 other；KPI 估值使用 other 行数据。"` |
| **Missing required params** | `topic` / `niche` / `target_audience` / `monetization_goal` missing | Ask user to provide (max 1 round of Q&A). Do not proceed with defaults. |

---

## 4. Reasoning Workflow (8 Steps)

This section defines the 8-step pipeline. Steps marked with `TODO Cycle N` contain framework-level descriptions; full reasoning instructions will be implemented in the indicated cycle.

### Step 1: Parameter Parsing & Validation

**Status**: Full implementation (Cycle 1).

Execute the following in order:
1. **Parse** all input parameters from the user message or slash command args.
2. **Required-field check**: Verify `topic`, `niche`, `target_audience`, and `monetization_goal` are all present. If any is missing, ask the user once (max 1 Q&A round) to provide the missing value(s).
3. **Default value population**: Apply defaults from §3.2 for any optional parameter not provided.
4. **Boundary handling**: Apply the rules from §3.3 in order:
   - Check `duration_sec` thresholds and set `meta.duration_meets_crp` accordingly.
   - Truncate `ab_variants` if > 5.
   - Validate `language` enum; fallback to `en` if unrecognized.
   - Validate `niche` enum; fallback to `other` if unrecognized.
5. **Output intermediate state**:
   ```
   [Step 1: Parameter Parsing]
   - topic: {value}
   - niche: {value}
   - target_audience: {age}, {region}, {interests}
   - monetization_goal: {value}
   - duration_sec: {value} → duration_meets_crp: {true/false}
   - language: {value}
   - ab_variants: {value}
   - output_format: {value}
   - platform_region: {value}
   - brand_constraint: {present/null}
   - style_preference: {value/null}
   - reference_creators: [{...}]
   - trending_inject: {present/null}
   - warnings: [{...}]
   ```

### Step 2: Audience & Hook Strategy Selection

**Status**: Full implementation (Cycle 3).

**Instructions**:

1. **Load** `references/hook-strategies.md`. Read the full file to access: §1 (5 hook type definitions with retention_tactic mapping), §2 (decision table: niche × age × monetization_goal → primary + secondary hook), §3 (3-piece templates per hook type), §4 (variant generation strategy with 5 dimensions), §5 (niche-specific override rules), §6 (edge cases), and §7 (real-world examples for context).

2. **Lookup** the decision table in §2 to find the matching row:
   - Key: `{niche} × {target_audience.age} × {monetization_goal}`
   - If `target_audience.age` is unspecified or ambiguous, default to `18-24` (largest TikTok demographic).
   - Output: `primary_hook` (hook type to use for the main hook_3s) and `secondary_hook` (hook type for variant 1).
   - If the niche × age cell is empty (e.g., finance for 13-17), use the closest older age bracket's entry and add a note to `compliance_check.notes`.
   - Use the monetization_goal-specific sub-table (§2.1–§2.5) matching the user's `monetization_goal` parameter.

3. **Generate primary hook** (`hook_3s`):
   - Use the 3-piece template from §3 corresponding to the `primary_hook` type.
   - Produce: `visual` (1–2 sentence visual shot description), `voiceover` (≤8 words), `on_screen_text` (≤6 chars ZH / ≤5 words EN).
   - Assign `retention_tactic` = the hook type's mapped tactic (see §1 mapping table).
   - Apply niche-specific override rules from §5 if the niche is not `other`.

4. **Generate hook variants** (`hook_3s.variants[]`):
   - Count = `ab_variants` (default 2, capped at 5 per §3.3).
   - **Mandatory constraint**: Each variant MUST use a different `retention_tactic` from the 5-tactic pool. If `ab_variants=5`, all 5 tactics are used exactly once.
   - Variant allocation per §4:
     - **Variant 1** (if `ab_variants >= 2`): Use `secondary_hook` type from decision table (D1 dimension — hook type change). Its `retention_tactic` = the secondary type's mapped tactic.
     - **Variant 2** (if `ab_variants >= 3`): Use `primary_hook` type but different voiceover angle (D2 dimension). Pick a `retention_tactic` not yet used.
     - **Variant 3** (if `ab_variants >= 4`): Use `primary_hook` type but different visual composition (D3 dimension). Pick an unused tactic.
     - **Variant 4** (if `ab_variants = 5`): Use `primary_hook` type but different pacing/timing (D4 dimension). Use the last unused tactic.
   - Each variant outputs the same 3-piece structure: `{visual, voiceover, on_screen_text, retention_tactic}`.

5. **Populate** the `hook_3s` output field:
   - `hook_3s.visual`, `hook_3s.voiceover`, `hook_3s.on_screen_text` = primary hook values.
   - `hook_3s.retention_tactic` = primary hook's retention tactic.
   - `hook_3s.variants[]` = array of variant objects.

6. **Output intermediate state**:
   ```
   [Step 2: Hook Strategy]
   - niche: {niche}
   - audience_age: {age}
   - monetization_goal: {goal}
   - primary_hook_type: {type}
   - primary_retention_tactic: {tactic}
   - secondary_hook_type: {type}
   - variant_count: {n}
   - variants_tactics: [{tactic1}, {tactic2}, ...]
   - niche_override_applied: true/false
   ```

### Step 3: Scene Orchestration & Seedance 15s Segmentation

**Status**: Full implementation (Cycle 4).

This step has four sub-steps. Execute them in order; each sub-step's output is the input for the next.

---

#### 3a. Macro Rhythm Planning

**Instructions**:

1. Determine the narrative arc based on `duration_sec`. The macro rhythm table below defines time boundaries for each narrative phase. These are **guidelines**, not rigid — adjust boundaries slightly to align with voiceover sentence endings (see 3b).

2. **Duration-to-Rhythm Mapping Table** (inline — data volume is small and tightly coupled with reasoning flow):

| Duration | Hook (0-start) | Value Show | Body Expansion | Deep Demo | Climax / Twist | CTA |
|---|---|---|---|---|---|---|
| **60s** | 0-3s | 3-12s | 12-27s | 27-42s | 42-55s | 55-60s |
| **75s** | 0-3s | 3-15s | 15-35s | 35-55s | 55-68s | 68-75s |
| **90s** | 0-3s | 3-18s | 18-40s | 40-65s | 65-82s | 82-90s |
| **120s** | 0-3s | 3-20s | 20-50s | 50-80s | 80-110s | 110-120s |
| **180s** | 0-3s | 3-25s | 25-65s | 65-110s | 110-170s | 170-180s |

3. **Interpolation rule**: If `duration_sec` is not in the table, scale the nearest tier linearly. Example: 68s → scale the 75s tier proportionally (each phase boundary × 68/75). Keep the hook at 0-3s unchanged.

4. **Niche adjustment** — apply these overrides to the macro rhythm:
   - `edu` / `finance`: Extend "Body Expansion" phase (+10%), shorten "Climax/Twist" (-10%) — information density requires more exposition time.
   - `comedy`: Shorten "Deep Demo" (-20%), extend "Climax/Twist" (+20%) — comedic payoff needs buildup.
   - `shop`: Shorten "Climax/Twist" (-15%), extend "CTA" (+15%) — conversion needs more CTA breathing room.
   - `beauty` / `food`: Standard rhythm, no adjustment.

5. **Output** the macro rhythm time anchors as input for 3b — e.g., for 60s:
   ```
   Hook: 0-3s | Value Show: 3-12s | Body: 12-27s | Deep Demo: 27-42s | Climax: 42-55s | CTA: 55-60s
   ```

---

#### 3b. Seedance 15s Segmentation Algorithm

**Core constraint**: Seedance 2.0 single-generation limit is **15 seconds**. Every `seedance_segment` must have `duration_sec ≤ 15`.

**Segmentation algorithm** (reasoning steps):

**3b.1. Coarse slice**: Slide a ≤15s window across the macro rhythm timeline to produce candidate cut points at integer-second boundaries.

**3b.2. Refinement priorities** (from highest to lowest):

- **Priority 1 — Semantic completeness (ALWAYS RESPECT)**:
  - NEVER cut in the middle of a voiceover sentence. Check the candidate cut point against the voiceover text for the scenes around it. If the cut point falls mid-sentence, shift it to the nearest complete sentence boundary (prefer moving earlier to keep the segment ≤15s).
  - For segments without voiceover (e.g., pure visual hook), this rule does not apply.

- **Priority 2 — Shot continuity (ALWAYS RESPECT)**:
  - NEVER cut during an active camera movement (推/拉/摇/移/跟/环绕). If the candidate cut point falls during a dolly-in, pan, orbit, or tracking shot, shift the boundary to after the movement completes.
  - Static/fixed camera scenes have no continuity restriction — cut anywhere.

- **Priority 3 — Beat alignment (NICE TO HAVE)**:
  - If `music_suggestion.bpm_range` is known, check whether the candidate cut point aligns with a beat boundary (integer multiple of `60 / bpm`). Prefer beat-aligned cut points, but NEVER at the expense of Priority 1 or 2.

**3b.3. Segment boundary hard constraints** (ALL must be satisfied):

| # | Constraint | Description |
|---|---|---|
| 1 | **Voiceover boundary** | Segment boundaries MUST align with complete voiceover sentence endings. Never cut mid-sentence. |
| 2 | **Camera move boundary** | Segment boundaries MUST NOT fall during an active camera movement (push/pull/pan/tilt/track/orbit). Cut after the move completes. |
| 3 | **CTA isolation** | The CTA occupies the last 1-2 segments exclusively. Do not mix CTA content with body/demo content in the same segment. |
| 4 | **Hook isolation** | The hook (0-3s) occupies segment[0] exclusively. If the hook is only 3s, segment[0] is 3s — remaining time up to 15s is NOT padded; it transitions naturally to segment[1]. |
| 5 | **Twist isolation** | If a `tiktok-twist-ending` template is used, it occupies the second-to-last segment (before the CTA segment). CTA follows immediately after twist. |
| 6 | **Scene count per segment** | Each segment contains **1-3 scenes**. If a macro phase has >3 scenes, split it across multiple segments. |
| 7 | **Duration balance** | Segments should be roughly balanced in duration (except first and last). Avoid extreme splits like 14s+1s. Minimum segment duration: 2s. |

**3b.4. Segmentation example** (75s video, beauty niche, crp goal):

```
Segment 0: 0-3s   (duration=3s,  scene_ids=[1],        template=tiktok-hook-3s)
Segment 1: 3-15s  (duration=12s, scene_ids=[2,3],      template=tiktok-product-on-face)
Segment 2: 15-30s (duration=15s, scene_ids=[4,5],      template=cosmetic-texture)
Segment 3: 30-45s (duration=15s, scene_ids=[6,7,8],    template=tiktok-beat-sync)
Segment 4: 45-60s (duration=15s, scene_ids=[9,10],     template=tiktok-before-after)
Segment 5: 60-68s (duration=8s,  scene_ids=[11],       template=tiktok-twist-ending)
Segment 6: 68-75s (duration=7s,  scene_ids=[12],       template=talking-head, CTA)
```

**3b.5. Post-segmentation validation** (run these checks after segmentation):

1. **Duration sum check**: `sum(seedance_segments[*].duration_sec) == meta.duration_sec` (tolerance ±1s due to integer rounding).
2. **Max duration check**: Every `seedance_segments[i].duration_sec ≤ 15`.
3. **Segment[0] check**: `seedance_segments[0].start_sec == 0` AND `seedance_segments[0].template_id == "tiktok-hook-3s"`.
4. **Gap check**: For all adjacent segments i and i+1: `segments[i].end_sec == segments[i+1].start_sec` (no gaps, no overlaps).
5. **Scene count check**: Each segment's `scene_ids` array length is ≤ 3.

**3b.6. Edge cases**:

- **duration_sec ≤ 15**: Only one seedance_segment (no segmentation needed). `continuity_hint` is `"opening"`. No CTA isolation possible — embed the CTA softly within the single segment.
- **duration_sec > 180**: Normal segmentation applies. Add a note to `compliance_check.notes`: `"本脚本共 N 段，需要 N 次 Seedance 生成 + 拼接。"` Set `meta.total_seedance_segments = N` for user visibility (R13).
- **Voiceover is a single continuous monologue**: If no natural sentence boundaries exist in a long voiceover passage, insert a brief pause sentence ("让我们看看..." 、"接下来...") at the segment boundary as a bridge cue.

---

#### 3c. Template Assignment

**Instructions**:

1. **scene[0] (start_sec=0) MUST use `tiktok-hook-3s`** (AC4). No exceptions — regardless of niche or monetization_goal. The corresponding `seedance_segments[0].template_id` is also `tiktok-hook-3s`.

2. **For each subsequent scene**, select a `template_id` from the 18-template registry based on niche and monetization_goal:

   | Condition | Preferred Templates |
   |---|---|
   | **beauty / skincare niche** | `tiktok-product-on-face`, `cosmetic-texture`, `product-rotation` |
   | **shop monetization_goal** | `tiktok-before-after`, `product-rotation`, `appliance-usage` |
   | **fashion / accessories niche** | `tiktok-product-on-face` (→ `clothing-tryout` hint), `orbit-shot` |
   | **food niche** | `food-closeup`, `tiktok-before-after` |
   | **tech niche** | `product-rotation`, `orbit-shot`, `appliance-usage` |
   | **edu / finance niche** | `talking-head`, `tiktok-twist-ending` |
   | **comedy niche** | `tiktok-twist-ending`, `talking-head`, `tiktok-beat-sync` |
   | **Last non-CTA segment** (all niches) | `tiktok-twist-ending` (optional — use if twist adds value; skip if content does not support a twist) |
   | **Multi-angle fast cuts** | `tiktok-beat-sync` |

3. **Loading rule**: When a scene uses a TikTok-specific template (prefixed `tiktok-`), load the corresponding `references/templates/tiktok-{name}.md` file to access its full constraints, prompt examples, and caveats.

4. **Same-segment template consistency**: ALL scenes within the same `seedance_segment` MUST share the same `template_id`. This prevents style jump within a single Seedance generation.

5. **Template ID domain validation**: Every `scene_list[*].template_id` MUST be one of the 18 valid values (13 legacy + 5 TikTok-specific). See `references/template-registry.md` for the complete list.

---

#### 3d. Output Structure & Continuity Hint Generation

**Instructions**:

1. **Populate `scene_list[]`** with per-scene details:
   - `id` — sequential integer starting from 1
   - `start_sec` / `end_sec` — time boundaries in seconds (integer)
   - `shot_type` — from the assigned template's typical shot_type list
   - `template_id` — from the 18-template registry
   - `visual_description` — 1-2 sentence description of the scene's visual content
   - `voiceover` — (populated in Step 4, leave placeholder for now)
   - `on_screen_text` — (populated in Step 4, leave placeholder for now)
   - `retention_tactic` — from the template's retention_tactic domain (5 valid values)

2. **Populate `seedance_segments[]`** with segment-level grouping:
   - `segment_id` — sequential integer starting from 1
   - `start_sec` / `end_sec` / `duration_sec`
   - `scene_ids[]` — array of `scene_list[].id` values contained in this segment
   - `template_id` — the shared template for this segment's scenes
   - `continuity_hint` — cross-segment visual consistency string (see below)
   - `prompt_draft` — (populated in Step 5, leave empty for now)
   - `target_model` — (populated in Step 5, fixed `"seedance-2.0"`)
   - `template_hint` — (populated in Step 5, via downstream_mapping)
   - `reference_files` — `[]` initially, user can fill manually
   - `requires_manual_review` — `false` initially

3. **Generate `continuity_hint`** for each segment:

   - **segment[0]** → `continuity_hint = "opening"` (fixed value).
   
   - **All other segments** → generate a continuity_hint containing three components:
     - **Lighting continuity**: Describe the light direction that should persist from the previous segment. Example: `"保持上一段的左上方柔光"`
     - **Subject/Product position continuity**: Describe where the product/subject should appear in frame, matching the previous segment's last frame. Example: `"产品位置与上一段尾帧一致，在画面中央偏右下"`
     - **Color tone/filter continuity**: Describe the color temperature and filter that should persist. Example: `"继续使用暖色调，与上一段画面色彩无跳变"`
     
     Full continuity_hint example: `"保持上一段的左上方柔光、产品在画面中央偏右的位置，暖色调不变，确保拼接后无明显跳变。"`

4. **Segment-to-scene linking**: `seedance_segments[i].scene_ids` must point to valid `scene_list[*].id` values. For every `scene.id` in `scene_list`, there must be exactly one `seedance_segment` containing it. No orphan scenes.

5. **Output intermediate state**:
   ```
   [Step 3: Scene Orchestration]
   - duration_sec: {n}
   - total_scenes: {n}
   - total_seedance_segments: {n}
   - macro_rhythm: [{0-3s: hook}, {3-15s: value_show}, ...]
   - segments: [
       {id: 1, start: 0, end: 3, duration: 3, scenes: [1], template: tiktok-hook-3s, continuity: "opening"},
       {id: 2, start: 3, end: 15, duration: 12, scenes: [2,3], template: tiktok-product-on-face, continuity: "保持左上方柔光..."},
       ...
     ]
   - validation: {all_segments_≤15s: true, duration_sum_matches: true, template_domain_valid: true}
   ```

### Step 4: Copy Generation

**Status**: Full implementation (Cycle 3).

Generate all copy fields in strict DAG order. Dependencies must be respected — do not generate downstream fields before their upstream inputs are finalized.

**Copy DAG** (dependency graph):

```
hook (from Step 2) ──→ voiceover (per scene) ──→ on_screen_text (per scene)
                                                     │
                                                     ▼
                                                   cta ──────────────┐
                                                     │                │
                                                     ▼                ▼
                                                   title ◄── hashtags (trending + niche + branded)
                                                     │
                                                     ▼
                                                   cover
```

**Instructions**:

1. **Load** `references/monetization-playbooks.md`. Read the full file to access: §1–§5 (5 playbook definitions), §6 (CTA timing per video duration), §7 (playbook combination rules), §8 (language unification), §9 (brand_constraint integration), §10 (execution checklist).

2. **Branch** by `monetization_goal` to select the active playbook:
   - `crp` → §1 CRP Playbook (goal: completion rate + interaction, CTA type: `comment_question`/`like_prompt`/`share_bait`)
   - `shop` → §2 Shop Playbook (goal: conversion, CTA type: `shop_cta`)
   - `brand_deal` → §3 Brand Deal Playbook (goal: brand exposure + favorability, CTA type: `composite`)
   - `live_funnel` → §4 Live Funnel Playbook (goal: livestream entry, CTA type: `live_cta`)
   - `awareness` → §5 Awareness Playbook (goal: follow rate, CTA type: `follow_cta`)
   - Apply playbook combination rules from §7 if a secondary effect scenario is detected (e.g., `shop` + brand awareness as secondary goal).

3. **Generate in DAG order**:

   **3a. hook** — Already generated in Step 2. Reuse `hook_3s` values directly. Do not regenerate.

   **3b. voiceover (per scene)** — For each scene in `scene_list[]`:
   - Generate voiceover text matching the scene's visual_description and retention_tactic.
   - Must be in the `language` specified by the user (AC3).
   - If the playbook defines a specific opening (e.g., brand intro at 8-12s for `brand_deal`), place the opening voiceover in the scene closest to that timing.
   - For `crp` playbook: insert a soft mid-roll engagement prompt ("Did you know...?") at 40-50% of total duration, in the voiceover of the scene covering that time range.

   **3c. on_screen_text (per scene)** — For each scene:
   - Extract the key phrase from the scene's voiceover.
   - Must be in the `language` specified by the user, ≤6 characters (ZH) or ≤5 words (EN).
   - Hook scene (id=1): use the hook's on_screen_text from Step 2.

   **3d. cta** — Generate the CTA using the active playbook:
   - `cta.type`: Set to the playbook's CTA type (see §3 of the playbook).
   - `cta.text`: Select a template from the playbook's CTA templates (§1–§5). Choose the template that best fits the topic and content context. Fill in any template variables (e.g., `[Brand]`, `[product]`).
   - `cta.timing_sec`: Calculate per §6 of the playbook (CTA timing by video duration). Must be in the last seedance_segment's time window.
   - For `brand_deal`: the brand intro voiceover must appear at 8–12s (in the scene voiceover field), and the closing CTA must follow logo appearance rules from §3 of the playbook.

   **3e. title** — Generate a video title:
   - `12–25 words` for EN, `8–20 characters` for ZH.
   - Based on `topic` + hook angle. Use hook's voiceover or on_screen_text as the title nucleus.
   - Must be in the user's `language` (AC3).

   **3f. hashtags** — Generate four categories:
   - `hashtags.trending`: From `trending_inject.hashtags` if provided (AC11). If not provided, generate 3–5 niche-relevant trending hashtags based on `niche`, `platform_region`, and current best-practice patterns (e.g., #beautytok for beauty, #techtok for tech, #fyp for broad reach).
   - `hashtags.niche`: 3–5 niche-specific hashtags derived from `niche` and `topic`.
   - `hashtags.branded`: From `brand_constraint.must_say` or brand name if applicable. Empty array if no brand context.
   - `hashtags.rationale`: 1-sentence explanation of the hashtag strategy in the output `language`.

   **3g. cover** — Generate cover specification:
   - `cover.image_prompt`: Seedance-style visual description for a thumbnail frame. Should capture the hook's most compelling visual moment. 30–80 characters.
   - `cover.overlay_text`: Title abbreviation, ≤10 words EN / ≤8 chars ZH.
   - `cover.color_scheme`: Pick based on niche + mood (e.g., "high-contrast" for visual-shock hooks, "warm" for beauty/food, "cool-neon" for tech).

4. **Apply constraints**:
   - **Language unification**: All copy fields (voiceover, on_screen_text, cta.text, title, hashtags.rationale, cover.overlay_text) MUST be in the `language` parameter (AC3). Exception: `music_suggestion.tiktok_library_keywords` is always in English.
   - **brand_constraint.must_say**: Each phrase in this array MUST appear in at least one scene's `voiceover` or the `cta.text`. Verify before proceeding to Step 5. If missing, insert the phrase into the most contextually appropriate scene's voiceover.
   - **brand_constraint.banned**: Pass through to Step 6 (compliance pre-check). Do not use these words in any generated copy.
   - **trending_inject.sounds**: If provided, merge into `music_suggestion.tiktok_library_keywords` (AC11).
   - **trending_inject.hashtags**: If provided, write directly to `hashtags.trending` (AC11). Merge with auto-generated trending hashtags rather than replacing.

5. **Output intermediate state**:
   ```
   [Step 4: Copy Generation]
   - monetization_goal: {goal} → playbook: {playbook_section}
   - language: {lang}
   - cta_type: {type}
   - cta_timing_sec: {sec} (window: {start}-{end}s)
   - brand_must_say_included: true/false
   - brand_must_say_location: "{field}:{scene_id}" (or null)
   - trending_hashtags_injected: true/false
   - trending_sounds_injected: true/false
   - title_length: {n} words/chars
   - hashtag_count: trending={n}, niche={n}, branded={n}
   ```

### Step 5: Seedance Prompt Draft Generation

**Status**: Full implementation (Cycle 4).

Generate one `prompt_draft` per `seedance_segment` (not per scene), apply mirror self-checks against R1/R3/R4/R6, and populate all segment-level downstream fields.

---

#### 5.1 Load Mirror Rules

1. Load `references/seedance-mirror-rules.md`. This file defines:
   - **R1**: 14-word abstract adjective blacklist (史诗/震撼/绝美/电影感/大师之作/完美/高质量/超高清/8K/极致/惊艳/史无前例/无与伦比/终极)
   - **R3**: Single action per shot constraint
   - **R4**: Single camera movement per shot constraint
   - **R6**: Length (50-150 Chinese chars), no negative prompts, continuity constraint, multi-shot limit (≤3 shots)

2. Keep these rules in working memory throughout Step 5. Every `prompt_draft` generated must be checked against all four rules.

---

#### 5.2 Generate prompt_draft per Segment

For each `seedance_segments[i]`:

**5.2.1. Determine the generation format based on scene count**:

- **Single scene in segment** (scene_ids length = 1):
  - Format: `<镜头类型>：<动作描述>，<场景描述>，<光影描述>，<运镜描述>。`
  - Follow the video_prompt_gen eight-element framework: subject + action + scene + camera + lighting + style + optional audio + duration.

- **Multiple scenes in segment** (scene_ids length = 2-3):
  - Format: `镜头1：<动作+运镜> 镜头2：<动作+运镜> 镜头3：<动作+运镜> <整体节奏/风格描述>`
  - Each 镜头N block describes ONE shot independently (one action + one camera movement per shot).
  - Maximum 3 shots (R6d constraint). If a segment spans 4+ scenes, split into multiple segments in Step 3b.

**5.2.2. Generation constraints**:

| Constraint | Value | Source |
|---|---|---|
| **Language** | Chinese (Seedance 2.0 native language) | R6 (seedance-mirror-rules.md) |
| **Character count** | 50-150 Chinese characters | R6a |
| **Abstract adjectives** | NONE from the 14-word blacklist | R1 |
| **Actions per shot** | Exactly 1 per 镜头 block | R3 |
| **Camera moves per shot** | Exactly 1 per 镜头 block | R4 |
| **Negative prompts** | NONE — use positive description instead | R6b |
| **Shot count** | ≤ 3 per prompt_draft | R6d |

**5.2.3. Content generation rules per template**:

- Load the template file `references/templates/{template_id}.md` for the segment.
- Use the template's `seedance_prompt_example` as a style reference (not copy).
- Apply the template's `typical_shot_type` to frame the shot description.
- Incorporate the template's `caveats` into the generated prompt (e.g., for `tiktok-product-on-face`: use "模特" not specific person descriptors).

**5.2.4. Multi-scene merge example**:

Segment with 2 scenes (产品正面展示 + 产品 45 度展示), template = `product-rotation`:

```
镜头1：产品正面特写，纯黑背景，环形柔光，镜头静止 2 秒。
镜头2：产品 45 度俯拍，金属台面反光，镜头轻微右移展示侧面纹理 2 秒。
整体暖色调，背景纯黑高对比，画面简洁干净。
```

---

#### 5.3 Append Continuity Constraint

For **segment[0]**: No continuity constraint appended (it is the opening segment; `continuity_hint = "opening"`).

For **all other segments** (segment_id > 1): Append the following sentence to the END of `prompt_draft`:

```
保持与上一段画面的光照、色调、主体位置连续性，确保拼接后无明显跳变。
```

**Note**: This continuity sentence does NOT count toward the 50-150 character limit (R6c from seedance-mirror-rules.md).

---

#### 5.4 Populate Downstream Fields

For each segment, fill the remaining fields:

| Field | Value | Source |
|---|---|---|
| `target_model` | `"seedance-2.0"` | Fixed — all tiktok-script-gen prompts target Seedance 2.0 |
| `template_hint` | Mapped value from downstream_mapping table | See §5.4.1 below |
| `reference_files` | `[]` | Empty initially; user can add `@图片1` / `@视频1` references manually |
| `requires_manual_review` | `false` initially | Set to `true` if mirror self-check fails after 1 rewrite attempt |

**5.4.1. template_hint mapping logic**:

Look up the segment's `template_id` in `template-registry.md` downstream_mapping table:

1. If `template_id` is one of the 13 legacy templates → `template_hint` = `template_id` (identity mapping).
2. If `template_id` is one of the 5 TikTok-specific templates → use the downstream_mapping table. Select the target based on **niche**:
   - `tiktok-hook-3s`:
     - beauty → `cosmetic-texture`
     - food → `food-closeup`
     - shop / tech / other → `product-rotation`
     - edu / finance / comedy / travel / fitness → `talking-head`
   - `tiktok-product-on-face`:
     - beauty / other → `cosmetic-texture`
     - shop (fashion/accessories) → `clothing-tryout`
   - `tiktok-before-after`:
     - beauty / fitness / other → `cosmetic-texture`
     - tech / shop / food → `appliance-usage`
   - `tiktok-beat-sync`:
     - all (default) → `product-rotation`
     - travel / tech (dynamic) → `orbit-shot`
   - `tiktok-twist-ending`:
     - all (narrative-style) → `narrative-suspense`
     - edu / comedy (talking) → `talking-head`

3. **Validation**: The resulting `template_hint` MUST be one of the 13 valid video_prompt_gen template IDs (AC14). If the mapping produces an unrecognized value, fallback to `product-rotation`.

---

#### 5.5 Mirror Self-Check & Rewrite

After generating all `prompt_draft` values, run the self-check for each segment:

**5.5.1. Check R1 (14-word blacklist)**:
- Scan `prompt_draft` for any of the 14 banned abstract adjectives (whole-word matching for Chinese, substring for English variants).
- Match count determines action:
  - **0 matches** → PASS. No action.
  - **1-2 matches** → SOFT WARNING. Replace each with a specific visual description.
  - **3+ matches** → FORCE REWRITE. Replace all abstract adjectives with specific visual descriptions.

**5.5.2. Check R3 (multi-action)**:
- For each 镜头 block: count the number of main verbs describing distinct actions.
- If > 1 action per block → VIOLATION. Split into separate shots or pick the primary action.
- Exception: natural causal chains ("倒入杯中，液面上升" — pouring causes rising = 1 action).

**5.5.3. Check R4 (multi-camera-move)**:
- For each 镜头 block: count camera movement keywords (推/拉/摇/移/跟/环绕/变焦/orbit/dolly/pan/tilt/track/zoom).
- If > 1 camera move per block → VIOLATION. Reduce to single camera movement.

**5.5.4. Check R6 (length/format)**:
- R6a: Character count 50-150 (excluding continuity sentence).
- R6b: No negative prompt syntax (不要/别/禁止/no/don't).
- R6c: Continuity sentence appended for non-first segments.
- R6d: ≤ 3 shots per prompt_draft.

**5.5.5. Rewrite strategy**:

```
For each segment with violations:
  1. DETECT: Identify which rule(s) violated and the specific offending text.
  2. PRIORITIZE: Fix in order: R6b > R1 > R3 > R4 > R6a.
  3. REWRITE ONCE: Modify prompt_draft to resolve all violations.
  4. RE-CHECK: Run all 4 rules again on the rewritten prompt_draft.
  5. DECIDE:
     - ALL PASS → output the rewritten prompt_draft. requires_manual_review = false.
     - STILL FAILING → keep the best version (least violations). Set requires_manual_review = true.
                       Add a note: "{rule_id} 自检未通过，已标记为需人工审核。"
```

**5.5.6. R1 replacement quick reference**:

| Instead of... | Use... |
|---|---|
| "震撼的画面" | "高对比度画面，前景物体快速推入镜头" |
| "绝美的风景" | "暖色调日落，金色光线穿透云层" |
| "电影感" | "21:9 宽银幕比例，柔光滤镜，暖色调调色" |
| "完美构图" | "三分法构图，主体位于画面右下交叉点" |
| "超高清/8K" | "锐利对焦，纹理清晰可见" |
| "高质量" | "锐利对焦，低噪点，专业级画质" |
| "惊艳" | "逆光剪影、光影穿透水珠" |
| "终极/极致" | "（直接删除，用具体动作替代）" |

---

#### 5.6 Output Intermediate State

```
[Step 5: Seedance Prompt Generation]
- segment_count: {n}
- segments_checked: {n}
- segments_passed_first_try: {n}
- segments_rewritten: {n}
- segments_marked_manual_review: {n}
- mirror_check_details:
    - R1_violations: [{segment_id, word, replacement}]
    - R3_violations: [{segment_id, shot_block, action_count}]
    - R4_violations: [{segment_id, shot_block, camera_move_count}]
    - R6_violations: [{segment_id, rule_type, detail}]
- template_hint_validation: {all_valid: true/false, invalid_segments: []}
```

### Step 6: Compliance Check

**Status**: Full implementation (Cycle 5).

Execute a two-phase compliance pipeline. This step loads `references/compliance-rules.md` and applies all rules from that file. Compliance checks NEVER block output — issues are annotated, not rejected (per Q4/D6 decision).

---

#### 6a. Pre-Check (Before Copy Generation)

Execute BEFORE Step 4 copy generation. This intercepts input-level contamination.

**Instructions**:

1. **Load** `references/compliance-rules.md` §2 (banned word blacklist, 20+ terms ZH+EN) and §1 (duration warning rules).

2. **Scan user-supplied inputs** against the banned word blacklist:
   - `brand_constraint.banned` — user-supplied forbidden words, merged with the built-in blacklist for this session only.
   - `trending_inject.sounds` — if contains Spotify/Apple Music/SoundCloud link patterns, auto-rewrite by extracting genre keywords. Populate `compliance_check.music_copyright_advice` with a warning.

3. **Evaluate duration thresholds** (per compliance-rules.md §1):
   - `duration_sec < 60` → set `meta.duration_meets_crp = false`, append CRP ineligibility warning to `compliance_check.notes`.
   - `60 <= duration_sec < 75` → append marginal duration note.
   - `duration_sec > 180` → append completion rate risk note.

4. **Output intermediate state**:
   ```
   [Step 6a: Pre-Check]
   - duration_meets_crp: {true/false}
   - duration_notes: [{...}]
   - brand_banned_scanned: {count} words checked
   - trending_sounds_scanned: clean / {issues found}
   ```

---

#### 6b. Final Check (After All Outputs Generated)

Execute AFTER Steps 2-5 have produced all output fields. Scan the complete output for violations.

**6b.1. Banned Word Scan**:

Scan ALL output copy fields against compliance-rules.md §2 blacklist:
- Fields scanned: `voiceover` (all scenes), `on_screen_text` (all scenes), `cta.text`, `title`, `visual_description` (all scenes), `prompt_draft` (all segments), `cover.overlay_text`, `cover.image_prompt`.
- Matching rules (per compliance-rules.md §2.3):
  - **Chinese terms**: substring matching (e.g., "最便宜" matches "全网最便宜的...").
  - **English terms**: case-insensitive whole-word matching (e.g., "cheapest ever" does NOT match "affordable").
  - Words in `brand_constraint.must_say` are excluded from banned word scanning.
- Each hit recorded in `compliance_check.banned_words_hit[]` as: `{ "word": "...", "location": "scene_{id}.voiceover", "severity": "high" }`.
- The skill auto-rewrites detected hits during Step 4 (already done). This scan catches any that slipped through. If hits are found, record them but do NOT block output.

**6b.2. Watermark Detection Scan**:

Scan `visual_description`, `on_screen_text`, `prompt_draft` against compliance-rules.md §3 detection keywords:
- **Platform watermark keywords** (9 platforms): instagram/youtube/facebook/douyin/xiaohongshu/snapchat/bilibili/weibo/wechat watermark patterns.
- **@username from <other platform>** pattern: regex `@[\w.]+.*(instagram|youtube|facebook|douyin|xiaohongshu|snapchat|bilibili|weibo|wechat|twitter|x\.com)`.
- If ANY keyword matches → populate `compliance_check.watermark_warning` with the warning template:
  - ZH: `"检测到 {keyword}，TikTok 不分成第三方水印素材，请确保最终视频不含此元素。"`
  - EN: `"Detected {keyword}. TikTok does not monetize content with third-party watermarks. Ensure the final video is free of this element."`
- Only ONE warning is emitted (first match). If no matches → `watermark_warning = null`.

**6b.3. Music Copyright Validation**:

Validate `music_suggestion` against compliance-rules.md §4:
- **tiktok_library_keywords**: MUST have `length >= 1` (AC9). If empty, populate with at least one keyword from the recommended pool:
  - `trending tiktok sound` / `tiktok original audio` / `commercial library upbeat pop` / `royalty-free lofi`
- **Prohibited link patterns**: Scan for Spotify/Apple Music/SoundCloud/YouTube/Tidal/Deezer/Amazon Music link patterns. If found, replace with TikTok Commercial Music Library keywords.
- **Prohibited commercial song references**: If a direct artist + song title is detected (e.g., "Taylor Swift - Shake It Off"), replace with genre description (e.g., "upbeat pop similar to early 2010s dance-pop").
- **music_copyright_advice**: MUST ALWAYS be populated (AC9). Default text:
  - ZH: `"建议使用 TikTok Commercial Music Library 中的曲目，避免外链 Spotify/Apple Music/SoundCloud 等第三方平台音乐。"`
  - EN: `"Use tracks from the TikTok Commercial Music Library. Avoid linking to third-party platforms like Spotify, Apple Music, or SoundCloud."`
- Set `music_suggestion.license_safe = true` (always; we default to safe recommendations).

**6b.4. Duration Validation (re-confirm)**:

Re-confirm the `duration_meets_crp` flag from pre-check is propagated to both locations:
- `meta.duration_meets_crp` (set in Step 1 boundary handling, re-confirmed here).
- `compliance_check.duration_meets_crp` (MUST equal `meta.duration_meets_crp` — AC1).

**6b.5. originality_ok**:

- Default value: `true` (we generate original content by design).
- Set to `false` if watermark warning is non-null (content contains references to third-party platforms).

---

#### 6c. Output Assembly

After all checks complete, assemble the `compliance_check` object:

```json
{
  "originality_ok": true | false,
  "watermark_warning": null | "检测到 {keyword}...",
  "banned_words_hit": [
    { "word": "最便宜", "location": "scene_3.voiceover", "severity": "high" }
  ],
  "duration_meets_crp": true | false,
  "music_copyright_advice": "建议使用 TikTok Commercial Music Library...",
  "notes": [
    "时长 45s 不足 60s（实际 45s），不计入 TikTok CRP（Creator Rewards Program）收益。如需 CRP 准入，请将 duration_sec 调整为 >=60。"
  ]
}
```

**Non-blocking principle (Q4/D6)**: ALL compliance issues are annotations only. The full script is delivered regardless of warnings. The user is the final reviewer — flagged items help them make informed decisions before publishing.

---

#### 6d. Output Intermediate State

```
[Step 6: Compliance Check]
- pre_check_passed: true/false
- banned_words_hit: [{word, location, severity}] (count: {n})
- watermark_warning: null / "检测到 {keyword}..."
- duration_meets_crp: true/false
- music_copyright_advice: "{advice text}"
- originality_ok: true/false
- notes: ["...", "..."]
- all_checks_non_blocking: true
```

### Step 7: KPI Estimation Lookup

**Status**: Full implementation (Cycle 5).

Look up expected completion rate and RPM ranges from the KPI reference table. All values are estimates — NOT guarantees (per KPI table header disclaimer). No improvement suggestions are offered (Q7 decision: avoid misleading auto-suggestions).

---

#### 7a. Determine Duration Bucket

Classify `duration_sec` into one of 6 buckets using the following boundary logic (inclusive ranges):

```
if duration_sec < 60           → bucket = "<60s"
elif 60 <= duration_sec <= 74  → bucket = "60-74s"
elif 75 <= duration_sec <= 89  → bucket = "75-89s"
elif 90 <= duration_sec <= 119 → bucket = "90-119s"
elif 120 <= duration_sec <= 180 → bucket = "120-180s"
else (duration_sec > 180)      → bucket = ">180s"
```

Edge case: if `duration_sec` is exactly on a boundary (e.g., exactly 60), it falls into the lower bucket (`60-74s`) because the "60-74s" range is inclusive (`60 <= duration_sec <= 74`).

---

#### 7b. Lookup Completion Rate

1. **Load** `references/kpi-table.md` (already loaded in §2, but confirm the completion rate table in §3.1 is accessible).

2. **Query**: Use `{niche} × {duration_bucket}` as the lookup key in the completion rate table (10 niches × 6 columns).

3. **Fallback**: If `niche` is not in the 10-class enum (e.g., user typed an unrecognized value that was already resolved to `"other"` in Step 1), use the `other` row.

4. **Result assignment**: Set `meta.estimated_kpi.expected_completion_rate = "{min}%-{max}%"` (e.g., `"48-60%"`).

5. **Pattern awareness** (for notes, not for rewriting):
   - **Emotion-type niches** (comedy, beauty, food): completion rate decreases as duration increases. This is expected — shorter content works better for emotional impact.
   - **Information-type niches** (edu, tech, finance): completion rate may peak at 90-119s. Viewers seeking depth may watch longer.
   - **Shop niche**: steepest completion rate decline with duration — purchase decisions are quick.

---

#### 7c. Lookup RPM

1. **Query**: Use `{niche} × {duration_bucket}` as the lookup key in the RPM table (10 niches × 6 columns).

2. **<60s special rule**: If the bucket is `"<60s"`, RPM is FORCED to `"0-0"` regardless of what the table shows (CRP only counts views on videos >= 60 seconds). Also check that `compliance_check.notes` contains the Rule 1.1 CRP ineligibility warning (should have been added in Step 6a).

3. **Result assignment**: Set `meta.estimated_kpi.expected_rpm_usd = "{min}-{max}"` (e.g., `"0.5-0.8"`). For `<60s` bucket: `"0-0"`.

4. **Monotonicity assertion** (AC7, verification-only, no action needed):
   - The KPI table in `kpi-table.md` §3.3 has been pre-verified: all 10 niches pass monotonicity (RPM upper bound is non-decreasing from `60-74s` through `120-180s`).
   - This step does NOT recalculate monotonicity — it relies on the pre-verified table.
   - If the user requests a duration outside these buckets (e.g., 200s), the `>180s` bucket values apply with the expected RPM degradation noted.

5. **RPM degradation at >180s**: Note that RPM drops for `>180s` vs `120-180s` for all niches — ultra-long videos have lower completion rates and therefore lower effective RPM.

---

#### 7d. Niche Not in Table

If `niche` was already resolved to `"other"` in Step 1 (because user input was not in the 10-class enum), the lookup automatically uses the `other` row. The Step 1 niche fallback note in `compliance_check.notes` already explains this. No additional action needed here.

---

#### 7e. No Improvement Suggestions

Per Q7 decision, this step does NOT generate improvement suggestions such as "your hook is weak, try..." or "increase duration to 90s for better RPM." The `estimated_kpi` is purely informational. If the user wants optimization advice, they will ask explicitly.

---

#### 7f. Output Intermediate State

```
[Step 7: KPI Estimation]
- niche: {niche}
- duration_sec: {n} → bucket: {bucket}
- expected_completion_rate: "{min}%-{max}%"
- expected_rpm_usd: "{min}-{max}" (or "0-0" for <60s)
- niche_fallback_applied: true/false
- monotonicity_pre_verified: true (table-level, AC7)
```

### Step 8: Output Serialization

**Status**: Full implementation (Cycle 5).

Serialize the complete script into the requested output format(s). This step also generates cross-platform adaptation tips (AC13) and appends the archive signature block.

---

#### 8a. Format Branching

Branch by the `output_format` parameter (default: `"both"`):

| output_format | Action |
|---|---|
| `"both"` | Output Markdown first, then JSON (in a code block). |
| `"markdown"` | Output Markdown only. |
| `"json"` | Output JSON only. |

---

#### 8b. Markdown Output

Structure the Markdown output following the field order defined in requirements.md §5. Each section below is mandatory.

**8b.1. Document Header**:

```markdown
# TikTok Video Script: {topic}
> Generated by tiktok-script-gen v1 | Niche: {niche} | Goal: {monetization_goal} | Duration: {duration_sec}s | Language: {language}
```

**8b.2. Meta** (always first section):

```markdown
## Script Meta
- **Duration**: {duration_sec}s
- **CRP Eligible**: {meta.duration_meets_crp ? "Yes (>=60s)" : "No (<60s — CRP requires >=60s)"}
- **Language**: {meta.language}
- **Niche**: {meta.niche}
- **Monetization Goal**: {meta.monetization_goal}
- **Estimated KPI**:
  - Completion Rate: {estimated_kpi.expected_completion_rate}
  - RPM (USD): {estimated_kpi.expected_rpm_usd}
```

**8b.3. Hook (3s)** (with variants):

```markdown
## Hook (First 3 Seconds)

### Primary Hook
- **Visual**: {hook_3s.visual}
- **Voiceover**: "{hook_3s.voiceover}"
- **On-Screen Text**: "{hook_3s.on_screen_text}"
- **Retention Tactic**: {hook_3s.retention_tactic}

### Hook Variants ({n} total)
#### Variant 1: {variant_description}
- **Visual**: {variant.visual}
- **Voiceover**: "{variant.voiceover}"
- **On-Screen Text**: "{variant.on_screen_text}"
- **Retention Tactic**: {variant.retention_tactic}

[Repeat for all variants]
```

**8b.4. Scene List** (one subsection per scene):

```markdown
## Full Scene Breakdown

### Scene {id}: {start_sec}s–{end_sec}s ({end_sec - start_sec}s)
- **Shot Type**: {shot_type}
- **Template**: {template_id}
- **Visual Description**: {visual_description}
- **Voiceover**: "{voiceover}"
- **On-Screen Text**: "{on_screen_text}"
- **Retention Tactic**: {retention_tactic}

[Repeat for all scenes]
```

**8b.5. CTA (Call-to-Action)**:

```markdown
## Call-to-Action (CTA)
- **Type**: {cta.type}
- **Timing**: {cta.timing_sec}s
- **Text**: "{cta.text}"
```

**8b.6. Hashtags**:

```markdown
## Hashtags
- **Trending**: {hashtags.trending | join(" ")}
- **Niche**: {hashtags.niche | join(" ")}
- **Branded**: {hashtags.branded | join(" ") or "(none)"}
- **Strategy**: {hashtags.rationale}
```

**8b.7. Cover**:

```markdown
## Cover / Thumbnail
- **Image Prompt**: {cover.image_prompt}
- **Overlay Text**: "{cover.overlay_text}"
- **Color Scheme**: {cover.color_scheme}
```

**8b.8. Title**:

```markdown
## Video Title
**{title}**
```

**8b.9. Music Suggestion**:

```markdown
## Music & Sound
- **Mood**: {music_suggestion.mood}
- **BPM Range**: {music_suggestion.bpm_range}
- **License Safe**: {music_suggestion.license_safe ? "Yes (TikTok Commercial Library)" : "Check manually"}
- **Library Keywords**: {music_suggestion.tiktok_library_keywords | join(", ")}
```

**8b.10. Seedance Segments** (independent cards per segment):

```markdown
## Seedance 2.0 Generation Segments ({n} segments total)

### Segment {segment_id}/n: {start_sec}s–{end_sec}s ({duration_sec}s)
- **Scenes Included**: {scene_ids | join(", ")}
- **Template**: {template_id} (hint: {template_hint})
- **Continuity**: {continuity_hint}
- **Target Model**: {target_model}
- **Prompt Draft**:
  > {prompt_draft}
- **Reference Files**: {reference_files | join(", ") or "(none)"}
- **Manual Review Needed**: {requires_manual_review ? "YES — mirror check failed after rewrite" : "No"}

[Repeat for all segments]
```

**8b.11. Compliance Check**:

```markdown
## Compliance Check
- **Originality**: {originality_ok ? "OK" : "Warning — see notes"}
- **Watermark Warning**: {watermark_warning || "(none)"}
- **Banned Words Detected**: {banned_words_hit.length > 0 ? banned_words_hit list : "(none)"}
- **CRP Eligible**: {duration_meets_crp ? "Yes" : "No"}
- **Music Copyright**: {music_copyright_advice}
- **Notes**: {notes | join("; ") or "(none)"}
```

**8b.12. Cross-Platform Tips** (AC13):

```markdown
## Cross-Platform Adaptation Tips

### Instagram Reels
{ig_reels adaptation tips — 2-4 sentences, covering max 90s, 9:16 ratio, hashtag strategy differences,
music copyright strictness, and any content adjustments needed for IG audience expectations.}

### YouTube Shorts
{yt_shorts adaptation tips — 2-4 sentences, covering max 60s limit, title SEO optimization,
description link strategy, hashtag differences (#Shorts vs TikTok trend tags),
and any content pacing adjustments for YT's different audience behavior.}
```

**cross_platform_tips generation rules**:

For `ig_reels`:
- Mention the 90s max length constraint. If the script is >90s, suggest which segments to trim.
- Note that IG Reels has stricter music copyright enforcement — recommend using Meta Sound Collection.
- Hashtag strategy: fewer hashtags (3-5 max), more niche-specific, less trending-spam.
- Audience expectation: IG Reels audience tends to prefer polished/curated aesthetics over raw TikTok energy.

For `yt_shorts`:
- Mention the 60s hard limit. If the script is >60s, suggest a condensed version or series split.
- Title optimization: YouTube's search-driven discovery means title SEO matters. Suggest keyword-rich title variant.
- Description strategy: include timestamps, product links, and subscribe CTA in the description.
- Hashtag strategy: use #Shorts + 2-3 niche tags (YouTube hashtags work differently than TikTok).
- Both fields MUST be non-empty strings (AC13).

**8b.13. Signature Block** (append at the very end, after all content):

```markdown
---
本次使用 skill：tiktok-script-gen v1
KPI 表版本：v1
合规规则版本：v1
template_registry：v1
```

---

#### 8c. JSON Output

1. **Load** `references/output-schema.json` (Draft-07 JSON Schema defining all required fields and constraints).

2. **Serialize** all script fields into a JSON object matching the schema structure. The JSON follows the same hierarchical field organization as the YAML schema in §5.

3. **Validation** (in-prompt reasoning, not runtime code):

   a. **Required field check**: Verify all 11 top-level `required` fields are present:
      `meta`, `hook_3s`, `scene_list`, `seedance_segments`, `cta`, `hashtags`, `cover`, `title`, `music_suggestion`, `compliance_check`, `cross_platform_tips`.

   b. **Nested required check**: Verify `meta.estimated_kpi`, `hook_3s.variants` (minItems: 1), `cta.type/cta.text/cta.timing_sec`, `seedance_segments[*]` (all 10 required sub-fields present), `compliance_check` (all 6 required sub-fields present).

   c. **Type constraint check**: Verify types match schema definitions:
      - `duration_sec` is integer
      - `duration_meets_crp` / `originality_ok` / `license_safe` / `requires_manual_review` are boolean
      - `scene_ids` / `hashtags.trending` / `hashtags.niche` / `hashtags.branded` / `reference_files` / `tiktok_library_keywords` / `banned_words_hit` / `notes` are arrays
      - `seedance_segments[].duration_sec` is a number with `maximum: 15`

   d. **Enum constraint check**:
      - `niche` is one of 10 valid values
      - `monetization_goal` is one of 5 valid values
      - `cta.type` is one of 7 valid values
      - `hook_3s.retention_tactic` is one of 5 valid values
      - `seedance_segments[].duration_sec <= 15` (AC16)

4. **Fallback strategy** (degraded-field handling, per R8):
   - Array fields with no content → `[]` (not `null`, not missing).
   - String fields with no content → `""` (not `null`, not missing).
   - Boolean fields → MUST be explicitly `true` or `false` (never `null`).
   - Number fields → MUST have a value (never `null`). Use `0` as last-resort fallback for required numeric fields.
   - Missing `seedance_segments` → generate at least 1 segment (even for very short videos, there must be 1 segment).
   - Missing `cross_platform_tips` sub-fields → populate with generic adapter messages (AC13 requires non-empty).

5. **Schema validation reporting**: At the end of the JSON output (or as a note alongside it), report:
   ```json
   "_schema_validation": {
     "passed": true,
     "errors": []
   }
   ```
   If validation fails, `passed: false` with `errors` listing each missing/invalid field. Adjust the output to fix missing fields rather than just reporting. The goal is `passed: true` for every output.

6. **JSON output wrapping**: Present as a fenced code block with `json` language tag when `output_format=both` (alongside Markdown). When `output_format=json`, output raw JSON directly.

---

#### 8d. Output Intermediate State

```
[Step 8: Output Serialization]
- output_format: {format}
- markdown_sections_emitted: [meta, hook_3s, scene_list, cta, hashtags, cover, title, music, compliance, seedance, cross_platform, signature]
- json_schema_validation: passed / failed
- schema_errors: [{field, issue}] (count: {n})
- cross_platform_tips: ig_reels={non_empty}, yt_shorts={non_empty}
- signature_block_appended: true
```

---

## 5. Output Format Specification

This section defines the complete output structure. The canonical schema is `references/output-schema.json` (Draft-07) — all JSON output MUST validate against it (AC15). The Markdown output follows the same hierarchical structure with human-readable formatting.

---

### 5.1 YAML Schema Outline (All Fields + Annotations)

```yaml
script:
  # ── §1 Meta: duration, CRP eligibility, language, niche, monetization, KPI ──
  meta:
    duration_sec: 60                          # int, >=0; target video length
    duration_meets_crp: true                  # bool; >=60 → true, <60 → false (AC1/AC2)
    language: "zh"                            # string; output language code (AC3)
    niche: "beauty"                           # enum; 1 of 10: beauty/tech/edu/food/travel/fitness/finance/comedy/shop/other
    monetization_goal: "crp"                  # enum; 1 of 5: crp/shop/brand_deal/live_funnel/awareness
    estimated_kpi:                            # object; from kpi-table.md lookup (AC7)
      expected_completion_rate: "48-60%"      # string; format "{min}%-{max}%"
      expected_rpm_usd: "0.5-0.8"             # string; format "{min}-{max}" or "0-0" for <60s
    total_seedance_segments: 5                # int; optional, useful for >8 segments (R13)

  # ── §2 Hook (First 3s): primary hook + A/B variants ──
  hook_3s:                                    # 抓手 #1: first 3-5s retention
    visual: "红色口红从画面右侧快速推入"         # string; visual shot description
    voiceover: "黄皮涂这支口红直接白两个度？"     # string; voiceover text (≤8 words EN / ≤12 chars ZH)
    on_screen_text: "黄皮救星"                 # string; overlay text (≤6 chars ZH / ≤5 words EN)
    retention_tactic: "curiosity-question"    # enum; visual-shock/curiosity-question/pain-point/number-promise/contrast
    variants:                                 # array; length = ab_variants (default 2, max 5) (AC5)
      - visual: "..."
        voiceover: "..."
        on_screen_text: "..."
        retention_tactic: "visual-shock"

  # ── §3 Scene List: full scene-by-scene breakdown ──
  scene_list:                                 # array; min 1 element
    - id: 1                                   # int; sequential, starts at 1
      start_sec: 0                            # number; scene start time
      end_sec: 3                              # number; scene end time
      shot_type: "close-up"                   # string; from template's typical_shot_type
      template_id: "tiktok-hook-3s"           # string; MUST be 1 of 18 templates (AC4/AC14)
      visual_description: "..."               # string; 1-2 sentence visual description
      voiceover: "..."                        # string; scene voiceover text
      on_screen_text: "..."                   # string; scene overlay text
      retention_tactic: "curiosity-question"  # string; scene-level retention tactic

  # ── §4 Seedance Segments: 15s-capped generation groups ──
  seedance_segments:                          # array; min 1 element (AC10/AC16)
    - segment_id: 1                           # int; sequential, starts at 1
      start_sec: 0                            # number; segment start time
      end_sec: 3                              # number; segment end time
      duration_sec: 3                         # number; MUST be <=15 (AC16)
      scene_ids: [1]                          # array; 1-3 scene IDs in this segment
      template_id: "tiktok-hook-3s"           # string; internal TikTok template
      template_hint: "cosmetic-texture"       # string; mapped to video_prompt_gen 13 templates (AC14)
      continuity_hint: "opening"              # string; "opening" for seg[0]; lighting/position/color for others (AC10)
      prompt_draft: "特写镜头：一支红色口红从画面右侧快速推入..."  # string; Chinese, 50-150 chars (AC10)
      target_model: "seedance-2.0"            # string; fixed value
      reference_files: []                     # array; user-fillable @图片1 references
      requires_manual_review: false           # bool; true if mirror check failed after rewrite

  # ── §5 CTA: Call-to-Action ──
  cta:                                        # 抓手 #9: interaction / conversion prompt
    type: "comment_question"                  # enum; comment_question/like_prompt/share_prompt/shop_cta/live_cta/follow_cta/composite
    text: "你们觉得哪个色号最显白？评论区告诉我！"  # string; CTA text
    timing_sec: 57                            # number; when CTA triggers (last 3-10s)

  # ── §6 Hashtags: trending + niche + branded ──
  hashtags:                                   # 抓手 #7: discoverability
    trending: ["#summerglow", "#beautytok"]   # array; from trending_inject or auto-generated (AC11)
    niche: ["#lipstickreview", "#kbeauty"]    # array; niche-specific tags
    branded: ["#YourBrand"]                   # array; brand tags or []
    rationale: "组合热门标签+垂类标签，覆盖FYP和搜索双通道"  # string; strategy explanation

  # ── §7 Cover / Thumbnail ──
  cover:                                      # 抓手 #7: CTR optimization
    image_prompt: "红色口红特写，高对比黑背景"  # string; Seedance-style visual description
    overlay_text: "黄皮救星"                   # string; title abbreviation (≤10 words EN / ≤8 chars ZH)
    color_scheme: "high-contrast"             # string; high-contrast/warm/cool-neon/minimal

  # ── §8 Title ──
  title: "黄皮必看！这支口红让我直接白了两个度｜显白口红测评"  # string; 12-25 words EN / 8-20 chars ZH

  # ── §9 Music Suggestion ──
  music_suggestion:                           # 抓手 #6: music enhances retention
    mood: "upbeat"                            # string; music mood/genre
    bpm_range: "120-130"                      # string; recommended BPM range
    license_safe: true                        # bool; always true (we recommend TikTok Library)
    tiktok_library_keywords:                  # array; min 1 item (AC9)
      - "trending tiktok sound"
      - "upbeat corporate instrumental"

  # ── §10 Compliance Check ──
  compliance_check:                           # 抓手 #3: platform compliance annotations
    originality_ok: true                      # bool; false if watermark_warning is non-null
    watermark_warning: null                   # string|null; populated if watermark detected (AC8)
    banned_words_hit: []                      # array; [{word, location, severity}] (AC8)
    duration_meets_crp: true                  # bool; must equal meta.duration_meets_crp (AC1)
    music_copyright_advice: "建议使用 TikTok Commercial Music Library 中的曲目..."  # string; always present (AC9)
    notes: []                                 # array; accumulated warnings and notes

  # ── §11 Cross-Platform Tips ──
  cross_platform_tips:                        # AC13: both fields must be non-empty strings
    ig_reels: "适配 IG Reels：控制在 90s 内，使用 Meta Sound Collection 音乐..."  # string; 2-4 sentences
    yt_shorts: "适配 YouTube Shorts：时长限制 60s，建议拆分为上下集..."           # string; 2-4 sentences
```

---

### 5.2 Complete Script Example (Topic: "口红显白测评")

> **Parameters**: topic="口红显白测评" niche=beauty monetization_goal=crp duration_sec=60 language=zh
> **Scenario**: A Chinese beauty creator testing a red lipstick for yellow/warm skin tones, targeting CRP (Creator Rewards Program) revenue through high completion rate and engagement.

Below is a **condensed but structurally complete** example showing all fields with real filled values. This serves as both a user-facing example and a Claude reference for generating similar outputs.

````markdown
# TikTok Video Script: 口红显白测评

> Generated by tiktok-script-gen v1 | Niche: beauty | Goal: crp | Duration: 60s | Language: zh

## Script Meta
- **Duration**: 60s
- **CRP Eligible**: Yes (>=60s)
- **Language**: zh
- **Niche**: beauty
- **Monetization Goal**: crp
- **Estimated KPI**:
  - Completion Rate: 48-60%
  - RPM (USD): 0.5-0.8

## Hook (First 3 Seconds)

### Primary Hook
- **Visual**: 黑背景，一支红色口红从画面右侧快速推入，停在金属台面中央，台面反光高对比。轻微 dolly-in 运镜约 1 秒入场，最后 2 秒静止聚焦膏体细节。
- **Voiceover**: "黄皮涂这支口红，直接白两个度？"
- **On-Screen Text**: "黄皮救星"
- **Retention Tactic**: curiosity-question

### Hook Variants (2 total)
#### Variant 1
- **Visual**: 黑白滤镜下模特素颜半脸，手指轻触屏幕，画面瞬间切为彩色，唇部成为唯一高饱和色块。
- **Voiceover**: "同一张脸，涂这支口红前后差别有多大？"
- **On-Screen Text**: "差距惊人"
- **Retention Tactic**: contrast

## Full Scene Breakdown

### Scene 1: 0s–3s (3s)
- **Shot Type**: close-up
- **Template**: tiktok-hook-3s
- **Visual Description**: 黑背景，红色口红从画面右侧快速推入，停在金属台面中央。
- **Voiceover**: "黄皮涂这支口红，直接白两个度？"
- **On-Screen Text**: "黄皮救星"
- **Retention Tactic**: curiosity-question

### Scene 2: 3s–8s (5s)
- **Shot Type**: half-face close-up
- **Template**: tiktok-product-on-face
- **Visual Description**: 模特半脸特写，左上方柔光，嘴唇未涂口红，肤色偏黄。唇部占画面 1/3。
- **Voiceover**: "先看我的素唇状态，典型的黄一白，唇色偏深。"
- **On-Screen Text**: "素唇状态"
- **Retention Tactic**: pain-point

### Scene 3: 8s–13s (5s)
- **Shot Type**: half-face close-up
- **Template**: tiktok-product-on-face
- **Visual Description**: 同一角度同一光照，模特唇部已涂满红色口红，唇纹细腻，颜色饱和。
- **Voiceover**: "涂一层，瞬间整个人白了一个色号，而且不挑唇部状态。"
- **On-Screen Text**: "涂一层"
- **Retention Tactic**: pain-point

### Scene 4: 13s–20s (7s)
- **Shot Type**: 45-degree portrait
- **Template**: cosmetic-texture
- **Visual Description**: 45 度角半身镜头，模特自然转头展示不同光线下的唇色，柔光从左上方不变。
- **Voiceover**: "室内光、室外光都给你们对比了，显白效果是真的能打。"
- **On-Screen Text**: "室内光 vs 室外光"
- **Retention Tactic**: contrast

### Scene 5: 20s–30s (10s)
- **Shot Type**: medium-close-up
- **Template**: tiktok-product-on-face
- **Visual Description**: 手持口红管身，旋转出膏体，膏体表面丝绒质感清晰。手指在膏体上轻触展示质地。
- **Voiceover**: "质地是丝绒哑光的，上嘴完全不会拔干，薄涂厚涂都好看。"
- **On-Screen Text**: "丝绒哑光"
- **Retention Tactic**: pain-point

### Scene 6: 30s–40s (10s)
- **Shot Type**: split-screen
- **Template**: tiktok-before-after
- **Visual Description**: 分屏对照：左侧素唇，右侧涂口红后。两画面同时呈现 5 秒，然后右侧缓慢放大特写 5 秒。
- **Voiceover**: "左边素唇，右边涂后——这差别不用我多说了吧？"
- **On-Screen Text**: "Before → After"
- **Retention Tactic**: contrast

### Scene 7: 40s–50s (10s)
- **Shot Type**: medium-shot
- **Template**: talking-head
- **Visual Description**: 模特中景，背景虚化为米色。模特对镜头说话，自然手势。柔光左上方，暖色调。
- **Voiceover**: "这支口红我连续用了一周，吃饭喝水都不怎么掉色，性价比真的很高。"
- **On-Screen Text**: "持久不沾杯"
- **Retention Tactic**: pain-point

### Scene 8: 50s–55s (5s)
- **Shot Type**: reveal shot
- **Template**: tiktok-twist-ending
- **Visual Description**: 镜头缓慢后拉，模特对着镜子里的自己微笑，画面边缘逐渐露出化妆台上摆了至少 20 支口红的凌乱场景。
- **Voiceover**: "不过说实话，我化妆台上 20 支口红，最近只宠这一支。"
- **On-Screen Text**: "最爱一支"
- **Retention Tactic**: curiosity-question

### Scene 9: 55s–60s (5s)
- **Shot Type**: medium-shot
- **Template**: talking-head
- **Visual Description**: 模特中景正对镜头，右手比心手势，画面底部出现评论区引导文字。
- **Voiceover**: "你们觉得哪个色号最显白？评论区告诉我，下期测你们选的！"
- **On-Screen Text**: "评论区见"
- **Retention Tactic**: curiosity-question

## Seedance 2.0 Generation Segments (4 segments total)

### Segment 1/4: 0s–3s (3s)
- **Scenes Included**: 1
- **Template**: tiktok-hook-3s (hint: cosmetic-texture)
- **Continuity**: opening
- **Target Model**: seedance-2.0
- **Prompt Draft**:
  > 特写镜头：一支红色口红从画面右侧快速推入，停在金属台面中央，台面反光高对比。左上方柔光，背景纯黑。轻微 dolly-in 运镜约 1 秒入场，最后 2 秒静止聚焦口红膏体细节。暖色调。
- **Reference Files**: (none)
- **Manual Review Needed**: No

### Segment 2/4: 3s–20s (17s → adjusted to 15s max: 3s–18s)
- **Scenes Included**: 2, 3, 4
- **Template**: tiktok-product-on-face (hint: cosmetic-texture)
- **Continuity**: 保持上一段的左上方柔光、黑色背景高对比风格，暖色调不变。
- **Target Model**: seedance-2.0
- **Prompt Draft**:
  > 镜头1：模特半脸特写，素唇状态，左上方柔光，唇部占画面 1/3，镜头静止 3 秒。
  > 镜头2：同一角度，唇部已涂满红色丝绒哑光口红，唇纹细腻，颜色饱和，镜头静止 3 秒。
  > 镜头3：45 度角半身镜头，模特自然转头展示不同光线下的唇色，柔光左上方不变，镜头缓慢右移 4 秒。
  > 整体暖色调，背景虚化米色。保持与上一段画面的光照、色调、主体位置连续性，确保拼接后无明显跳变。
- **Reference Files**: (none)
- **Manual Review Needed**: No

### Segment 3/4: 18s–40s (22s → split: 18s–33s + 33s–40s)
- **Scenes Included**: 5, 6
- **Template**: tiktok-before-after (hint: cosmetic-texture)
- **Continuity**: 保持上一段的左上方柔光、模特半脸构图比例，暖色调不变。
- **Target Model**: seedance-2.0
- **Prompt Draft**:
  > 镜头1：手持口红管身特写，旋转出膏体，膏体表面丝绒质感清晰，手指轻触展示质地 5 秒，镜头静止。
  > 镜头2：分屏对照，左侧素唇右侧涂口红后，两画面同时呈现 5 秒。
  > 整体暖色调，左上方柔光，背景纯黑。保持与上一段画面的光照、色调、主体位置连续性，确保拼接后无明显跳变。
- **Reference Files**: (none)
- **Manual Review Needed**: No

### Segment 4/4: 40s–60s (20s → split: 40s–55s + 55s–60s)
- **Scenes Included**: 7, 8, 9
- **Template**: tiktok-twist-ending (hint: narrative-suspense)
- **Continuity**: 保持上一段的左上方柔光、暖色调，模特位置居中不变。
- **Target Model**: seedance-2.0
- **Prompt Draft**:
  > 镜头1：模特中景正对镜头说话，自然手势，背景虚化米色，镜头静止 8 秒。
  > 镜头2：镜头缓慢后拉，模特对镜微笑，画面边缘逐渐露出化妆台上凌乱的 20 支口红 5 秒。
  > 镜头3：模特中景正对镜头，右手比心手势 5 秒。
  > 左上方柔光，暖色调。保持与上一段画面的光照、色调、主体位置连续性，确保拼接后无明显跳变。
- **Reference Files**: (none)
- **Manual Review Needed**: No

> **Note**: The above is a simplified illustrative example. The actual implementation should adjust segment boundaries to meet the 15s hard constraint. The 60s video splits into 5 segments: `[0-3s][3-15s][15-27s][27-42s][42-55s][55-60s]` as defined in Step 3a's macro rhythm table, with each segment ≤15s.

## Call-to-Action (CTA)
- **Type**: comment_question
- **Timing**: 57s
- **Text**: "你们觉得哪个色号最显白？评论区告诉我，下期测你们选的！"

## Hashtags
- **Trending**: #beautytok #makeupreview #口红测评
- **Niche**: #lipstickreview #kbeauty #显白口红
- **Branded**: (none)
- **Strategy**: 热门标签覆盖FYP流量池，垂类标签精准触达美妆兴趣用户

## Cover / Thumbnail
- **Image Prompt**: 红色口红特写，金属台面反光，黑背景高对比，暖色调
- **Overlay Text**: "黄皮救星"
- **Color Scheme**: high-contrast

## Video Title
**黄皮必看！这支口红让我直接白了两个度｜显白口红真实测评**

## Music & Sound
- **Mood**: upbeat
- **BPM Range**: 120-130
- **License Safe**: Yes (TikTok Commercial Library)
- **Library Keywords**: trending tiktok sound, upbeat corporate instrumental

## Compliance Check
- **Originality**: OK
- **Watermark Warning**: (none)
- **Banned Words Detected**: (none)
- **CRP Eligible**: Yes
- **Music Copyright**: 建议使用 TikTok Commercial Music Library 中的曲目，避免外链 Spotify/Apple Music/SoundCloud 等第三方平台音乐。
- **Notes**: (none)

## Cross-Platform Adaptation Tips

### Instagram Reels
此脚本为 60s，在 IG Reels 90s 限制内可直接发布。建议将音乐替换为 Meta Sound Collection 中的 upbeat pop 曲目（IG 版权审核更严格）。Hashtag 精简为 3-5 个（#beautyreels #lipstickreview #makeuptutorial），IG 受众偏好更精致的美学呈现——封面建议使用高对比唇部特写并叠加柔和滤镜。

### YouTube Shorts
YT Shorts 限制 60s，此脚本可直接适配。标题建议 SEO 优化为 "黄皮显白口红测评｜这支红管让我白了两个度！" 以提升 YouTube 搜索触达。描述区添加时间戳章节（0:00 钩子 / 0:03 素唇对比 / 0:40 一周实测 / 0:55 CTA）和订阅引导。Hashtag 使用 #Shorts #口红测评 #beautytips 即可，YT 的 hashtag 权重低于 TikTok。

---
本次使用 skill：tiktok-script-gen v1
KPI 表版本：v1
合规规则版本：v1
template_registry：v1
````

---

### 5.3 JSON Output Example (Same Script, Condensed)

```json
{
  "meta": {
    "duration_sec": 60,
    "duration_meets_crp": true,
    "language": "zh",
    "niche": "beauty",
    "monetization_goal": "crp",
    "estimated_kpi": {
      "expected_completion_rate": "48-60%",
      "expected_rpm_usd": "0.5-0.8"
    },
    "total_seedance_segments": 5
  },
  "hook_3s": {
    "visual": "黑背景，一支红色口红从画面右侧快速推入，停在金属台面中央",
    "voiceover": "黄皮涂这支口红，直接白两个度？",
    "on_screen_text": "黄皮救星",
    "retention_tactic": "curiosity-question",
    "variants": [
      {
        "visual": "黑白滤镜下模特素颜半脸，手指轻触屏幕，画面切为彩色",
        "voiceover": "同一张脸，涂这支口红前后差别有多大？",
        "on_screen_text": "差距惊人",
        "retention_tactic": "contrast"
      }
    ]
  },
  "scene_list": [
    {
      "id": 1, "start_sec": 0, "end_sec": 3, "shot_type": "close-up",
      "template_id": "tiktok-hook-3s",
      "visual_description": "黑背景，红色口红从画面右侧快速推入",
      "voiceover": "黄皮涂这支口红，直接白两个度？",
      "on_screen_text": "黄皮救星", "retention_tactic": "curiosity-question"
    }
  ],
  "seedance_segments": [
    {
      "segment_id": 1, "start_sec": 0, "end_sec": 3, "duration_sec": 3,
      "scene_ids": [1], "template_id": "tiktok-hook-3s", "template_hint": "cosmetic-texture",
      "continuity_hint": "opening",
      "prompt_draft": "特写镜头：一支红色口红从画面右侧快速推入，停在金属台面中央...",
      "target_model": "seedance-2.0", "reference_files": [], "requires_manual_review": false
    }
  ],
  "cta": { "type": "comment_question", "text": "你们觉得哪个色号最显白？评论区告诉我！", "timing_sec": 57 },
  "hashtags": {
    "trending": ["#beautytok", "#makeupreview"],
    "niche": ["#lipstickreview", "#kbeauty"],
    "branded": [],
    "rationale": "热门标签覆盖FYP，垂类标签精准触达"
  },
  "cover": {
    "image_prompt": "红色口红特写，金属台面反光，黑背景高对比",
    "overlay_text": "黄皮救星",
    "color_scheme": "high-contrast"
  },
  "title": "黄皮必看！这支口红让我直接白了两个度｜显白口红真实测评",
  "music_suggestion": {
    "mood": "upbeat", "bpm_range": "120-130", "license_safe": true,
    "tiktok_library_keywords": ["trending tiktok sound", "upbeat corporate instrumental"]
  },
  "compliance_check": {
    "originality_ok": true, "watermark_warning": null, "banned_words_hit": [],
    "duration_meets_crp": true,
    "music_copyright_advice": "建议使用 TikTok Commercial Music Library 中的曲目，避免外链 Spotify/Apple Music/SoundCloud 等第三方平台音乐。",
    "notes": []
  },
  "cross_platform_tips": {
    "ig_reels": "适配 IG Reels：控制在 90s 内，使用 Meta Sound Collection 音乐。Hashtag 精简为 3-5 个。封面建议使用高对比唇部特写并叠加柔和滤镜。",
    "yt_shorts": "适配 YouTube Shorts：时长限制 60s，此脚本可直接适配。标题 SEO 优化，描述区添加时间戳章节和订阅引导。Hashtag 使用 #Shorts #口红测评 #beautytips。"
  },
  "_schema_validation": { "passed": true, "errors": [] }
}
```

---

### 5.4 Field-Level Constraints Summary

| Field Path | Constraint | AC |
|---|---|---|
| `meta.duration_meets_crp` | boolean; true iff duration_sec >= 60 | AC1 |
| `meta.estimated_kpi.expected_rpm_usd` | "0-0" if duration_sec < 60 | AC2 |
| All copy fields | Must use `language` parameter value | AC3 |
| `scene_list[0].template_id` | Must be "tiktok-hook-3s" | AC4 |
| `hook_3s.variants` | length = ab_variants (capped at 5) | AC5 |
| Output format | Respect `output_format` parameter | AC6 |
| `meta.estimated_kpi.*` | Non-empty range strings | AC7 |
| `compliance_check.banned_words_hit` | Non-empty if banned words detected | AC8 |
| `compliance_check.music_copyright_advice` | Always present, non-empty string | AC9 |
| `seedance_segments[*].continuity_hint` | "opening" for seg[0]; cross-segment hints for others | AC10 |
| `seedance_segments[*].prompt_draft` | Chinese, 50-150 chars, no 14-word blacklist, 1 action + 1 camera move | AC10 |
| `hashtags.trending` / `music_suggestion` | Include trending_inject values if provided | AC11 |
| Skill activation | Slash command + keyword triggers both work | AC12 |
| `cross_platform_tips.ig_reels` / `yt_shorts` | Both non-empty strings | AC13 |
| `scene_list[*].template_id` | Within 18 templates (13 reused + 5 new) | AC14 |
| `seedance_segments[*].template_hint` | Within video_prompt_gen's 13 templates | AC14 |
| JSON output | Passes output-schema.json validation | AC15 |
| `seedance_segments[*].duration_sec` | <= 15; sum = meta.duration_sec (+/- 1s) | AC16 |
| Segment boundaries | Not mid-voiceover-sentence; not mid-camera-move | AC16 |

---

## 6. Built-in Rule Sets (Index)

> **Cycle 5 (final)**: All reference files are now created and loadable. All 8 steps are fully implemented. All listed file paths are valid. This skill is at v1 production readiness.

Rule set files located under `references/`:

| File | Purpose | Cycle | Status |
|---|---|---|---|
| `template-registry.md` | 18-template catalog (13 reused + 5 new) + downstream_mapping + drift_check | Cycle 2 | Created |
| `compliance-rules.md` | Banned word blacklist + watermark detection + music copyright rules | Cycle 2 | Created |
| `kpi-table.md` | Completion rate & RPM estimation tables (10 niche x 6 duration buckets) | Cycle 2 | Created |
| `output-schema.json` | JSON Schema for output validation (draft-07) | Cycle 2 | Created |
| `seedance-mirror-rules.md` | R1/R3/R4/R6 mirror rules + 1-rewrite strategy | Cycle 2 | Created |
| `hook-strategies.md` | Niche x audience x monetization_goal → hook type decision table | Cycle 3 | Created |
| `monetization-playbooks.md` | 5 monetization goal CTA styles + copy templates | Cycle 3 | Created |
| `templates/tiktok-hook-3s.md` | 0-3s hook shot template | Cycle 4 | Created |
| `templates/tiktok-product-on-face.md` | Product-on-face close-up template | Cycle 4 | Created |
| `templates/tiktok-before-after.md` | Before/after comparison template | Cycle 4 | Created |
| `templates/tiktok-beat-sync.md` | Beat-sync rapid cut template | Cycle 4 | Created |
| `templates/tiktok-twist-ending.md` | Twist ending template | Cycle 4 | Created |

---

## 7. Integration with video_prompt_gen

### 7.1 Integration Mode: Plan C (Loose Coupling)

tiktok-script-gen does **NOT** directly invoke video_prompt_gen. Instead, it produces structured `seedance_segments[].prompt_draft` strings that are pre-validated against video_prompt_gen's R1/R3/R4/R6 rules. The user or downstream agent copies the `prompt_draft` value and feeds it to video_prompt_gen in a separate conversation.

### 7.2 Key Contract: seedance_segments[]

Each `seedance_segments[i]` element contains everything video_prompt_gen needs for fast-track processing:

| Field | Purpose for video_prompt_gen |
|---|---|
| `prompt_draft` | Pre-filled Seedance 2.0 prompt in Chinese (50-150 chars). Already passed R1/R3/R4/R6 mirror self-check → video_prompt_gen can use "直接给" / fast track without triggering rewrites. |
| `template_hint` | Maps to one of video_prompt_gen's 13 known template IDs (via `template-registry.md` downstream_mapping table). Tells video_prompt_gen which template's parameter defaults to use. |
| `target_model` | Always `"seedance-2.0"` — tells video_prompt_gen to use Seedance 2.0 routing, Chinese-native prompt conventions, and the @引用 system. |
| `continuity_hint` | Optional metadata — if the user is generating all segments sequentially, this hint helps maintain visual consistency across segments. Passed as contextual note, not part of the prompt. |
| `reference_files` | User-fillable `@图片1` / `@视频1` references. Empty by default; user adds before feeding to video_prompt_gen. |
| `requires_manual_review` | If `true`, video_prompt_gen should flag the prompt for human review rather than auto-processing. |

### 7.3 User Workflow

The typical user workflow for generating a full TikTok video:

1. **Step 1**: Run tiktok-script-gen (this skill) → get a complete script with N `seedance_segments`.
2. **Step 2**: For each segment i (1 to N):
   - Copy `seedance_segments[i].prompt_draft`
   - In a NEW conversation, invoke video_prompt_gen
   - Say "直接给" (fast track) or paste the prompt_draft directly
   - video_prompt_gen routes to Seedance 2.0 and applies the `template_hint` defaults
3. **Step 3**: Collect all N video outputs
4. **Step 4**: Stitch them together in post-production, using `continuity_hint` values to guide cross-fade or match-cut transitions

### 7.4 template_hint Mapping via downstream_mapping

The 5 TikTok-specific templates (`tiktok-hook-3s`, etc.) are internal orchestration units. They MUST NOT appear as direct `template_hint` values because video_prompt_gen only recognizes its 13 built-in template IDs.

The `template-registry.md` downstream_mapping table provides the bridge:

```
tiktok-hook-3s          → cosmetic-texture | food-closeup | product-rotation | talking-head
tiktok-product-on-face  → cosmetic-texture | clothing-tryout
tiktok-before-after     → cosmetic-texture | appliance-usage
tiktok-beat-sync        → product-rotation | orbit-shot
tiktok-twist-ending     → narrative-suspense | talking-head
```

Selection logic: for each mapping with multiple targets, choose based on the video's `niche` (see Step 5.4.1 for the full decision table).

### 7.5 Failure Handling

If video_prompt_gen's self-check detects issues despite our mirror rules:

1. **R1/R3/R4/R6 violation**: video_prompt_gen auto-rewrites once. If our `requires_manual_review` was already `true`, it may append its own review flag.
2. **R2/R5/R7/R8 advisories**: video_prompt_gen appends warnings but does not rewrite. These are acceptable — they are soft-level suggestions.
3. **Model routing mismatch**: If the user wants a model other than Seedance 2.0, the `target_model` field serves as a recommendation only; the user can override in video_prompt_gen.

### 7.6 Drift Protection

video_prompt_gen's template list and rules may evolve independently. Drift protection mechanisms:

1. **CI drift check** in `template-registry.md` (§CI Drift Check): Detects when video_prompt_gen adds/removes templates, ensuring our `reused_from_video_prompt_gen` list stays synchronized.
2. **Mirror rule version tracking** in `seedance-mirror-rules.md`: Stores `sync_source` and `last_synced` date. If video_prompt_gen's `principles.md` updates R1/R3/R4/R6, a manual sync is required.
3. **Recommended maintenance**: Quarterly manual diff of our mirror rules against video_prompt_gen's principles.md.

### 7.7 Segment Count Advisory

For videos with many seedance_segments (>8, typically >120s content), append a practical note in the output:

```
> 本脚本共 N 个 Seedance 生成段，需要 N 次 Seedance 2.0 生成 + 后期拼接。
> 建议流程：按 segment 顺序逐个生成，每段参考 continuity_hint 保持视觉一致性。
```

This keeps the user aware of the practical generation effort (R13 mitigation).

---

## 8. End-of-Output Archive Note

Every output from tiktok-script-gen must end with the following signature block:

```
---
本次使用 skill：tiktok-script-gen v1
KPI 表版本：v1
合规规则版本：v1
template_registry：v1
```

---

## 9. Self-Verification Checklist

After generating a complete script, Claude MUST run through this 16-item checklist to verify the output before presenting it to the user. This is an in-prompt self-check, not a runtime test suite.

### AC1: CRP Eligibility Flag
- [ ] `compliance_check.duration_meets_crp` is present and is a boolean. (AC1)
- [ ] `meta.duration_meets_crp` equals `compliance_check.duration_meets_crp`. (AC1)
- [ ] If `duration_sec >= 60` → both are `true`. (AC1)

### AC2: Short Duration Warning
- [ ] If `duration_sec < 60` → `duration_meets_crp = false` AND `compliance_check.notes` contains the CRP ineligibility warning text ("时长不足 60s..."). (AC2)
- [ ] If `duration_sec < 60` → `expected_rpm_usd = "0-0"`. (AC2)
- [ ] Output is NOT blocked by short duration — full script is delivered regardless. (AC2)

### AC3: Language Unification
- [ ] All copy fields (voiceover, on_screen_text, cta.text, title, hashtags.rationale) match the `language` parameter. (AC3)
- [ ] If `language=zh` → all copy is Chinese. (AC3)
- [ ] If `language=en` (default) → all copy is English. (AC3)
- [ ] `prompt_draft` is in Chinese regardless (Seedance 2.0 native language). (AC3)

### AC4: TikTok-Specific Templates
- [ ] `scene_list[0].template_id == "tiktok-hook-3s"` AND `scene_list[0].start_sec == 0`. (AC4)
- [ ] At least one scene uses one of the 5 TikTok-specific template IDs (hook-3s / product-on-face / before-after / beat-sync / twist-ending). (AC4)

### AC5: A/B Variant Constraints
- [ ] `hook_3s.variants.length == ab_variants` (default=2, max=5). (AC5)
- [ ] If `ab_variants > 5` → truncated to 5 with a note. (AC5)
- [ ] Each variant has a different `retention_tactic` (when `ab_variants <= 5`). (AC5)

### AC6: Output Format Branching
- [ ] `output_format="both"` (default) → both Markdown and JSON are present. (AC6)
- [ ] `output_format="markdown"` → only Markdown, no JSON code block. (AC6)
- [ ] `output_format="json"` → only JSON, no Markdown sections. (AC6)

### AC7: KPI Interval Validity
- [ ] `estimated_kpi.expected_completion_rate` is a non-empty range string (format: "X%-Y%"). (AC7)
- [ ] `estimated_kpi.expected_rpm_usd` is a non-empty range string (format: "X-Y"), or "0-0" for <60s. (AC7)
- [ ] Values change with different niche/duration inputs. (AC7)
- [ ] Monotonicity: for the same niche, RPM upper bound at duration >=75s is >= RPM upper bound at duration=60s (pre-verified in kpi-table.md). (AC7)

### AC8: Banned Word & Watermark Detection
- [ ] If any output field contains a banned word from compliance-rules.md §2 → `banned_words_hit[]` is non-empty with `{word, location, severity}` records. (AC8)
- [ ] If any visual/on-screen field contains a watermark keyword → `watermark_warning` is a non-null string. (AC8)
- [ ] If neither is detected → `banned_words_hit == []` AND `watermark_warning == null`. (AC8)
- [ ] Even with hits detected, the full script is still delivered (non-blocking). (AC8)

### AC9: Music Copyright Safety
- [ ] `compliance_check.music_copyright_advice` is ALWAYS present as a non-empty string. (AC9)
- [ ] `music_suggestion.tiktok_library_keywords` has `length >= 1`. (AC9)
- [ ] `music_suggestion.license_safe == true`. (AC9)
- [ ] No Spotify/Apple Music/SoundCloud links in any music field. (AC9)
- [ ] No direct commercial song naming (artist + title). (AC9)

### AC10: Seedance Segment Integrity
- [ ] `seedance_segments[]` array exists with `length >= 1`. (AC10)
- [ ] `seedance_segments[0].continuity_hint == "opening"`. (AC10)
- [ ] All non-first segments have a non-empty `continuity_hint` containing lighting/position/color references. (AC10)
- [ ] Every `prompt_draft` is in Chinese (target_model=seedance-2.0). (AC10)
- [ ] Every `prompt_draft` is 50-150 Chinese characters (excluding continuity appended sentence). (AC10)
- [ ] No `prompt_draft` contains any of the 14 banned abstract adjectives (史诗/震撼/绝美/电影感/大师之作/完美/高质量/超高清/8K/极致/惊艳/史无前例/无与伦比/终极). (AC10)
- [ ] Each shot block within `prompt_draft` has exactly 1 action + 1 camera movement. (AC10)
- [ ] `scene_ids[]` links `seedance_segments` to `scene_list` — every scene ID is accounted for. (AC10)

### AC11: Trending Injection
- [ ] If `trending_inject.hashtags` was provided → `hashtags.trending` includes those values. (AC11)
- [ ] If `trending_inject.sounds` was provided → `music_suggestion.tiktok_library_keywords` includes those values. (AC11)
- [ ] If `trending_inject` was not provided → both fields use auto-generated experience-based values. (AC11)

### AC12: Skill Activation (Design-Time Check)
- [ ] Slash command `/tiktok-script` is registered in frontmatter. (AC12)
- [ ] Trigger keywords (Chinese + English) are present in the `description` field. (AC12)
- [ ] Exit/mis-trigger response templates are defined in §1.2. (AC12)

### AC13: Cross-Platform Tips
- [ ] `cross_platform_tips.ig_reels` is a non-empty string (2-4 sentences). (AC13)
- [ ] `cross_platform_tips.yt_shorts` is a non-empty string (2-4 sentences). (AC13)
- [ ] IG Reels tip mentions 90s limit, Meta Sound Collection, hashtag differences. (AC13)
- [ ] YT Shorts tip mentions 60s limit, title SEO, description strategy, #Shorts tag. (AC13)

### AC14: Template Domain Validation
- [ ] Every `scene_list[*].template_id` is one of the 18 known templates (13 reused + 5 new). (AC14)
- [ ] Every `seedance_segments[*].template_hint` is one of video_prompt_gen's 13 known template IDs. (AC14)

### AC15: JSON Schema Compliance
- [ ] JSON output contains all 11 top-level required fields. (AC15)
- [ ] JSON output passes all nested required constraints (estimated_kpi, variants, cta.type/text/timing_sec, all compliance sub-fields). (AC15)
- [ ] JSON output type constraints are satisfied (integers, booleans, arrays, strings as specified). (AC15)
- [ ] `_schema_validation.passed == true`. (AC15)

### AC16: 15s Segmentation Hard Constraint
- [ ] Every `seedance_segments[i].duration_sec <= 15`. (AC16)
- [ ] `sum(seedance_segments[*].duration_sec) == meta.duration_sec` (tolerance +/- 1s for rounding). (AC16)
- [ ] Segment boundaries do NOT cut in the middle of voiceover sentences — check each boundary against the corresponding voiceover text. (AC16)
- [ ] Segment boundaries do NOT cut during active camera movements — check each boundary against the shot descriptions. (AC16)
- [ ] All segment `start_sec`/`end_sec` are contiguous with no gaps or overlaps. (AC16)
- [ ] `seedance_segments[0].start_sec == 0`. (AC16)

---

### How to Use This Checklist

After generating a complete script through Steps 1-8, run through each AC item sequentially. For each item:

- **PASS** → mark `[x]` and continue.
- **FAIL** → fix the issue in the relevant step and re-check. Do NOT present the output to the user until all 16 items pass (or the failing items are explicitly in non-blocking categories: AC8 banned word hits do not block, AC2 short duration does not block).

This checklist is executed in-prompt by Claude as part of the quality gate before final output serialization. It is NOT a separate tool or test runner.
