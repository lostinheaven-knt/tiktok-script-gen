# Hook Strategies -- tiktok-script-gen v1

> **version**: v1
> **purpose**: Given `niche × target_audience.age × monetization_goal`, determine the optimal hook type and generate hook 3-piece sets (visual / voiceover / on_screen_text) plus A/B variants with distinct retention tactics.
> **used_in**: SKILL.md Step 2 (Audience & Hook Strategy Selection)
> **references**: tiktok-video-gen-report.md §2 (audience targeting), §4 (creation workflow)

---

## 1. Hook Type Definitions (5 types)

Each hook type maps to exactly one `retention_tactic`. The type name is used for the decision table lookup; the tactics drive variant generation.

| # | Hook Type | retention_tactic | Core Technique | Best For | Duration |
|---|---|---|---|---|---|
| 1 | **visual-shock** | `visual-shock` | High-contrast visuals, fast camera entry, unexpected color/composition, extreme close-up | Grabbing attention in noisy FYP feed; beauty/food/comedy niches | 2-3s |
| 2 | **curiosity-question** | `curiosity-question` | Pose a counter-intuitive question, leave information incomplete, create "need to know" gap | edu/tech niches; live_funnel goals; driving comments | 3s |
| 3 | **pain-point** | `pain-point` | Show relatable daily frustration scene the viewer instantly recognizes | shop niche; solving a problem the product addresses | 2-3s |
| 4 | **contrast** | `contrast` | Before/after reveal, expectation vs. reality, split-screen comparison | beauty/fitness niches; brand_deal goals; demonstrating transformation | 3s |
| 5 | **number-promise** | `number-promise` | "3 methods", "5 mistakes", "90% of people don't know", specific quantified claim | edu/finance/tech niches; awareness goals; information-dense content | 3s |

---

## 2. Hook Decision Table

Lookup key: `{niche} × {audience_age} × {monetization_goal} → {primary_hook, secondary_hook}`.

The table below provides the **primary hook type** for every combination. The **secondary hook** is a fallback used for A/B variant generation (variant 2+). When `ab_variants=1`, only the primary is used.

### 2.1 CRP (Creator Rewards Program) -- maximize completion rate + interaction

| Niche | 13-17 | 18-24 | 25-34 | 35+ |
|---|---|---|---|---|
| **beauty** | visual-shock / curiosity-question | visual-shock / curiosity-question | contrast / number-promise | contrast / number-promise |
| **tech** | visual-shock / number-promise | number-promise / curiosity-question | number-promise / pain-point | number-promise / pain-point |
| **edu** | curiosity-question / number-promise | curiosity-question / number-promise | number-promise / curiosity-question | number-promise / contrast |
| **food** | visual-shock / number-promise | visual-shock / curiosity-question | curiosity-question / contrast | pain-point / number-promise |
| **travel** | visual-shock / curiosity-question | visual-shock / curiosity-question | curiosity-question / number-promise | contrast / curiosity-question |
| **fitness** | visual-shock / contrast | contrast / number-promise | contrast / pain-point | pain-point / number-promise |
| **finance** | — (age 18+ for finance content) | number-promise / curiosity-question | number-promise / pain-point | pain-point / number-promise |
| **comedy** | visual-shock / contrast | visual-shock / curiosity-question | contrast / curiosity-question | contrast / pain-point |
| **shop** | visual-shock / contrast | pain-point / visual-shock | pain-point / number-promise | pain-point / contrast |
| **other** | visual-shock / curiosity-question | curiosity-question / visual-shock | number-promise / pain-point | pain-point / number-promise |

**Key insights for CRP**:
- Young (13-24): visual-shock wins on FYP scroll-stop. Curiosity-question as secondary drives comments.
- Adult (25-34): shifts to contrast/number-promise -- viewers stay for value, not just visuals.
- Mature (35+): pain-point (relatable problems) and number-promise (credible structure) dominate.
- **edu niche is inversion**: young audiences prefer curiosity questions, older prefer structured number-promises.
- **comedy niche**: visual-shock dominates all ages -- the first 1 second must trigger emotion.
- **finance**: number-promise across all ages -- data/stats are the universal hook for money topics.

### 2.2 Shop (TikTok Shop -- maximize conversion rate)

| Niche | 13-17 | 18-24 | 25-34 | 35+ |
|---|---|---|---|---|
| **beauty** | visual-shock / pain-point | pain-point / contrast | pain-point / contrast | contrast / number-promise |
| **tech** | number-promise / pain-point | pain-point / visual-shock | pain-point / number-promise | pain-point / contrast |
| **edu** | curiosity-question / number-promise | pain-point / curiosity-question | pain-point / number-promise | number-promise / pain-point |
| **food** | visual-shock / pain-point | pain-point / visual-shock | pain-point / contrast | pain-point / number-promise |
| **travel** | visual-shock / contrast | pain-point / visual-shock | pain-point / curiosity-question | contrast / pain-point |
| **fitness** | contrast / visual-shock | pain-point / contrast | pain-point / contrast | pain-point / number-promise |
| **finance** | — | number-promise / pain-point | pain-point / number-promise | number-promise / pain-point |
| **comedy** | visual-shock / pain-point | pain-point / contrast | contrast / pain-point | pain-point / contrast |
| **shop** | pain-point / visual-shock | pain-point / contrast | pain-point / number-promise | pain-point / contrast |
| **other** | pain-point / visual-shock | pain-point / contrast | pain-point / number-promise | pain-point / contrast |

**Key insights for Shop**:
- **pain-point dominates**: product solves a problem → show the problem first. Pain-point is primary across 80%+ of combinations.
- Product must appear in the hook shot (within first 3s) -- no prolonged suspense for shop content.
- Young audiences (13-17): visual-shock as secondary (grab attention with product aesthetics before pain).
- Older audiences (35+): number-promise as secondary (quantified benefit claims build trust).

### 2.3 Brand Deal (brand sponsorship -- maximize brand exposure + favorability)

| Niche | 13-17 | 18-24 | 25-34 | 35+ |
|---|---|---|---|---|
| **beauty** | visual-shock / contrast | contrast / curiosity-question | contrast / number-promise | contrast / pain-point |
| **tech** | number-promise / visual-shock | contrast / number-promise | contrast / pain-point | number-promise / contrast |
| **edu** | curiosity-question / contrast | contrast / number-promise | number-promise / curiosity-question | number-promise / contrast |
| **food** | visual-shock / contrast | contrast / visual-shock | contrast / pain-point | contrast / number-promise |
| **travel** | visual-shock / contrast | contrast / curiosity-question | contrast / number-promise | contrast / curiosity-question |
| **fitness** | contrast / visual-shock | contrast / pain-point | contrast / number-promise | contrast / pain-point |
| **finance** | — | number-promise / contrast | contrast / number-promise | contrast / number-promise |
| **comedy** | visual-shock / contrast | contrast / visual-shock | contrast / curiosity-question | contrast / pain-point |
| **shop** | visual-shock / contrast | contrast / visual-shock | contrast / number-promise | contrast / pain-point |
| **other** | visual-shock / contrast | contrast / visual-shock | contrast / number-promise | contrast / pain-point |

**Key insights for Brand Deal**:
- **contrast dominates**: brand stories work best with before/after or expectation/reality structures.
- Brand logo must not dominate the hook shot (users distrust upfront branding). Natural appearance in scene.
- Young: visual-shock secondary (product aesthetic matters). Older: pain-point/number-promise (credibility).
- `brand_constraint.must_say` phrases must appear in voiceover or CTA, not on-screen text in the hook.

### 2.4 Live Funnel (livestream traffic -- maximize "come to live" intent)

| Niche | 13-17 | 18-24 | 25-34 | 35+ |
|---|---|---|---|---|
| **beauty** | curiosity-question / visual-shock | curiosity-question / contrast | curiosity-question / pain-point | pain-point / curiosity-question |
| **tech** | curiosity-question / number-promise | curiosity-question / number-promise | curiosity-question / pain-point | number-promise / curiosity-question |
| **edu** | curiosity-question / number-promise | curiosity-question / number-promise | curiosity-question / number-promise | curiosity-question / number-promise |
| **food** | visual-shock / curiosity-question | curiosity-question / visual-shock | curiosity-question / pain-point | pain-point / curiosity-question |
| **travel** | curiosity-question / visual-shock | curiosity-question / visual-shock | curiosity-question / contrast | curiosity-question / contrast |
| **fitness** | contrast / curiosity-question | curiosity-question / contrast | curiosity-question / pain-point | pain-point / curiosity-question |
| **finance** | — | curiosity-question / number-promise | curiosity-question / number-promise | curiosity-question / number-promise |
| **comedy** | curiosity-question / visual-shock | curiosity-question / visual-shock | curiosity-question / contrast | curiosity-question / contrast |
| **shop** | curiosity-question / pain-point | curiosity-question / pain-point | curiosity-question / pain-point | pain-point / curiosity-question |
| **other** | curiosity-question / visual-shock | curiosity-question / visual-shock | curiosity-question / pain-point | curiosity-question / pain-point |

**Key insights for Live Funnel**:
- **curiosity-question dominates ALL combinations**: the hook must create "need to see more" that only the livestream satisfies.
- "未完待续" (to be continued) is the universal live funnel hook -- create an information gap.
- Secondary: visual-shock for young, pain-point for older (remind them of the problem the livestream will solve).
- Hook must pre-announce something that only happens in the livestream (exclusive reveal, live demo, limited drop).

### 2.5 Awareness (brand awareness / follower growth -- maximize follow rate)

| Niche | 13-17 | 18-24 | 25-34 | 35+ |
|---|---|---|---|---|
| **beauty** | visual-shock / number-promise | visual-shock / number-promise | number-promise / contrast | number-promise / pain-point |
| **tech** | number-promise / visual-shock | number-promise / curiosity-question | number-promise / pain-point | number-promise / contrast |
| **edu** | number-promise / curiosity-question | number-promise / curiosity-question | number-promise / contrast | number-promise / contrast |
| **food** | visual-shock / number-promise | number-promise / visual-shock | number-promise / contrast | number-promise / pain-point |
| **travel** | visual-shock / number-promise | number-promise / curiosity-question | number-promise / contrast | number-promise / contrast |
| **fitness** | contrast / number-promise | number-promise / contrast | number-promise / pain-point | number-promise / pain-point |
| **finance** | — | number-promise / contrast | number-promise / pain-point | number-promise / contrast |
| **comedy** | visual-shock / number-promise | visual-shock / number-promise | number-promise / contrast | number-promise / contrast |
| **shop** | visual-shock / number-promise | number-promise / visual-shock | number-promise / pain-point | number-promise / pain-point |
| **other** | visual-shock / number-promise | number-promise / visual-shock | number-promise / contrast | number-promise / contrast |

**Key insights for Awareness**:
- **number-promise dominates**: "X things you need to know about Y" builds authority and follow intent.
- Young: visual-shock secondary (memorable brand aesthetic).
- Adult: contrast or pain-point secondary (brand solves a problem).
- Series teaser in hook: "This is part 1 of 3..." drives follow behavior.
- Hook must establish the creator's expertise/credibility in 3 seconds.

---

## 3. Hook 3-Piece Templates

For each hook type, the following templates define the standard visual / voiceover / on_screen_text structure.
These are starting patterns -- the LLM adapts them to the specific topic, niche, and language.

### 3.1 visual-shock

| Component | Template / Pattern |
|---|---|
| **visual** | "[Product/Subject] enters frame at [speed] from [direction]. [Contrast element: dark bg / neon light / color pop]. [Camera move: dolly-in / whip pan / fast zoom]. Focus locks on [detail] for remaining 1.5s. No static opening frame." |
| **voiceover** | Start with an exclamation or rhetorical burst. Pattern: "(Gasps) [Subject] just [surprising action]?!" or "Wait -- [unexpected observation]?!" Keep under 8 words. |
| **on_screen_text** | ≤6 characters (ZH) / ≤5 words (EN). Examples: "What?!", "Wait.", "No way.", "这也行？", "太快了", "看这个" |
| **Example (beauty, EN)** | visual: "A crimson red lipstick bullet enters from frame right at high speed, slams onto a reflective black marble surface. Dolly-in to extreme close-up on the embossed gold logo. No static opening." voiceover: "Wait -- THIS shade?!" on_screen_text: "Wait." |

### 3.2 curiosity-question

| Component | Template / Pattern |
|---|---|
| **visual** | "Medium close-up of [subject/presenter] looking directly at camera. Slight head tilt. Background: defocused but contextually relevant (lab/science for edu, room for lifestyle). Camera: slow push-in during delivery. Freeze frame at question peak." |
| **voiceover** | Question format, counter-intuitive. Pattern: "Did you know [surprising fact]?" or "Why does [common thing] actually [opposite effect]?" or "[X number]% of people think [wrong thing] -- are you one of them?" |
| **on_screen_text** | ≤6 characters (ZH) / ≤5 words (EN). Examples: "Did you know?", "真相是？", "你知道吗？", "Why??", "Ever wonder?" |
| **Example (edu, EN)** | visual: "Close-up of presenter's face, slight raise of eyebrows. Behind: defocused whiteboard with chemical formulas. Camera pushes in slowly." voiceover: "Did you know caffeine actually makes you MORE tired?" on_screen_text: "Did you know?" |

### 3.3 pain-point

| Component | Template / Pattern |
|---|---|
| **visual** | "Relatable daily frustration scene. [Person] struggling with [specific problem]. Shot type: over-shoulder or POV to maximize viewer identification. Lighting: slightly flat/realistic (not glossy). Color grade: slightly desaturated to reinforce 'problem' mood. 2s frustration → 0.5s freeze on worst moment." |
| **voiceover** | Pattern: "Ever [experienced this problem]?" or "If you [do X] and still [get Y result], this is for you." or "Stop [common mistake] -- here's why." |
| **on_screen_text** | ≤6 characters (ZH) / ≤5 words (EN). Examples: "Feel this?", "You too?", "有同感？", "太真实", "Stop." |
| **Example (shop, EN)** | visual: "Over-shoulder POV: person squeezing a nearly-empty lotion bottle, getting nothing. Frustrated shake. Freeze on empty bottle." voiceover: "Ever squeezed the last drop... and got nothing?" on_screen_text: "Feel this?" |

### 3.4 contrast

| Component | Template / Pattern |
|---|---|
| **visual** | "Split-screen or match-cut transition. Left/top: [before state] -- desaturated, flat lighting, cluttered. Right/bottom or cut-to: [after state] -- vibrant, warm lighting, clean. Side-by-side hold 2s. Alternative: rapid match-cut A/B/A/B 0.5s each 4 times." |
| **voiceover** | Pattern: "From [negative state] to [positive state] in [timeframe]." or "[Before number] vs [after number] -- here's how." |
| **on_screen_text** | ≤6 characters (ZH) / ≤5 words (EN). Examples: "Before→After", "3 weeks later", "变化太大", "vs.", "The difference." |
| **Example (beauty, EN)** | visual: "Split screen. Left: bare face, flat morning light, no makeup. Right: same face with dewy foundation, warm ring light, 45-degree angle. Hold 3s." voiceover: "From dull to dewy in 3 minutes. Watch." on_screen_text: "vs." |

### 3.5 number-promise

| Component | Template / Pattern |
|---|---|
| **visual** | "Presenter or animated text: [Number] appears large on screen (bold sans-serif, pop-in animation). Behind: relevant context imagery (product lineup, books, locations). Camera: push-in on the number, then cut to presenter. Text stays as persistent overlay." |
| **voiceover** | Pattern: "[Number] [category] that [benefit/claim]." or "[Percentage] of people don't know [fact]." or "The [ordinal] [thing] will surprise you." |
| **on_screen_text** | The number itself + minimal label. Examples: "3 Ways", "80%", "5 个方法", "#1 Mistake", "90% don't know" |
| **Example (finance, EN)** | visual: "Large '3' pops onto screen (white text, dark bg). Below: 'money myths' fades in. Camera pushes in on the number. Cut to presenter in clean office setting." voiceover: "3 money myths keeping you broke in 2026." on_screen_text: "3 money myths" |

---

## 4. Variant Generation Strategy

When `ab_variants > 1`, generate distinct hook variants. The **mandatory rule** is that each variant must use a **different `retention_tactic`** from the 5-tactic pool. This ensures measurable A/B test results rather than cosmetic rewrites (addresses R10).

### 4.1 Variant Dimensions (5 total)

| Dimension | Description | When to Use |
|---|---|---|
| **D1: Hook type** | Change the primary hook type (e.g., from pain-point to number-promise). Use the `secondary_hook` from the decision table. | Default for variant 2. |
| **D2: Copy angle** | Same hook type, different voiceover phrasing (question vs. statement vs. command). Different on_screen_text wording. | When D1 is exhausted or the secondary hook is too close to primary. |
| **D3: Visual treatment** | Same hook type + copy, different visual composition (angle, lighting, color grade, prop arrangement). | For variant 3+ when copy angles are exhausted. |
| **D4: Pacing** | Same content, different timing (fast 1s entry vs. slow 2s reveal, hard cut vs. fade in). | For variant 4+ when visual angles are exhausted. |
| **D5: Appeal angle** | Same hook type, different emotional/cognitive appeal (fear-of-missing-out vs. aspirational vs. practical). | For variant 5 (max). Requires substantial rewrite of both visual and voiceover. |

### 4.2 Default Variant Allocation (when ab_variants=2)

| Variant | Dimension Used | retention_tactic | Hook Type Source |
|---|---|---|---|
| **Primary (v0)** | (baseline) | primary hook's tactic | Primary from decision table |
| **Variant 1 (v1)** | D1 (hook type change) | secondary hook's tactic | Secondary from decision table |

### 4.3 Extended Allocation (when ab_variants=3 to 5)

| Variant | Dimension | retention_tactic | Hook Type |
|---|---|---|---|
| **Primary** | baseline | tactic #1 | Primary from decision table |
| **Variant 1** | D1 | tactic #2 | Secondary from decision table |
| **Variant 2** | D2 | tactic #3 | Primary hook type, different copy angle |
| **Variant 3** | D3 | tactic #4 | Primary hook type, different visual treatment |
| **Variant 4** | D4 | tactic #5 | Primary hook type, different pacing |

When `ab_variants=5`, all 5 retention tactics are used exactly once each. When `ab_variants > 5`, the value is capped to 5 per §3.3 boundary handling.

If the secondary hook type has the **same retention tactic** as the primary (rare edge case, e.g., if both map to pain-point), bump variant 1 to the next available tactic and use D2 (different copy angle within the same hook type).

### 4.4 Variant Output Format

Each variant must produce the same `{visual, voiceover, on_screen_text, retention_tactic}` structure as the primary hook. All variants are stored in `hook_3s.variants[]`.

---

## 5. Niche-Specific Hook Rules

These override or refine the generic decision table output for specific niches.

### 5.1 comedy

- **Hard rule**: Hook must trigger an emotional response (laugh, surprise, cringe) within **first 1 second**. No slow build-up.
- Common sub-type: unexpected facial expression, physical comedy, absurd prop.
- If `visual-shock` is selected: the "shock" must be funny, not aggressive.
- Avoid voiceover cliches ("You won't believe what happened next").

### 5.2 shop

- **Hard rule**: Product MUST appear in the hook shot. No abstract "teaser" that delays product reveal beyond 3s.
- Hook should show the product in a benefit context, not a spec sheet.
- If `pain-point`: the pain must be directly solvable by the product shown.
- No price mention in hook (price anchoring comes later in the body).

### 5.3 edu

- **Hard rule**: Hook must create an "information gap" -- the viewer must feel they're missing critical knowledge.
- Curiosity-question is the most effective primary, even if the decision table suggests otherwise.
- If `number-promise`: the number must be surprising or counter-intuitive, not "5 tips" (too generic).
- Hook should preview the depth: "This isn't your average science class."

### 5.4 beauty

- **Hard rule**: Hook should prioritize product-on-face or product-on-hand close-up (tactile, sensory).
- Visual-shock in beauty = macro texture shot (lipstick creaminess, foundation blend, highlighter shimmer).
- Avoid "perfect skin" / "flawless" language (R1 compliance).
- Natural lighting in hook if possible -- beauty audience distrusts heavy filters.

### 5.5 finance

- **Hard rule**: Hook MUST open with a number, statistic, or quantified claim.
- Voiceover must sound authoritative (not clickbaity) -- credibility is the retention driver.
- On-screen text should reinforce the number (persistent overlay through hook).
- Avoid get-rich-quick framing (community guideline risk).

### 5.6 tech

- **Hard rule**: Hook must show the product in use, not just on a table.
- Number-promise works well for spec comparisons ("3 reasons this beats...").
- Visual-shock works for design aesthetic reveals (unboxing-style hook).
- Avoid "best X ever" superlatives (compliance).

### 5.7 food

- **Hard rule**: Hook must trigger sensory imagination (sound of sizzle, visual of steam, texture close-up).
- Visual-shock = macro food shot (cheese pull, sauce drip, crispy crunch).
- If `pain-point`: "Hungry with nothing in the fridge?" type scenarios.
- Lighting: warm tones, natural or slightly golden.

### 5.8 travel

- **Hard rule**: Hook must establish a sense of place within 1 second (landmark, landscape, local detail).
- Visual-shock = dramatic drone shot, unexpected perspective of famous location.
- Curiosity-question = "I came here expecting X, found Y instead."
- Color grade: vibrant, saturated (travel content competes with high-production creators).

### 5.9 fitness

- **Hard rule**: Hook should show movement (static poses don't communicate transformation).
- Contrast is highly effective (before/after body composition, form correction).
- Pain-point = relatable gym frustration (wrong form, no results, crowded gym).
- Avoid body-shaming framing or extreme before/after claims (compliance).

---

## 6. Edge Cases & Fallbacks

| Edge Case | Resolution |
|---|---|
| `audience_age` is missing or invalid | Default to `18-24` for hook decision (largest TikTok demographic). |
| `monetization_goal` is missing | Default to `crp` (most common use case). |
| `niche` is `other` | Use `other` row from the decision table. Apply no niche-specific hook rules (§5). |
| `language` is not EN or ZH | Apply the English hook templates (most TikTok content is EN). Adapt culturally. |
| Niche + age combo has `—` (e.g., finance 13-17) | Age-inappropriate content. If user insists, use the closest older age bracket (e.g., finance 13-17 → use 18-24). Add compliance note. |
| Decision table produces same primary for crp and shop | Use the secondary hook to differentiate. The CTA style will differentiate the script body; hooks for different goals can overlap. |

---

## 7. Appendix: Real-World Hook Examples from tiktok-video-gen-report.md

Extracted patterns from the 6 case study categories in the research report (§2 audience targeting, §4 creation workflow).

### 7.1 Dance/Challenge (comedy niche)

| Hook Technique | Visual | Voiceover | Why It Works |
|---|---|---|---|
| Instant movement start | Dancer in motion from frame 0 (no "Hi guys" intro) | No voiceover -- music-only | Algorithm counts <5s views as non-qualified; immediate motion maximizes 5s retention |
| Costume/visual surprise | Unexpected outfit reveal within first 1s | "Wait for it..." (on-screen only) | Visual shock triggers replay and share |
| Mirror challenge framing | Dancer and mirror reflection doing different moves | None -- text overlay: "Which is real?" | Curiosity question drives comments |

> Source: report §6 案例表 -- 舞蹈挑战视频, ~45s, ~80% 有效播放率, ~0.8 RPM

### 7.2 Comedy/Skit (comedy niche)

| Hook Technique | Visual | Voiceover | Why It Works |
|---|---|---|---|
| Setup-punchline in 3s | Character in absurd situation, freeze frame at peak | "You know that feeling when..." | Relatable pain-point hooks drive completion |
| Reaction bait | Over-the-top reaction to mundane event | Single word: "Really?!" | Comment bait -- viewers tag friends |
| Expectation vs. reality | Split screen: expectation (glamorous) vs. reality (messy) | No voiceover -- visual contrast alone carries | Contrast hooks require no language; universal appeal |

> Source: report §6 案例表 -- 搞笑小品/剧情视频, ~60s, ~75% 有效播放率, ~0.7 RPM

### 7.3 Education/Science (edu niche)

| Hook Technique | Visual | Voiceover | Why It Works |
|---|---|---|---|
| Mind-blowing fact first | Animated infographic pops up with surprising stat | "Did you know... [counter-intuitive fact]?" | Number-promise + curiosity-question combo maximizes both retention and share |
| Visual demo of invisible concept | Clear physical demo or animation of abstract concept | "This is actually happening inside your body right now." | Visual-shock with educational value -- high share rate |
| "You've been lied to" framing | Presenter with skeptical expression, text overlay | "Everything you know about [topic] is wrong." | Curiosity-question with contrarian angle -- high comment rate |

> Source: report §6 案例表 -- 教育科普/技能教学, ~180s, ~60% 有效播放率, ~0.5 RPM

### 7.4 Beauty/Makeup (beauty niche)

| Hook Technique | Visual | Voiceover | Why It Works |
|---|---|---|---|
| Product macro reveal | Extreme close-up of texture (cream swirl, powder puff) | "Look at this texture..." (whispered) | Visual shock via tactile sensory appeal; ASMR-quality audio |
| Half-face transformation | Split face: one side done, one bare | "This is 2 minutes of blending." | Contrast hook drives completion to see full face reveal |
| "Mistake everyone makes" | Presenter demonstrating common wrong technique | "Stop applying [product] like this." | Pain-point with authority; saves viewer from embarrassment |

> Source: report §6 案例表 -- 美妆/穿搭教程, ~120s, ~70% 有效播放率, ~0.6 RPM

### 7.5 Tech/Unboxing (tech niche)

| Hook Technique | Visual | Voiceover | Why It Works |
|---|---|---|---|
| Spec reveal in 3s | Product slides in from side with key spec numbers overlaid | "[Number] [unit] for only [price]?!" | Number-promise with price anchoring; drives curiosity |
| Box drop / first look | Dramatic box drop in slow-motion, then speed-ramp to unbox | "This just arrived..." | Visual-shock via packaging aesthetics; ASMR unboxing |
| Comparison hook | Two products side by side, identical test | "Let's see which one actually works." | Contrast hook with competition; high retention through results |

> Source: report §6 案例表 -- 科技产品开箱/评测, ~90s, ~65% 有效播放率, ~0.4 RPM

### 7.6 Travel/Food (travel niche)

| Hook Technique | Visual | Voiceover | Why It Works |
|---|---|---|---|
| Drone/dramatic reveal | Aerial pull-back from detail to vast landscape | "This is [location] -- and you've never seen it like this." | Visual-shock via scale contrast; cinematic quality |
| First bite reaction | Close-up on food, then person's reaction shot | "I flew 12 hours for this one bite." | Pain-point (effort) + number-promise (12 hours) = investment hooks retention |
| Hidden gem framing | Narrow alley or unmarked door, push-in | "No sign. No menu. Best [food] in [city]." | Curiosity-question + exclusivity appeal; drives save/bookmark |

> Source: report §6 案例表 -- 旅行/美食探店视频, ~150s, ~55% 有效播放率, ~0.3 RPM

---

## 8. Hook Strategy Execution Checklist

When Step 2 executes, the LLM should verify:

- [ ] Decision table lookup returned a non-null `primary_hook_type`.
- [ ] The primary hook uses exactly one `retention_tactic` from the 5-tactic pool.
- [ ] Primary hook 3-piece set (visual / voiceover / on_screen_text) is non-empty.
- [ ] `on_screen_text` is ≤6 characters (ZH) or ≤5 words (EN).
- [ ] Niche-specific rules (§5) have been applied (if niche is not `other`).
- [ ] Variant count matches `ab_variants` (capped at 5).
- [ ] Each variant uses a **different** `retention_tactic` from all others.
- [ ] Variant 1 uses `secondary_hook_type` from decision table (D1 dimension).
- [ ] No brand logo large close-up in any hook variant (per `tiktok-hook-3s` template caveat #3).
- [ ] `voiceover` does not contain banned words from `compliance-rules.md` §2.
