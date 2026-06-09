# Compliance Rules — tiktok-script-gen v1

> Single source of truth for all compliance checks performed by the skill.
> Used in Step 6 of the reasoning workflow: pre-check (before generation) and final check (after generation).
>
> **version**: v1
> **valid_until**: 2026-12-31
> **source**: TikTok Community Guidelines (US) + industry best practices for short-form video ads

---

## 1. Duration Warning Rules (Step 6 pre-check)

These are evaluated against the `duration_sec` parameter after Step 1 parsing.
Warnings are appended to `compliance_check.notes`, never blocking output.

### Rule 1.1: Short Duration (CRP ineligible)

**Condition**: `duration_sec < 60`

**Action**:
- Set `compliance_check.duration_meets_crp = false`
- Set `meta.duration_meets_crp = false`
- Append to `compliance_check.notes`:

```
"时长不足 60s（实际 {N}s），不计入 TikTok CRP（Creator Rewards Program）收益。如需 CRP 准入，请将 duration_sec 调整为 ≥60。"
```

### Rule 1.2: Marginal Duration (CRP minimum)

**Condition**: `duration_sec >= 60 AND duration_sec < 75`

**Action**:
- Append to `compliance_check.notes`:

```
"时长 {N}s 满足 CRP 准入下限，但 75s+ 通常 RPM 区间更优，建议考虑加长。"
```

### Rule 1.3: Long Duration (completion rate risk)

**Condition**: `duration_sec > 180`

**Action**:
- Append to `compliance_check.notes`:

```
"时长 >180s 完播率显著下降，建议拆分为系列短视频。"
```

---

## 2. Banned Word Blacklist (Step 6 pre-check + final check)

These words/phrases trigger `compliance_check.banned_words_hit[]` when detected in any output field
(voiceover, on_screen_text, cta.text, title, visual_description, prompt_draft).

Each detected hit records: `{ "word": "...", "location": "...", "severity": "high" }`.

The skill auto-rewrites to avoid these words during generation. If a rewrite fails,
the hit is recorded but output is NOT blocked.

### 2.1 Chinese Banned Words (10 required, extendable to 50+)

| # | Banned Word/Phrase | Category | Rationale |
|---|---|---|---|
| 1 | 最便宜 | 绝对化用语 | Implied price superiority — violates advertising laws |
| 2 | 全网最低 | 绝对化用语 | Platform-wide price claim — unverifiable |
| 3 | 100% 有效 | 绝对化用语 | Absolute efficacy claim — misleading |
| 4 | 绝对治愈 | 医疗夸大 | Medical cure guarantee — prohibited |
| 5 | 一夜变白 | 医疗夸大 | Implied instant medical result — prohibited |
| 6 | 神药 | 医疗夸大 | "Miracle drug" — prohibited health claim |
| 7 | 包治百病 | 医疗夸大 | "Cures all diseases" — prohibited |
| 8 | 永久去除 | 绝对化用语 | "Permanent removal" — unverifiable claim |
| 9 | 史上最强 | 绝对化用语 | "Strongest in history" — superlative |
| 10 | 国家认证 | 权威滥用 | "Nationally certified" — requires verifiable evidence; hit unless user provides proof in `brand_constraint` |

### 2.2 English Banned Words (10 required, extendable to 50+)

| # | Banned Word/Phrase | Category | Rationale |
|---|---|---|---|
| 1 | cheapest ever | 绝对化用语 | Absolute price claim — unverifiable |
| 2 | 100% guaranteed | 绝对化用语 | Unconditional guarantee — misleading |
| 3 | miracle cure | 医疗夸大 | "Miracle cure" — prohibited health claim |
| 4 | instantly heal | 医疗夸大 | "Instant healing" — impossible claim |
| 5 | doctors hate this | 权威滥用 | Manufactured authority — clickbait pattern |
| 6 | lose 10 pounds overnight | 医疗夸大 | Specific unverified weight loss claim |
| 7 | fda approved | 权威滥用 | FDA approval claim — hit unless verifiable evidence provided |
| 8 | never seen before | 绝对化用语 | "Unprecedented" — unverifiable novelty claim |
| 9 | world's best | 绝对化用语 | Global superiority claim — unverifiable |
| 10 | absolutely free | 价格夸大 | "Absolutely free" — hit unless conditions explicitly stated |

### 2.3 Matching Rules

- Case-insensitive matching for English terms.
- Substring matching for Chinese terms (e.g., "最便宜" matches "全网最便宜的...").
- Whole-word matching for English terms to avoid false positives.
- Words in `brand_constraint.must_say` are excluded from banned word scanning.
- Words in `brand_constraint.banned` are treated as additional banned words (merged into the blacklist for this session only).

---

## 3. Third-Party Watermark Detection (Step 6 final check)

Scanned fields: `visual_description`, `on_screen_text`, `prompt_draft`.

When any keyword matches, populate `compliance_check.watermark_warning` with the template message below.
Only one warning is emitted (first match), not one per keyword.

### 3.1 Detection Keywords

```
- instagram logo / instagram watermark / ig watermark / from instagram
- youtube watermark / youtube logo / yt watermark / shorts watermark / from youtube
- facebook logo / meta watermark / from facebook
- douyin watermark / douyin logo / 抖音水印 / from douyin
- xiaohongshu watermark / rednote watermark / 小红书水印 / from xiaohongshu
- snapchat logo / snap watermark / from snapchat
- bilibili logo / bilibili watermark / B站水印 / from bilibili
```

### 3.2 @username Pattern Detection

Any text matching `@username from <other platform>` (where platform is not TikTok) triggers the warning.
Pattern: `@[\w.]+.*(instagram|youtube|facebook|douyin|xiaohongshu|snapchat|bilibili|weibo|wechat|twitter|x\.com)`

### 3.3 Warning Template

```
"检测到 {keyword}，TikTok 不分成第三方水印素材，请确保最终视频不含此元素。"
```

English equivalent:
```
"Detected {keyword}. TikTok does not monetize content with third-party watermarks. Ensure the final video is free of this element."
```

---

## 4. Music Copyright Safety Rules (Step 6 final check)

### 4.1 Mandatory Field Rule

`music_suggestion.tiktok_library_keywords` array MUST have `length >= 1` (AC9).
If empty, populate with at least one keyword from the recommended pool (see below).

### 4.2 Prohibited Link Patterns

The following URL/link patterns are banned in `music_suggestion` any field:

```
- spotify.com / open.spotify.com / play.spotify.com
- music.apple.com / itunes.apple.com
- soundcloud.com / on.soundcloud.com
- youtube.com/watch (music-only links)
- amazon.com/music
- tidal.com
- deezer.com
```

### 4.3 Prohibited Commercial Song References

Do NOT directly name commercial artists + song titles (e.g., "Taylor Swift - Shake It Off").
Instead, describe the genre/style: "upbeat pop similar to early 2010s dance-pop".

### 4.4 Recommended Keyword Pool

`music_copyright_advice` default text:
```
"建议使用 TikTok Commercial Music Library 中的曲目，避免外链 Spotify/Apple Music/SoundCloud 等第三方平台音乐。"
```

Recommended `tiktok_library_keywords` pool (pick 1-3 most relevant):
- `trending tiktok sound`
- `tiktok original audio`
- `commercial library upbeat pop`
- `royalty-free lofi`
- `tiktok viral beat`
- `copyright-safe electronic`
- `creator music acoustic`
- `tiktok commercial hip-hop`
- `upbeat corporate instrumental`
- `ambient background corporate`

---

## 5. Rule Triggering Order

Compliance checks execute in a strict 4-phase pipeline within Step 6:

```
Phase 1: PRE-CHECK (before copy generation)
  ├── Scan all user-provided inputs (brand_constraint.banned + trending_inject.sounds) against banned word blacklist (§2)
  ├── Evaluate duration thresholds (§1) → set duration_meets_crp + notes
  └── Flag any banned words found in user inputs → carry forward to generation phase

Phase 2: COPY GENERATION (Step 4)
  ├── Generate hook/voiceover/on_screen_text/CTA/title with banned word avoidance
  └── Auto-rewrite any generated text that matches banned word patterns

Phase 3: FINAL CHECK (after all outputs generated)
  ├── Scan all output fields against banned word blacklist (§2) → populate banned_words_hit[]
  ├── Scan visual_description + on_screen_text against watermark keywords (§3) → populate watermark_warning
  ├── Validate music_suggestion against copyright rules (§4) → populate music_copyright_advice
  ├── Confirm duration_meets_crp boolean is set in both meta and compliance_check
  └── Populate compliance_check.notes with all accumulated warnings

Phase 4: SERIALIZATION (Step 8)
  ├── Embed compliance_check object into Markdown output
  └── Embed compliance_check object into JSON output validated against output-schema.json
```

### Non-Blocking Principle

Compliance checks NEVER block output. They only annotate and warn.
Rationale: per Q4/D6 decision, the user is the final reviewer.
Even if `banned_words_hit` is non-empty or `watermark_warning` is populated,
the full script is still delivered for the user to decide.

### Edge Cases

- **Double-match**: If a word appears in both the banned list AND watermark list, record it in both.
- **Empty output**: If no banned/watermark hits are found, `banned_words_hit = []` and `watermark_warning = null`.
- **Trending sound violation**: If `trending_inject.sounds` contains a Spotify/Apple Music link, auto-rewrite by extracting the genre keyword and populating `compliance_check.music_copyright_advice` with a warning.
