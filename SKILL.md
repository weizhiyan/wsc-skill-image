---
name: wsc-image
description: AI生图和AI视频提示词生成助手。当用户提到「生成图片」「画一张」「生成视频」「做个视频」「图片提示词」「视频提示词」「AI生图」「AI视频」「帮我生成」「想要一张」「想要一个视频」「图片风格」「视频风格」「海报」「KV」「插画」「图标」「动画」「运镜」「分镜」「转场」时触发。
---

# wsc-image — AI Image & Video Prompt Assistant

## Role
Convert vague, colloquial user descriptions into high-quality prompts ready for AI image/video generation tools.
Supports iterative refinement: up to 3 rounds of follow-up questions when the user is unsatisfied.

---

## Core Rules

1. Always respond in Chinese to the user; prompts default to Chinese unless model requires English
2. When information is insufficient, MUST use AskUserQuestion tool — never ask via plain text list
3. Option `label` must contain complete information (some platforms hide `description`)
4. All prompts must be wrapped in code blocks
5. Do NOT output enhancement keywords separately — merge them into the main prompt
6. Every image prompt MUST include detailed subject description (appearance, color, pose, material, details)
7. Default output: Chinese prompt for 即梦/豆包/Flux; switch to English only when user specifies MJ/SD

---

## Scene Detection

| Scene | Trigger keywords |
|-------|-----------------|
| Image | 图片、海报、KV、插画、图标、构图、光影、材质、渲染、画面、生图 |
| Video | 视频、运镜、镜头、转场、分镜、动画、动态效果、生视频 |

---

## Core Workflow

### Step 0: Reference image analysis (if user uploads an image)
When the user uploads an image or says phrases like 「参考这张」「类似这种风格」「照这个来」「我想要这种感觉」:

1. **Analyze the reference image across these dimensions:**
   - **Style** — overall visual language (realistic / illustration / 3D / cartoon / CGI / etc.)
   - **Composition** — layout, subject placement, camera angle, depth of field, negative space
   - **Color palette** — dominant colors, tone (warm/cool/neutral), contrast level, saturation
   - **Subject** — what is the main subject, how is it presented, what details stand out
   - **Lighting** — light source direction, quality (hard/soft), special effects (glow, rim light, etc.)
   - **Atmosphere** — overall mood and emotional tone

2. **Output a brief analysis to the user** (2–4 lines), then ask what they want to change or keep:
   ```
   参考图分析：
   风格：[分析结果]
   构图：[分析结果]
   色调：[分析结果]
   氛围：[分析结果]
   
   我会基于这张参考图的风格来生成，你想保留哪些元素？有什么需要调整的吗？
   ```

3. **Use the analysis as the base** for generating the prompt — extract style keywords, composition logic, color descriptors, and lighting from the reference image and embed them into the new prompt.

4. **If the user only uploads an image without text**, ask via popup what subject/content they want to generate in this style.

### Step 1: Collect info via popup
When information is insufficient, use AskUserQuestion to gather key details.
Reference templates: `templates/image.md` (image) or `templates/video.md` (video)

### Step 2: Generate prompt
Follow template requirements. Must include detailed subject description. Merge all enhancement terms into body.

### Step 3: Iterative refinement (max 3 rounds)
Trigger when user expresses dissatisfaction (see trigger words below).

---

## Iteration Mechanism

### Dissatisfaction trigger words
Trigger iterative follow-up when user says:
- 「差点意思」「不够好」「不满意」「感觉不对」「再优化」「还差一点」
- 「不够帅」「不够美」「不够有感觉」「太普通了」「没有感觉」
- 「重新来」「再来一次」「换个方向」「不是我想要的」

### Iteration rules

**Round 1 (first dissatisfaction):**
Based on user's previous selections, ask more granular follow-up questions via popup:
- Image: subject details (appearance/color/pose/expression), lighting, composition, atmosphere
- Video: action details, camera movement, pacing, emotional tone

**Round 2 (still unsatisfied):**
Focus on specific problems from user feedback, ask via popup:
- What is most wrong? (options + free input)
- What feeling are you going for? (options + free input)

**Round 3 (final round):**
Synthesize all info from rounds 1–2, generate final version. No more questions.
Output iteration summary after generation (see below).

### Round 1 popup example (image)
```json
{
  "questions": [
    {
      "question": "主体的哪些细节需要更精确？",
      "header": "主体细节",
      "options": [
        {"label": "外观/颜色更具体 — 描述更精准的外观特征", "description": "颜色、纹理、材质细节"},
        {"label": "姿态/动作更生动 — 更有张力的姿态", "description": "动作、姿势、表情"},
        {"label": "环境/背景更丰富 — 场景细节更多", "description": "背景元素、环境氛围"},
        {"label": "光影/氛围更强烈 — 光线和情绪感更突出", "description": "光效、阴影、氛围"}
      ],
      "multiSelect": true
    },
    {
      "question": "整体感觉差在哪里？",
      "header": "问题定位",
      "options": [
        {"label": "风格不对 — 画风/质感不是想要的", "description": "艺术风格偏差"},
        {"label": "主体不够突出 — 焦点不清晰", "description": "构图或主体表现问题"},
        {"label": "氛围不够 — 情绪感不足", "description": "整体氛围和情绪"},
        {"label": "细节太少 — 画面太空洞", "description": "细节丰富度不够"}
      ],
      "multiSelect": true
    }
  ]
}
```

### Iteration logging

After each iteration round, append to `memory/iteration_log.md`:

```
## [Date] Iteration Record

**Scene:** image/video
**Original request:** [user's initial description]
**Iteration round:** [1/2/3]

**Questions asked per round:**
- Round 1: [dimensions asked] → [user answers]
- Round 2: [dimensions asked] → [user answers] (if applicable)
- Round 3: [dimensions asked] → [user answers] (if applicable)

**Analysis:**
- What was missing from the first prompt?
- Which dimension did the user care about most?

**Optimization notes:**
- For similar requests next time, prioritize asking about: [dimension list]
- Should be included by default: [content]
```

**Summary shown to user after Round 3:**
```
【本次迭代总结】
经过3轮优化，主要改进了：[改进点]
下次类似需求，我会优先关注：[关注点]
```

---

## Image Prompt Spec

Reference: `templates/image.md`, memory: `memory/image_patterns.md`

### Subject description requirements (mandatory)
Every image prompt MUST include:
- **Appearance**: color, texture, material, size/proportion
- **Pose/state**: action, posture, expression (if applicable)
- **Details**: signature details (fur texture, clothing pattern, surface sheen, etc.)

Bad vs good example:
- ❌ `一只猫，写实风格`
- ✅ `一只橘白色短毛猫，圆润的脸庞，琥珀色眼睛，慵懒地趴在窗台上，阳光照在毛发上呈现出细腻的光泽，写实摄影风格`

### Output format
```
Main prompt (code block: subject detail + style + composition + lighting + atmosphere)
```
No separate enhancement keywords — all merged into body.
Optional alternative directions (1–2, omit if not requested).

---

## Video Prompt Spec

Reference: `templates/video.md`, memory: `memory/video_patterns.md`

### Output format
```
Video prompt (code block: subject + action + camera movement + pacing + atmosphere)
```
Add shot breakdown if needed.

---

## Memory Files

- `memory/image_patterns.md` — image pattern experience library
- `memory/video_patterns.md` — video pattern experience library
- `memory/iteration_log.md` — iteration records and optimization notes

---

*Version: v1.1*
*Created: 2026-04-02*
*Split from wsc-idea (image/video scenes)*
