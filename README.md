# tiktok-script-gen

> TikTok high-RPM script generation skill — produces complete video scripts (scenes, copy, CTA, hashtags, compliance checks, Seedance 2.0 prompt drafts) ready for downstream rendering.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-8A2BE2)](https://claude.com/claude-code)

---

## What It Does

**tiktok-script-gen** is a monetization-aware TikTok script generator built as a Claude Code skill. It orchestrates the full script pipeline — from hook design and scene breakdown to CTA strategy, hashtag optimization, compliance review, and pre-filled Seedance 2.0 AI video prompts.

Output is delivered in **Markdown** (creator-facing readable format) and **JSON** (machine-readable for downstream tools).

## Key Features

- **Hook-first architecture** — generates 2–5 A/B hook variants with 3-second retention strategies per variant
- **Full scene breakdown** — timestamped visual descriptions, voiceover text, on-screen text, sound design per scene
- **Monetization-aware** — scripts optimized for CRP / TikTok Shop / Brand Deals / Live Funnel / Awareness goals
- **Compliance engine** — platform rule checks (TikTok US, SEA, CN-overseas) built into every script
- **Seedance 2.0 prompt drafts** — each scene gets a pre-filled AI video generation prompt with model routing hints
- **18-template pool** — 13 reused from `video_prompt_gen` + 5 TikTok-specific (beat-sync, before-after, 3s-hook, product-on-face, twist-ending)
- **Multi-language output** — en / zh / es / pt / ja / ko and more
- **Style presets** — fast-cut / story / ASMR / talking-head / faceless
- **KPI estimation** — predicted completion rate, engagement rate, RPM range per monetization goal
- **Brand safety** — must-say / banned-term / logo-rule constraints injected per brand

## Quick Start

### As a Claude Code Skill

1. Install the skill into `~/.claude/skills/tiktok-script-gen/`
2. Invoke in Claude Code:

```
/tiktok-script \
  --topic "lipstick review for pale skin" \
  --niche beauty \
  --target_audience '{"age":"18-24","region":"US","interests":["kbeauty"]}' \
  --monetization_goal shop \
  --duration_sec 90 \
  --language en
```

### Trigger Phrases

Say any of these in Claude Code and the skill auto-activates:

| Chinese | English |
|---------|---------|
| TikTok 脚本 / 抖音脚本 | tiktok script |
| 短视频脚本 / 带货脚本 | short-form script / hook script |
| 高 RPM / 完播率 | high rpm / tiktok monetization |
| TikTok 变现 / TikTok Shop | tiktok shop / creator rewards |
| 帮我写一条 TikTok | viral tiktok script |

## Supported Monetization Goals

| Goal | Optimizes For | Typical RPM |
|------|--------------|-------------|
| `crp` | Completion rate, watch time (60s+ eligible) | $0.50–$2.00 |
| `shop` | Product showcases, affiliate links, conversion CTAs | Commission-based |
| `brand_deal` | Brand-safe messaging, sponsor integration | Flat-fee + bonus |
| `live_funnel` | Redirect to livestream, follower conversion | Live GPM |
| `awareness` | Reach, shares, trend alignment | Organic growth |

## File Structure

```
tiktok-script-gen/
├── SKILL.md                          # Skill definition & orchestration logic
├── README.md                         # This file
└── references/
    ├── compliance-rules.md           # Platform compliance by region
    ├── hook-strategies.md            # Hook patterns & 3-second retention tactics
    ├── kpi-table.md                  # KPI benchmarks per niche × duration
    ├── monetization-playbooks.md     # Monetization strategy per goal
    ├── output-schema.json            # JSON output schema spec
    ├── seedance-mirror-rules.md      # Seedance 2.0 prompt mirroring rules
    ├── template-registry.md          # 18-template pool catalog
    └── templates/                    # TikTok-specific script templates
        ├── tiktok-beat-sync.md
        ├── tiktok-before-after.md
        ├── tiktok-hook-3s.md
        ├── tiktok-product-on-face.md
        └── tiktok-twist-ending.md
```

## Output Format

### Markdown (creator view)
- Video title + cover text suggestion
- Hook variants (A/B/n)
- Scene table: timestamp | visual | voiceover | on-screen text | sound
- CTA strategy with placement timing
- Hashtag cluster (broad + niche + trending)
- Music/sound suggestions
- Compliance notes

### JSON (machine view)
- Structured per `references/output-schema.json`
- Seedance 2.0 prompt array with model routing
- A/B variant metadata
- KPI estimates

## Prerequisites

- [Claude Code](https://claude.com/claude-code)
- Downstream rendering: [Seedance 2.0](https://seedance.ai) or compatible AI video model (Sora 2, Veo 3.1, Kling 2.6)

## Related Skills

- [`video_prompt_gen`](https://github.com/lostinheaven-knt/video-prompt-gen) — AI video model prompt generation (13 shared templates)
- `geo-content-generator` — Brand marketing content for Weimob platform

## License

MIT
