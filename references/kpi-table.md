# KPI Estimation Table — tiktok-script-gen v1

> Lookup tables for expected completion rate and RPM ranges, indexed by niche x duration bucket.
> Used in Step 7 of the reasoning workflow. Values are industry estimates based on successful case analysis — NOT guarantees.
>
> **source**: tiktok-video-gen-report.md §6 case studies + industry experience estimates
> **valid_until**: 2026-12-31
> **version**: v1
> **note**: Ranges are for reference only. They do not constitute investment or income guarantees.

---

## Duration Bucket Classification

| Bucket | Range | CRP Eligible |
|---|---|---|
| `<60s` | 1-59 seconds | No |
| `60-74s` | 60-74 seconds | Yes (minimum) |
| `75-89s` | 75-89 seconds | Yes |
| `90-119s` | 90-119 seconds | Yes |
| `120-180s` | 120-180 seconds | Yes |
| `>180s` | 181+ seconds | Yes (with completion rate risk) |

The bucket is determined from `duration_sec` after Step 1 parsing:
- `duration_sec < 60` → bucket `<60s`
- `60 <= duration_sec <= 74` → bucket `60-74s`
- `75 <= duration_sec <= 89` → bucket `75-89s`
- `90 <= duration_sec <= 119` → bucket `90-119s`
- `120 <= duration_sec <= 180` → bucket `120-180s`
- `duration_sec > 180` → bucket `>180s`

---

## 1. Expected Completion Rate (%)

The completion rate is the percentage of viewers who watch the entire video.
Higher is better for algorithm recommendation weight.

| Niche | <60s | 60-74s | 75-89s | 90-119s | 120-180s | >180s |
|---|---|---|---|---|---|---|
| **comedy** (搞笑) | 55-70% | 50-65% | 45-60% | 38-52% | 30-45% | 22-35% |
| **beauty** (美妆) | 50-65% | 48-60% | 45-58% | 40-52% | 32-45% | 25-38% |
| **shop** (带货) | 45-60% | 42-55% | 38-50% | 32-45% | 25-38% | 18-30% |
| **edu** (教育/科普) | 40-55% | 45-58% | 48-60% | 50-62% | 45-58% | 35-48% |
| **tech** (科技) | 42-55% | 45-58% | 45-58% | 42-55% | 35-48% | 28-40% |
| **food** (美食) | 50-65% | 48-60% | 42-55% | 35-48% | 28-40% | 20-32% |
| **travel** (旅行) | 45-60% | 45-58% | 45-58% | 42-55% | 38-50% | 30-42% |
| **fitness** (健身) | 48-62% | 45-58% | 42-55% | 38-50% | 32-45% | 25-38% |
| **finance** (财经) | 38-52% | 42-55% | 45-58% | 45-58% | 40-52% | 32-45% |
| **other** (其他) | 40-55% | 40-55% | 40-55% | 38-52% | 32-45% | 25-38% |

### Pattern Notes

- **Emotion-type niches** (comedy, beauty, food): completion rate decreases as duration increases. Shorter is better.
- **Information-type niches** (edu, tech, finance): completion rate peaks at 90-119s (viewers seek depth). Longer can perform better than shorter.
- **Shop niche**: steepest completion rate decline — viewers decide quickly whether they want to buy.
- **Travel/fitness**: moderate decline, less sensitive to duration within 60-180s range.

---

## 2. Expected RPM (USD)

RPM = Revenue Per Mille (earnings per 1,000 qualified views).
Only calculated for duration >= 60s (CRP-eligible). Shorter videos always show `0-0`.

| Niche | <60s | 60-74s | 75-89s | 90-119s | 120-180s | >180s |
|---|---|---|---|---|---|---|
| **comedy** | 0-0 | 0.3-0.6 | 0.4-0.8 | 0.5-0.9 | 0.5-0.9 | 0.4-0.7 |
| **beauty** | 0-0 | 0.5-0.8 | 0.6-1.0 | 0.7-1.2 | 0.7-1.2 | 0.6-1.0 |
| **shop** | 0-0 | 0.6-1.2 | 0.8-1.5 | 1.0-1.8 | 1.0-1.8 | 0.8-1.4 |
| **edu** | 0-0 | 0.5-0.9 | 0.7-1.1 | 0.8-1.4 | 0.9-1.5 | 0.8-1.3 |
| **tech** | 0-0 | 0.6-1.0 | 0.7-1.2 | 0.9-1.5 | 1.0-1.6 | 0.8-1.4 |
| **food** | 0-0 | 0.4-0.7 | 0.5-0.9 | 0.6-1.0 | 0.6-1.0 | 0.4-0.7 |
| **travel** | 0-0 | 0.4-0.8 | 0.5-0.9 | 0.6-1.0 | 0.7-1.2 | 0.6-1.0 |
| **fitness** | 0-0 | 0.4-0.7 | 0.5-0.9 | 0.6-1.0 | 0.6-1.0 | 0.5-0.8 |
| **finance** | 0-0 | 0.7-1.3 | 0.9-1.6 | 1.1-1.9 | 1.2-2.0 | 1.0-1.8 |
| **other** | 0-0 | 0.3-0.6 | 0.4-0.7 | 0.5-0.8 | 0.5-0.8 | 0.4-0.7 |

### Monotonicity Self-Check (AC7)

For each niche, verify that RPM upper bound is monotonically non-decreasing from `60-74s` to `120-180s`:

| Niche | 60-74s upper | 75-89s upper | 90-119s upper | 120-180s upper | Monotonic? |
|---|---|---|---|---|---|
| comedy | 0.6 | 0.8 | 0.9 | 0.9 | OK |
| beauty | 0.8 | 1.0 | 1.2 | 1.2 | OK |
| shop | 1.2 | 1.5 | 1.8 | 1.8 | OK |
| edu | 0.9 | 1.1 | 1.4 | 1.5 | OK |
| tech | 1.0 | 1.2 | 1.5 | 1.6 | OK |
| food | 0.7 | 0.9 | 1.0 | 1.0 | OK |
| travel | 0.8 | 0.9 | 1.0 | 1.2 | OK |
| fitness | 0.7 | 0.9 | 1.0 | 1.0 | OK |
| finance | 1.3 | 1.6 | 1.9 | 2.0 | OK |
| other | 0.6 | 0.7 | 0.8 | 0.8 | OK |

All rows pass monotonicity check. RPM upper bound never decreases from 60-74s to 120-180s.

### Degradation at >180s

RPM drops for `>180s` vs `120-180s` for all niches. This reflects the reality that ultra-long videos have lower completion rates
and therefore lower effective RPM despite longer ad placement opportunities.

### <60s Rule

All `<60s` buckets show `0-0` because CRP only counts views on videos >= 60 seconds.
When this bucket is selected, the lookup function MUST also populate `compliance_check.notes` with the Rule 1.1 warning.

---

## 3. Lookup Function Pseudocode

When Step 7 executes, the following logic determines the KPI ranges:

```
function lookup_kpi(niche, duration_sec):
    bucket = classify_bucket(duration_sec)
    completion_rate = COMPLETION_TABLE[niche][bucket]  # fallback to "other" row if niche not found
    rpm = RPM_TABLE[niche][bucket]                      # fallback to "other" row if niche not found

    if bucket == "<60s":
        rpm = "0-0"
        append note: "duration_sec < 60, not CRP-eligible, RPM set to 0-0"

    return {
        expected_completion_rate: completion_rate,
        expected_rpm_usd: rpm
    }
```

### Fallback Logic

- If `niche` is not in the 10-class enum → use `other` row.
- If `duration_sec` is exactly on a bucket boundary (e.g., exactly 60) → use the lower bucket (`60-74s`).
- No improvement suggestions are offered (per Q7 decision: "不做自动改进建议").

---

## 4. Version History

| Version | Date | Changes |
|---|---|---|
| v1 | 2026-06-02 | Initial release. 10 niches x 6 duration buckets. Based on tiktok-video-gen-report.md §6 case studies + industry estimates. |
