# Template Registry — tiktok-script-gen v1

> This file is the single source of truth for all template IDs used in tiktok-script-gen.
> It defines the 18-template pool (13 reused from video_prompt_gen + 5 added TikTok-specific),
> the downstream mapping from new templates to legacy template IDs,
> and the drift-check command for CI synchronization.

---

## File Metadata

```yaml
version: v1
sync_source: ~/.claude/skills/video_prompt_gen/SKILL.md §3-§4
total_templates: 18
last_synced: 2026-06-02
```

---

## Template Pool

### 1. Reused from video_prompt_gen (13 templates)

These template IDs are copied directly from `~/.claude/skills/video_prompt_gen/SKILL.md` §4 Model Routing Table.
They serve as valid values for `seedance_segments[].template_hint` and `scene_list[].template_id` (when using a legacy template directly).

| # | Template ID | Category | Default Model | Typical Scene |
|---|---|---|---|---|
| 1 | `product-rotation` | E-commerce | Seedance 2.0 | Product 360-degree rotation shot |
| 2 | `clothing-tryout` | E-commerce | Kling 2.6 | Virtual try-on / outfit change |
| 3 | `cosmetic-texture` | E-commerce | Seedance 2.0 | Macro texture rendering (lipstick, foundation, serum) |
| 4 | `food-closeup` | E-commerce | Seedance 2.0 | Food macro / pouring / steam / sizzling |
| 5 | `talking-head` | E-commerce | Seedance 2.0 | Presenter speaking to camera, Chinese voiceover |
| 6 | `logo-animation` | Brand | Veo 3.1 | Logo reveal / motion graphics |
| 7 | `appliance-usage` | Brand | Veo 3.1 | Product demonstration (kitchen tools, electronics) |
| 8 | `brand-emotional` | Brand | Veo 3.1 | Emotional lifestyle brand ad (3-6 sentence narrative) |
| 9 | `narrative-suspense` | Narrative | Sora 2 | Suspense / thriller short narrative |
| 10 | `hitchcock-zoom` | Narrative/Camera | Veo 3.1 | Dolly-zoom effect (Hitchcock-style) |
| 11 | `orbit-shot` | Camera | Kling 2.6 | Orbital camera movement around subject |
| 12 | `style-ink-wash` | Stylized | Kling 2.6 | Chinese ink-wash painting aesthetic |
| 13 | `style-cyberpunk` | Stylized | Veo 3.1 | Cyberpunk / neon / HDR aesthetic |

### 2. Added TikTok-Specific (5 templates)

These template IDs are used **internally** by tiktok-script-gen for scene orchestration.
They MUST NOT appear as direct `template_hint` values (use `downstream_mapping` instead).

| # | Template ID | Suitable Scene | Recommended Duration | Typical Shot Type |
|---|---|---|---|---|
| 14 | `tiktok-hook-3s` | First 3 seconds — attention grab for all niches | 3s | Close-up / extreme close-up |
| 15 | `tiktok-product-on-face` | Beauty/skincare product demo on face/body | 5-15s | Half-face close-up / 45-degree portrait |
| 16 | `tiktok-before-after` | Before/after comparison (weight loss, skincare, cleaning) | 4-8s (split) / 8-15s (cut) | Split-screen / match-cut |
| 17 | `tiktok-beat-sync` | Dance / challenge / rapid-cut product showcase | 8-20s | Rapid cut / multi-angle |
| 18 | `tiktok-twist-ending` | Script ending reversal shot (boosts comment rate) | 3-5s | Reveal shot / pull-back |

---

## Downstream Mapping

When tiktok-script-gen generates `seedance_segments[].template_hint`, the value MUST be one of the 13 legacy template IDs.
The following mapping table defines how each of the 5 new TikTok templates resolves to a legacy template ID,
based on the video's niche.

| New Template | Niche | Legacy template_hint |
|---|---|---|
| `tiktok-hook-3s` | beauty | `cosmetic-texture` |
| `tiktok-hook-3s` | food | `food-closeup` |
| `tiktok-hook-3s` | shop / tech / other | `product-rotation` |
| `tiktok-hook-3s` | edu / finance / comedy / travel / fitness | `talking-head` |
| `tiktok-product-on-face` | beauty / other | `cosmetic-texture` |
| `tiktok-product-on-face` | shop (fashion/accessories) | `clothing-tryout` |
| `tiktok-before-after` | beauty / fitness / other | `cosmetic-texture` |
| `tiktok-before-after` | tech / shop / food | `appliance-usage` |
| `tiktok-beat-sync` | all (default) | `product-rotation` |
| `tiktok-beat-sync` | travel / tech (dynamic) | `orbit-shot` |
| `tiktok-twist-ending` | all (narrative-style) | `narrative-suspense` |
| `tiktok-twist-ending` | edu / comedy (talking) | `talking-head` |

---

## CI Drift Check

The following command verifies that `reused_from_video_prompt_gen` in this file stays synchronized
with the actual template list in video_prompt_gen's source of truth (its SKILL.md §4 routing table).

```bash
# Drift check: compare template IDs in this file vs video_prompt_gen SKILL.md
#
# Step 1: Extract the reused_from_video_prompt_gen list from this file
# Step 2: Extract template IDs from video_prompt_gen SKILL.md §4 (Model Routing Table)
# Step 3: Diff

# From this file (grep for legacy template names):
grep -E '^| [0-9]+ \| `[a-z-]+`' ~/.claude/skills/tiktok-script-gen/references/template-registry.md \
  | grep -v 'Category\|Default\|--\|Template ID\|tiktok-\|downstream\|Total' \
  | sed 's/.*`\(.*\)`.*/\1/' | sort > /tmp/tiktok_registry_templates.txt

# From video_prompt_gen SKILL.md §4:
grep -E '^\| [a-z]' ~/.claude/skills/video_prompt_gen/SKILL.md \
  | head -13 \
  | awk -F'|' '{gsub(/^[ \t]+|[ \t]+$/, "", $2); print $2}' \
  | sort > /tmp/video_prompt_gen_templates.txt

# Diff:
diff /tmp/tiktok_registry_templates.txt /tmp/video_prompt_gen_templates.txt
```

If `diff` returns non-zero, **video_prompt_gen has added or removed templates**.
Action: coder-agent must update this file's `reused_from_video_prompt_gen` list and bump `version`.
