# Image Prompt Template

Default platform: 即梦、豆包、千问 (Chinese natural language).
Use Midjourney / SD format ONLY when user explicitly specifies.
Always reference `memory/image_patterns.md` first.

## Model Classification & Prompt Strategy

| Model Type | Examples | Language | Format |
|-----------|----------|----------|--------|
| **Natural language (default)** | 即梦、豆包、Gemini、Flux、千问 | Chinese or English | Fluent sentences, no special syntax |
| **Tag-weight models** | SDXL、SD1.5、NovelAI | English | Comma-separated tags, `(tag:1.2)` weight syntax, negative prompt required |
| **MJ-style models** | Midjourney、Niji | English | Concise keywords + params (`--ar 16:9 --v 6`), style words at end |

**Default behavior:** Do NOT ask about model. Output Chinese natural language prompt for 即梦/豆包/Flux directly.
**Switch trigger:** Auto-switch format when user mentions a specific model name.
- 「SD」「SDXL」「SD1.5」→ English tag format + negative prompt
- 「MJ」「Midjourney」「Niji」→ English keyword format + params
- 「Flux」「即梦」「豆包」「千问」「Gemini」→ Keep natural language (Chinese or English)

## Output Structure

Output prompt directly. Do NOT show reasoning or analysis.
Merge style, composition, lighting, material, and enhancement terms into the main prompt body — never list them separately.

```
Main prompt (code block: subject detail + style + composition + lighting + atmosphere + enhancements merged in)

Optional alternative directions (1–2, omit if not requested)
```

## Subject Description Requirements (MANDATORY)

Every image prompt MUST include these dimensions for the subject:

| Dimension | Description | Example |
|-----------|-------------|---------|
| Appearance/Color | Specific color, texture, material | 橘白色短毛、金属光泽表面 |
| Shape/Proportion | Size, form, proportional relationships | 圆润的脸庞、修长的身形 |
| Pose/Action | Movement, posture, orientation | 慵懒地趴着、侧身回望 |
| Expression/State | Emotion, demeanor (if applicable) | 眼神锐利、表情温和 |
| Signature details | Distinctive features | 琥珀色眼睛、毛发光泽 |

**Bad vs good:**
- ❌ `一只猫，写实风格，温馨氛围`
- ✅ `一只橘白色短毛猫，圆润的脸庞，琥珀色眼睛，慵懒地趴在窗台软垫上，阳光从侧面照射在毛发上呈现出细腻的金色光泽，写实摄影风格，温馨家庭氛围，浅景深，8K超清`

## Follow-up Questions

When information is insufficient, MUST use AskUserQuestion popup. Combine related questions into one popup — avoid multiple interruptions.

**Model question rule:** Do NOT ask about model by default. Only add a model question to the popup when user's description contains model keywords like 「SD」「MJ」「Flux」「SDXL」, or explicitly says 「我用的是xxx模型」.

**multiSelect rule:**
- Mutually exclusive choices (only one applies) → `multiSelect: false`
- Stackable/combinable choices (multiple can apply) → `multiSelect: true`

### Initial popup (4–5 questions):

```json
{
  "questions": [
    {
      "question": "你想要生成什么类型的图片？",
      "header": "图片类型",
      "options": [
        {"label": "海报/KV主视觉 — 宣传推广用", "description": "海报、主视觉图、宣传画面"},
        {"label": "插画/概念图 — 艺术创作", "description": "插画作品、概念设计、艺术创作"},
        {"label": "图标/UI元素 — 界面组件", "description": "图标、按钮、UI界面元素"},
        {"label": "产品/实物图 — 商品展示", "description": "产品展示、实物拍摄效果"}
      ],
      "multiSelect": false
    },
    {
      "question": "想要哪种画面风格？（只能选一种）",
      "header": "画面风格",
      "options": [
        {"label": "写实/摄影 — 真实照片质感", "description": "写实风格、摄影质感、真实感"},
        {"label": "3D渲染 — 立体感强，C4D/Blender风格", "description": "3D立体、C4D渲染、Blender、黏土/充气/毛绒等3D风格"},
        {"label": "插画/手绘 — 艺术绘画风格", "description": "插画、手绘、水彩、国风、扁平插画"},
        {"label": "动画/卡通 — Pixar/迪士尼/游戏风格", "description": "Pixar、迪士尼、游戏原画、卡通IP"}
      ],
      "multiSelect": false
    },
    {
      "question": "主色调方向？（只能选一种）",
      "header": "主色调",
      "options": [
        {"label": "温暖色调 — 橘黄暖调，温馨感", "description": "温暖、温馨、家庭氛围"},
        {"label": "冷色调 — 蓝青冷调，清新感", "description": "清新、凉爽、干净"},
        {"label": "深色/黑金 — 高级奢华感", "description": "深色背景、金色光效、高端质感"},
        {"label": "高对比 — 强烈视觉冲击", "description": "高对比、鲜艳、艺术感"}
      ],
      "multiSelect": false
    },
    {
      "question": "想要哪些氛围关键词？（可多选）",
      "header": "氛围调性",
      "options": [
        {"label": "科技感/未来感 — 现代科技风", "description": "科技、未来、现代"},
        {"label": "高级感/奢华感 — 精致高端", "description": "高级、奢华、精致"},
        {"label": "热闹/喜庆 — 节日促销氛围", "description": "热闹、喜庆、节日"},
        {"label": "清新/自然 — 轻盈干净", "description": "清新、自然、干净"}
      ],
      "multiSelect": true
    },
    {
      "question": "构图和视角有偏好吗？（可多选）",
      "header": "构图视角",
      "options": [
        {"label": "中心构图 — 主体居中，稳重大气", "description": "中心对称、主体突出"},
        {"label": "仰视/低角度 — 产品更有气势", "description": "仰视视角、广角感"},
        {"label": "大面积留白 — 简洁高级", "description": "留白、极简、呼吸感"},
        {"label": "俯视/平铺 — 全局展示", "description": "俯视、平铺、全景"}
      ],
      "multiSelect": true
    }
  ]
}
```

### Iteration Round 1 popup (first dissatisfaction):

```json
{
  "questions": [
    {
      "question": "整体感觉差在哪里？（可多选）",
      "header": "问题定位",
      "options": [
        {"label": "风格不对 — 画风/质感不是想要的", "description": "艺术风格偏差"},
        {"label": "主体不够突出 — 焦点不清晰", "description": "构图或主体表现问题"},
        {"label": "氛围不够 — 情绪感不足", "description": "整体氛围和情绪"},
        {"label": "细节太少 — 画面太空洞", "description": "细节丰富度不够"}
      ],
      "multiSelect": true
    },
    {
      "question": "主体哪些细节需要加强？（可多选）",
      "header": "主体细节",
      "options": [
        {"label": "外观/颜色更具体 — 描述更精准的外观特征", "description": "颜色、纹理、材质细节"},
        {"label": "姿态/动作更生动 — 更有张力的姿态表现", "description": "动作、姿势、表情"},
        {"label": "环境/背景更丰富 — 场景细节更多", "description": "背景元素、环境氛围"},
        {"label": "光影/氛围更强烈 — 光线和情绪感更突出", "description": "光效、阴影、氛围"}
      ],
      "multiSelect": true
    },
    {
      "question": "整体感觉期望更偏向哪种？（只能选一种）",
      "header": "期望方向",
      "options": [
        {"label": "更写实/真实 — 接近真实照片质感", "description": "写实摄影风格"},
        {"label": "更艺术/风格化 — 有明显的艺术处理", "description": "艺术化风格"},
        {"label": "更简洁/干净 — 减少复杂元素", "description": "极简风格"},
        {"label": "更丰富/细腻 — 增加更多细节层次", "description": "细节丰富"}
      ],
      "multiSelect": false
    }
  ]
}
```

### Iteration Round 2 popup (still unsatisfied):

```json
{
  "questions": [
    {
      "question": "最核心的问题是什么？（可多选）",
      "header": "核心问题",
      "options": [
        {"label": "主体描述还不够准确 — 需要更精细的主体刻画", "description": "主体细节不足"},
        {"label": "风格方向需要调整 — 整体风格偏差较大", "description": "风格需要重新定向"},
        {"label": "构图/视角需要改变 — 画面角度或布局问题", "description": "构图视角调整"},
        {"label": "色调/光影需要重来 — 颜色和光线完全不对", "description": "色彩光影重新设定"}
      ],
      "multiSelect": true
    },
    {
      "question": "构图上有什么具体要求？（可多选）",
      "header": "构图调整",
      "options": [
        {"label": "换视角 — 仰视/俯视/侧面等", "description": "视角调整"},
        {"label": "主体更大/更突出 — 占画面比例更高", "description": "主体比例"},
        {"label": "背景更简洁 — 减少干扰元素", "description": "背景简化"},
        {"label": "增加景深/虚化 — 前后层次感更强", "description": "景深效果"}
      ],
      "multiSelect": true
    },
    {
      "question": "光影和色彩方向？（只能选一种）",
      "header": "光影色彩",
      "options": [
        {"label": "更强烈的戏剧性光影 — 明暗对比更大", "description": "高对比光影"},
        {"label": "更柔和均匀的光 — 细节更清晰", "description": "柔和光线"},
        {"label": "换色调方向 — 整体颜色重新定", "description": "色调重设"},
        {"label": "增加发光/粒子效果 — 更有氛围感", "description": "光效增强"}
      ],
      "multiSelect": false
    }
  ]
}
```

## Reference Image Analysis

When user uploads an image or mentions 「参考这张」「类似这种」「照这个风格」:

### Analysis dimensions (analyze ALL of these):

| Dimension | What to extract |
|-----------|----------------|
| **Style** | Realistic / illustration / 3D render / cartoon / CGI / photography style |
| **Composition** | Subject placement, camera angle (eye-level/low/bird's eye), depth of field, negative space ratio |
| **Color palette** | Dominant colors (name them specifically), warm/cool tone, contrast level, saturation |
| **Subject** | Main subject description, how it's presented, key visual details |
| **Lighting** | Light source direction, quality (hard/soft/diffused), special effects (glow, rim light, volumetric) |
| **Atmosphere** | Overall mood, emotional tone, scene context |

### Output format after analysis:
```
参考图分析：
- 风格：[具体风格描述]
- 构图：[构图方式、视角]
- 色调：[主色调、冷暖、对比度]
- 光影：[光源、光效特点]
- 氛围：[整体情绪感]

我会基于这张参考图的风格来生成。你想要保留哪些元素？主体内容是什么？
```

### Rules:
- Extract style/composition/color/lighting keywords from the reference and embed them directly into the generated prompt
- If user only uploads image with no text → ask via popup: what subject do they want in this style?
- If user says 「类似这种但是要XXX」→ keep the style, replace the subject/content
- Reference image analysis replaces the need to ask about style/color in the initial popup (those dimensions are already known)

---


- Prompt MUST be in a code block
- Default: Chinese natural language, for 即梦/豆包/Flux
- 「SD」「SDXL」「SD1.5」→ English tag format + negative prompt
- 「MJ」「Midjourney」「Niji」→ English keyword format + `--ar` params
- No filler adjectives
- All enhancement terms merged into body, never listed separately

## Model Format Reference

**Natural language (即梦/豆包/Flux/Gemini — default):**
```
一只橘白色短毛猫，圆润的脸庞，琥珀色眼睛，慵懒地趴在窗台软垫上，阳光从侧面照射在毛发上呈现出细腻的金色光泽，写实摄影风格，温馨家庭氛围，浅景深，8K超清
```

**Tag format (SDXL/SD1.5):**
```
orange tabby cat, fluffy fur, amber eyes, lying on windowsill, warm sunlight, photorealistic, shallow depth of field, 8k uhd, masterpiece
Negative prompt: blurry, deformed, ugly, bad anatomy, watermark
```

**MJ format (Midjourney/Niji):**
```
orange tabby cat lying on windowsill, warm afternoon sunlight, photorealistic, cozy home atmosphere --ar 4:3 --v 6 --style raw
```
