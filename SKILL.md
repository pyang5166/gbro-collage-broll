---
name: gbro-collage-broll
description: 将约 5 秒口播文稿、观点句或抽象概念做成高级 editorial halftone paper-collage / 半调纸拼贴 B-roll。用户说“collage b-roll”“纸拼贴 b-roll”“半调拼贴”“拼贴风格配画面”“用这段文稿做拼贴动画”“gbro-collage-broll”，或希望把一句文稿转成拼贴视觉隐喻时，必须使用此 skill。强制采用三阶段审批：先只提视觉隐喻，用户确认后才生成彩色拼贴静帧，静帧再次确认后才默认调用 Gemini Omni Flash 生成首尾帧组装动画。默认视频模型固定为 gemini-omni-flash-preview，不再默认使用 Veo；只有用户明确指定其他模型时才切换。
compatibility: 支持 Codex 和 Claude Code。Gate 2 使用 Codex 内置 image_gen，或 Claude Code 中已安装的图片生成 skill（默认查找 image-gen）。视频生成另需 Python >= 3.10、google-genai >= 2.10.0、GEMINI_API_KEY，以及 ffmpeg / ffprobe。
---

# gbro Collage B-roll

把一句约 5 秒的口播压成一个 sharp visual idea，再做成高级编辑风纸拼贴组装动画。

默认链路：

1. 只设计视觉隐喻，等待用户确认
2. 只生成最终静帧，等待用户确认
3. 自动调用 Gemini Omni Flash 生成视频并完成 QA

这两个确认闸门是工作流的一部分。它们让用户把注意力放在审美和方向上，同时避免错误隐喻或错误静帧直接消耗视频生成成本。

## 首次使用：环境自检

每次触发本 skill 时，进入 Gate 1 之前先运行自检脚本：

```bash
bash <本skill目录>/scripts/check_setup.sh
```

全部通过则直接开始 Gate 1，不要向用户重复配置信息。任何一项失败时，视为首次使用：不进入 Gate 1，先向用户输出下面的配置指南（只列出缺失项），等用户确认配置完成后重新自检。

### 配置指南（按缺失项输出）

1. **GEMINI_API_KEY 未设置**
   到 [Google AI Studio](https://aistudio.google.com/apikey) 创建 API key，然后写入 shell 配置：

   ```bash
   echo 'export GEMINI_API_KEY="你的key"' >> ~/.zshrc && source ~/.zshrc
   ```

   注意：视频生成按量计费，由这个 key 对应的 Google 账号承担。

2. **ffmpeg / ffprobe 缺失**
   macOS：`brew install ffmpeg`；Debian/Ubuntu：`sudo apt install ffmpeg`。

3. **Python 环境缺失或版本过旧**（需要 >= 3.10）
   macOS：`brew install python3`；或从 python.org 安装。

4. **共享 venv 未创建**（google-genai >= 2.10.0）
   征得用户同意后自动创建，无需用户手动操作：

   ```bash
   python3 -m venv ~/hyperframes-projects/.omni-venv
   ~/hyperframes-projects/.omni-venv/bin/python -m pip install --upgrade "google-genai>=2.10.0" "httpx[socks]"
   ```

5. **没有可用的图片生成能力**
   Gate 2 需要以下任一能力：Codex 内置 `image_gen`；Claude Code 中已安装的
   `image-gen` skill；或用户明确指定的等效图片生成 skill。都不可用时，再请用户
   安装图片生成 skill 或手动提供静帧图片。

## 强制审批协议

### Gate 1：隐喻确认

收到文稿后，先提视觉隐喻，不生成图片、不生成视频、不调用任何视频模型。

向用户交付每条的：

- 核心意思
- 情绪
- 一句话视觉命题
- 3–6 个关键物件
- 建议底色与局部点色
- 预期组装顺序

然后明确停下，等待用户回复“可以”“通过”“全部通过”或给出逐条修改意见。

如果用户只确认部分编号，只让通过的条目进入 Gate 2；未通过条目继续修改隐喻。

### Gate 2：静帧确认

隐喻确认后，才写 visual spec 和 imagegen prompt，并按当前 agent 选择图片生成方式：

1. Codex：使用内置 `image_gen`（或本地 `imagegen` skill 规定的方式）。
2. Claude Code：用 Skill 工具调用已安装的 `image-gen`；读取并严格遵守它自己的
   `SKILL.md`，把英文 prompt 和项目内输出路径传给它。不要猜测或写死其脚本路径。
3. 用户明确指定了其他图片生成 skill：遵守用户指定。

如果 Claude Code 找不到 `image-gen`，先检查是否存在
`~/.claude/skills/image-gen/SKILL.md`。不要因为没有 Codex 内置工具就终止流程。

把原图保存到项目目录，生成带编号的静帧 contact sheet，向用户展示并再次停下。此阶段仍然不调用 Omni Flash，也不生成视频。

如果用户只确认部分静帧，只让通过的条目进入 Gate 3；需要修改的静帧先重生并重新确认。

### Gate 3：视频生成

静帧确认后，不再询问使用哪个视频模型，直接使用本 skill 自带的 `scripts/generate_video.py`，默认调用：

```text
gemini-omni-flash-preview
```

只有用户明确指定其他视频模型时，才覆盖这个默认值。不要自动调用 Veo，也不要把模型选择再抛给用户。

## 成功标准

- 一句话只表达一个清晰隐喻
- 同一批画面有统一设计语言，但不强制全部蓝底
- 背景是强烈、平坦、均匀的色场，可按语意变化
- 主体以黑白 halftone photographic cut-outs 为骨架
- 关键卡片、按钮、胶片、规则册等允许使用红、黄、青、橙、紫、奶油白等彩色纸张
- 所有纸片有清晰裁切边、奶油白 keyline、低透明度柔和阴影和纸张颗粒
- 动作是 assemble-from-empty，而不是轻微漂移、晃动或慢 zoom
- 无字幕、无口播全文、无 logo、无水印、无 UI
- 默认交付 9:16、5 秒、720×1280、24fps、无声 MP4

## 什么时候不要用

- 需要精确控制图层、遮挡、镜头穿越或可编辑时间线：改用分层动画工具（如 HyperFrames 类 HTML 渲染视频方案）
- 只需要视频提示词，不需要生成成片：直接写 prompt 即可，不用走本流程
- 需要真实人物产品广告或口播演员：不要走本拼贴流程
- 用户明确要可逐层修改的透明素材：本 skill 默认不拆透明图层

## 默认项目目录

使用北京时间 `Asia/Shanghai` 命名：

```text
~/hyperframes-projects/YYYY-MM-DD-collage-broll-标题/
```

批量项目推荐结构：

```text
<project>/
├── brief.md
├── visual-spec.json
├── imagegen-prompts.md
├── omni-jobs.json
├── gate2-qa.md
├── gate3-qa.md
├── still-contact-sheet.jpg
├── omni-contact-sheet-all.jpg
├── video-first-frame-all.jpg
├── end-frame-comparison-all.jpg
├── 01-概念名/
│   ├── omni-prompt.txt
│   ├── frames/
│   │   ├── last-frame-original.png
│   │   ├── first-frame.png
│   │   └── last-frame.png
│   └── omni/run-v01/
│       ├── final-5s.mp4
│       ├── final-5s-noaudio.mp4
│       ├── contact-sheet.jpg
│       ├── video-last-frame.jpg
│       └── end-frame-comparison.jpg
└── 02-概念名/...
```

## Phase 1：设计视觉隐喻

先把文稿压成一个视觉命题。

提取：

- 核心意思：观众最终要看懂什么
- 情绪：冷静、惊讶、紧迫、豁然开朗、荒诞、反讽
- 动作动词：打开、连接、漏掉、装订、归档、点亮、压缩、分叉、组装
- 可视化隐喻：机器、时钟、胶片、档案柜、控制台、规则册、漏斗、轨道、棋子

不要把文稿逐字放进画面。默认一条文稿只做一个隐喻，控制在 3–6 个关键物件；元素过多会让语意变弱，也会让 Omni 组装不稳定。

批量隐喻优先形成前后叙事：例如先表现手工消耗与经验流失，再表现规范沉淀与人机分工。

### Gate 1 输出示例

```text
1. 核心意思：经验每次都在重复消耗
   视觉隐喻：熟练剪辑师围着巨大的胶片时钟逐帧裁切，时钟走完一圈却只得到一小段成片
   关键物件：胶片时钟、剪辑师、剪刀、短胶片
   色彩：焦橙底，奶油白与浅青点色
   组装顺序：时钟 → 人物与剪刀 → 胶片 → 最终短输出
```

输出后停下等待确认。

## Phase 2：生成彩色拼贴静帧

隐喻确认后，先写自包含的 `visual-spec.json`，再写 imagegen prompt。

### Visual spec

```json
{
  "script_meaning": "",
  "visual_metaphor": "",
  "style_signature": "flat bold color field, mixed black-and-white halftone cut-outs and colored cardstock accents, crisp cut edges, cream keylines, soft paper shadows, editorial paper collage",
  "aspect_ratio": "9:16",
  "color_field": {
    "background_hex": "",
    "accent_colors": [],
    "paper_grain": "fine uncoated-paper fiber"
  },
  "elements": [
    {
      "what": "",
      "role": "",
      "motion": "",
      "placement": ""
    }
  ],
  "composition": {
    "layout": "",
    "negative_space": "",
    "final_frame": ""
  },
  "motion_plan": "structure first, subject or cards second, action and result last",
  "avoid": "typography, readable letters, numerals, logos, watermark, UI, subtitles, glossy 3D, photoreal environment"
}
```

### 色彩规则

不要把 cobalt blue 当成唯一默认值。根据语意挑选强色场，并在一批作品中保持“同设计语言、不同底色”：

- 焦橙 / 红：时间消耗、劳动、紧迫
- 芥末黄：工具、警示、经验漏失
- 墨绿：认知、审美、系统重置
- 深紫：规范、沉淀、长期记忆
- 青绿：判断、协作、自动执行

主体可以黑白半调为主，但局部彩色纸张必须服务信息层级，不要为了彩色而彩色。

### Imagegen prompt 模板

使用上面选定的图片生成方式：

```text
Use case: ads-marketing
Asset type: final still frame for a 9:16 image-to-video B-roll clip
Primary request: Create a finished editorial paper-collage image expressing [一句话视觉命题].
Scene/backdrop: perfectly flat [颜色] paper field [hex] with subtle uncoated paper fiber.
Style/medium: premium editorial stop-motion paper collage; black-and-white halftone photographic cut-outs mixed with selective [点色] colored cardstock.
Composition/framing: vertical 9:16 locked poster frame; central subject within the middle 70 percent; generous clean color-field negative space; 3–6 large separable paper groups for later assemble-from-empty animation.
Materials/textures: visible printed halftone dots, crisp machine-cut edges, thin warm-cream paper keylines, soft low-opacity physical drop shadows.
Constraints: [本条隐喻必须一眼看懂的关系].
Avoid: no typography, no readable letters, no numerals, no logos, no watermark, no UI, no subtitles, no glossy 3D, no photoreal environment, no clutter.
```

### 静帧 QA

- 隐喻是否一眼看懂
- 主体是否集中
- 是否有假字、logo、水印或 UI
- 是否保留足够纯色场，方便从空场组装
- 是否是 3–6 个清晰大组，而不是满屏碎片
- 同一批是否统一质感但有色彩变化

将通过 QA 的原图复制到项目目录，生成带编号的静帧 contact sheet，展示给用户并停下等待 Gate 2 确认。静帧 QA 结论写入 `<project>/gate2-qa.md`。

如果用户要求重生部分静帧，重生后生成 `still-contact-sheet-v2.jpg`（后续轮次递增 v3、v4…），保留旧版 contact sheet 不覆盖，方便对比。

## Phase 3：用 Omni Flash 生成视频

### 1. 准备首尾帧

保留 imagegen 原图，再统一尾帧：

```bash
ffmpeg -y -i <item>/frames/last-frame-original.png \
  -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
  <item>/frames/last-frame.png
```

首帧默认是与尾帧相同底色的纯色空纸面：

```bash
ffmpeg -y -f lavfi -i color=c=0x<HEX>:s=1080x1920 \
  -frames:v 1 <item>/frames/first-frame.png
```

如果用户明确要求不从完全空白开始，首帧才保留一个基础物件。

### 2. 写 Omni 动画 prompt

动作顺序默认采用：

```text
基础结构 → 人物或关键卡片 → 连接件 → 动作 → 最终结果
```

Prompt 模板：

```text
Paper-collage stop-motion assembly, using Image 1 as the exact empty first frame and Image 2 as the exact completed last frame. In one continuous locked-off vertical shot, open on the empty flat [color] paper field.

Assemble the scene piece by piece with crisp physical stop-motion timing: [按顺序描述 3–6 个元素如何滑入、卡位、连接和完成动作]. End by holding the supplied completed composition.

Preserve the exact 9:16 framing, [hex] color field, colored cardstock accents, uncoated paper grain, halftone dots, cream keylines, crisp cut edges and soft shadows. Restrained tactile 2D paper craft only.

No scene cuts, no camera movement, no zoom, no morphing, no new objects, no text, no letters, no numbers, no logos, no watermark, no UI, no sound.
```

每条 prompt 都要明确 Image 1 是空首帧、Image 2 是确认过的完成帧。最终构图必须贴近 Image 2，不让模型自由改造尾帧。

### 3. 检查 Omni 运行环境

使用实际运行脚本的 Python 解释器检查 `google-genai` 版本。需要 `>= 2.10.0`。

如果系统 Python 版本过旧或属于 externally managed environment，不要使用 `--break-system-packages`。使用共享隔离环境 `~/hyperframes-projects/.omni-venv/`——已存在就直接复用，只在不存在时创建，不要在每个项目内新建 `.venv`：

```bash
[ -x ~/hyperframes-projects/.omni-venv/bin/python ] || {
  python3 -m venv ~/hyperframes-projects/.omni-venv
  ~/hyperframes-projects/.omni-venv/bin/python -m pip install --upgrade "google-genai>=2.10.0" "httpx[socks]"
}
```

确认 `GEMINI_API_KEY` 已设置，但不要输出或记录密钥内容。

### 4. 批量调用 Gemini Omni Flash

创建 `omni-jobs.json`。每个 job 使用两张图片做关键帧插值：

```json
{
  "prompt": "<omni prompt>",
  "image": [
    "<item>/frames/first-frame.png",
    "<item>/frames/last-frame.png"
  ],
  "output": "<item>/omni/run-v01/final-5s.mp4",
  "aspect_ratio": "9:16",
  "duration": 5
}
```

使用本 skill 自带脚本：

```bash
~/hyperframes-projects/.omni-venv/bin/python \
  <本skill目录>/scripts/generate_video.py \
  --batch <project>/omni-jobs.json \
  --concurrency 3
```

脚本默认模型即 `gemini-omni-flash-preview`。如果出现 legacy Interactions API schema 错误，说明用错了旧 SDK；切换到共享环境 `~/hyperframes-projects/.omni-venv/bin/python` 后重试，不要退回 Veo。

### 5. 强制无声交付

即使 prompt 已写 `no sound`，仍用 ffmpeg 输出零音轨版本：

```bash
ffmpeg -y -i <run>/final-5s.mp4 \
  -map 0:v:0 -c:v copy -an \
  <run>/final-5s-noaudio.mp4
```

默认交付 `final-5s-noaudio.mp4`，保留原始 `final-5s.mp4` 作为中间产物。

## 视频 QA

不要只看尾帧，必须检查组装过程和最终落位。

### Contact sheet

```bash
ffmpeg -y -i <run>/final-5s-noaudio.mp4 \
  -vf "fps=1,scale=270:480,tile=5x1" \
  -frames:v 1 <run>/contact-sheet.jpg
```

通过标准：

- 首帧接近纯色空场；边缘轻微提前露出纸片可以接受
- 中段能看到结构、人物或卡片逐步进入，而不是整体淡入
- 没有切镜、zoom、3D 化或写实场景漂移
- 没有假字、logo、水印或 UI
- 最终帧与确认静帧一致；轻微姿态或细节漂移（如人物姿势微变、小零件增减）只要不影响隐喻语义即可判通过，不要为此重跑
- 成片为 720×1280、24fps、5 秒、零音轨

另外抽取视频末帧，与确认静帧并排生成 `end-frame-comparison.jpg`。批量项目再合并三张总览图：

- `omni-contact-sheet-all.jpg`：全部成片逐秒抽帧
- `video-first-frame-all.jpg`：全部成片实际首帧，验证真的从空色场开始
- `end-frame-comparison-all.jpg`：确认静帧与视频末帧并排对照

逐条 QA 结论（含带瑕疵通过的判定理由）写入 `<project>/gate3-qa.md`。

### 常见问题

- 首帧边缘提前露出：轻微可接受；严格空场需求改用 HyperFrames 补前段
- 组装感弱：缩短元素数量，并把 prompt 改为明确的逐件 slide in / snap into place 顺序
- 尾帧漂移：强化 “Image 2 is the exact completed last frame” 和 “End by holding the supplied completed composition”
- 出现假字：先回到静帧重生，不要直接用视频 prompt 修补
- 个别视频失败：只重跑对应 job，不要重跑已经通过的条目

## 默认交付

向用户交付：

- 每条 `<item>/omni/run-v01/final-5s-noaudio.mp4`
- 每条 contact sheet
- 批量总 contact sheet
- 最终帧对照图
- 一句说明每条文稿如何转成视觉隐喻

如果成片问题来自 Omni 的快速生成限制，直接说明；只有需要精确图层控制时，才建议切换到 HyperFrames。

## 旧 Veo 脚本

目录中的 `scripts/generate_veo_first_last.py` 仅为旧项目兼容保留。不要在默认流程中调用它。只有用户明确要求 Veo 时才使用。
