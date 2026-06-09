# TikTok 场景模板：前 3 秒钩子镜头

> 知识截止：2026-06
> 场景 ID: tiktok-hook-3s
> 分类：TikTok 专属 · 钩子镜头
> source: technical-design.md §5.1

---

## 1. 场景定义

适用于所有 niche 的 0-3 秒钩子镜头，是 TikTok 短视频最重要的 3 秒——FYP 推荐权重直接取决于前 3 秒的完播率。本模板通过高冲击力的视觉开场、悬念设置和极简 on-screen text，在极短时间内触发用户好奇心，阻止划走行为。

**核心设计原则**：视觉冲击优先于信息密度；on_screen_text 不超过 6 个字；不要在此镜头出现品牌 logo 大特写（用户在未建立信任前反感）。

---

## 2. 关键变量列表

| 变量 | 说明 | 示例值 |
|---|---|---|
| `{hook_visual}` | 钩子视觉主体（产品/人/场景元素） | "红色口红从右侧快速推入" / "打翻的咖啡杯慢镜头" |
| `{hook_motion}` | 入场动作（单动作） | "快速推入" / "自由落体" / "突然点亮" |
| `{hook_reveal}` | 揭晓/定格方式 | "停在金属台面中央" / "悬停空中 1 秒" |
| `{background}` | 背景类型 | "纯黑背景高对比" / "虚化街道" / "纯白工作室" |
| `{lighting}` | 光线策略 | "顶光高反差" / "左上方柔光" / "霓虹背光" |
| `{camera_move}` | 运镜类型（单运镜） | "轻微 dolly-in" / "快速 whip-pan" / "静止特写" |
| `{retention_tactic}` | 留存策略 | visual-shock / curiosity-question / pain-point / number-promise / contrast |

---

## 3. 推荐时长

| 时长 | 说明 |
|---|---|
| **3 秒**（固定） | 此模板不设可变时长，强制 3 秒。scene_list[0] 的 start_sec=0、end_sec=3。 |

---

## 4. 典型 shot_type

| 镜头类型 | 适用 niche | 说明 |
|---|---|---|
| **close-up** | beauty / food / shop | 产品/质地特写，细节冲击 |
| **extreme close-up** | beauty（唇部/眼妆）/ food（流体） | 超微距，前所未见的视角 |
| **high-angle dynamic** | tech / shop / comedy | 俯拍 + 快速运动，制造"正在发生"的紧迫感 |
| **talking-head close-up** | edu / finance / awareness | 人物直视镜头 + 悬念问题 |

---

## 5. Seedance prompt 示例

### 美妆类（cosmetic-texture 映射）— 76 字

```
特写镜头：一支红色口红从画面右侧快速推入，停在金属台面中央，
台面反光高对比，背景纯黑。轻微 dolly-in 运镜，约 1 秒内完成入场，
最后 2 秒静止聚焦口红膏体纹理细节。
```

### 食品类（food-closeup 映射）— 68 字

```
微距特写：巧克力酱从上方缓慢滴落到松饼表面，液体接触瞬间激起微小飞溅，
暖色顶光，背景虚化为厨房场景。镜头静止，焦点锁定滴落接触点。
```

### 电商通用（product-rotation 映射）— 72 字

```
俯拍特写：无线耳机从画面外快速滑入画面中央，黑色亚克力台面反射柔光，
背景纯黑高对比。镜头静止，耳机在台面上轻微旋转 90 度后定格。
```

### 口播类（talking-head 映射）— 65 字

```
近景特写：一位模特直视镜头，表情略带疑问，嘴唇微启似要说话，
左上方柔光勾勒面部轮廓，背景虚化为浅灰色。镜头静止 3 秒。
```

---

## 6. 典型失败反例

### 失败 prompt

```
震撼的 8K 超高清大片感开场，口红完美旋转同时镜头推拉摇移，
绝美光影，高质量品牌展示
```

**失败原因**：
- 抽象词 >= 6（震撼、8K、超高清、大片感、完美、绝美、高质量）→ 违反 R1
- 多动作（旋转） + 多运镜（推、拉、摇、移）→ 违反 R3、R4
- on_screen_text 意图本质是品牌展示 → 违反注意事项#2（hook 里不出现品牌 logo 大特写）

---

## 7. template_hint 映射（按 niche）

| niche | template_hint（传递给 video_prompt_gen） |
|---|---|
| beauty | `cosmetic-texture` |
| food | `food-closeup` |
| shop / tech / other | `product-rotation` |
| edu / finance / comedy / travel / fitness | `talking-head` |

> 映射规则来源：`template-registry.md` downstream_mapping 表。

---

## 8. retention_tactic 取值范围

| retention_tactic | 说明 | 典型视觉手法 |
|---|---|---|
| `visual-shock` | 视觉冲击 | 高对比画面、快速运镜、意外色彩 |
| `curiosity-question` | 悬念问题 | 未完成动作、反常画面、信息缺口 |
| `pain-point` | 痛点演示 | 展示用户日常困扰场景 |
| `contrast` | 反差对比 | before/after、期望 vs 现实 |
| `number-promise` | 数字承诺 | "3 个方法" / "90% 的人不知道" — 配合 on_screen_text |

**强制约束**：`retention_tactic` 字段必填，取值必须在上表 5 种内。

---

## 9. on_screen_text 约束

- 不超过 **6 个汉字**（中文）/ **5 个单词**（英文）。
- 位置：画面中央或下方 1/3 处，不遮挡视觉主体。
- 内容：与视觉互补，不重复 visual_description 已表达的信息。

**示例**（对应上面的 prompt 示例）：
- 美妆口红 → "这支不一样"
- 巧克力酱 → "停不下来"
- 无线耳机 → "听一次就懂"
- 口播悬念 → "你真的会洗脸吗"

---

## 10. 注意事项（Caveats）

1. **视觉冲击优先于信息密度**：3 秒内只能传达一个视觉锚点，不要试图塞入产品功能 / 价格 / 品牌信息。
2. **避免品牌 logo 大特写**：用户在未建立信任前对硬广敏感，hook 阶段出现 logo 大特写会触发反感。
3. **必须配置"未完成动作"或"悬念问题"**：完整的 3 秒动作（如产品从头到尾旋转完毕）会让用户觉得"已看完"而划走。保留一个未完成状态（如旋转到一半定格、口型停在"话还没说出口"）。
4. **retention_tactic 必填**：从 5 种策略中选 1 种，不得为空。此字段用于 Step 2 变体生成时的策略轮选。
5. **scene_list[0] 强制使用此模板**（AC4）：无论 niche 和 monetization_goal 如何，第一个 scene 的 `template_id` 必须为 `tiktok-hook-3s`。
