# Codex Skills 输入要求说明

本目录下的三个 skill 分别用于生成不同 AI 模型的英文提示词。以下按技能说明各自的功能与输入要求。

| Skill | 用途 | 输入 | 输出 |
| --- | --- | --- | --- |
| anima-prompt-writer | Anima 文生图 | 中文描述、灵感或标签 | 英文+中文提示词两段 |
| minimax-h3-video-prompt | MiniMax H3-Base 视频生成（T2VA/I2VA/FL2VA/L2VA） | 参考帧、时长、场景/角色、动作、对白、声音 | 纯英文提示词（指令行+3 个字段） |
| minimax-ref2va-prompt | MiniMax H3-Base Ref2VA 视频编辑（主体替换） | 参考图标签（1-2 张）+ 最终场景描述 | 纯英文 6 块 Context-IR 提示词 |

## 1. anima-prompt-writer

**功能**：将中文描述、灵感或标签润色并翻译成 Anima 文本到图像大模型（主打动漫、插画风格）适用的英文自然语言提示词。

**输入要求**：
- 可以是中文描述、灵感或标签，两者都可混合提供。
- 可带前置标签（角色/作品/基础特征），例如：`kal'tsit $arknights$, arknights, 1girl, cat girl, cat ears`。
- 也可以是纯文字描述；没有标签时直接转化为纯英文自然语言，不会强行造标签。

**输出规则**：
- 固定格式：【英文提示词】与【中文提示词】两段，两段内容一一对应。
- 若输入带标签，标签原样保留在英文/中文段最开头，标签与自然语言之间空一行。
- 英文自然语言至少两句，人物动作、姿态、特征占比至少 70%，背景与光影仅作物理衬托、极度精简。
- 禁止抽象/感性/华丽形容词（如 beautifully、dreamy），只用客观物理视觉词汇。
- 禁止画质词、评分词（如 masterpiece、best quality）与负面提示词。
- 特殊模式：说"一句话描述 xxx"（背景/动作/场景）时，直接给出一句英文描述及中文对照；背景类以 "The background is ..." 开头，不套用完整两段格式。

## 2. minimax-h3-video-prompt

**功能**：将中文输入（参考帧、视频时长、场景与角色描述、动作、对白、声音）转换为 MiniMax H3-Base 视频生成的自然英文提示词，支持 T2VA / I2VA / FL2VA / L2VA 四种模式。

**输入要求**（中文即可，英文也可）：
- 参考帧：无参考帧（T2VA）、只有首帧（I2VA）、首帧+末帧（FL2VA）、只有末帧（L2VA）。
- 时长：显式总时长（如 5 秒），或时间轴（如 0–2s、2–5s，取最大结束时间为总时长）。两者都未提供时，会先问一个问题再输出，不会猜。
- 场景与角色描述、动作、镜头切换/运镜要求。
- 对白/演唱（保持原语言，一字不改）、环境声音与配乐要求。

**输出规则**：
- 只输出英文；第一行是模式对应的指令行，空一行后依次输出三个核心字段：`integrated_multimodal_description`、`overall_soundscape`、`non_diegetic_music`。
- `[Shot 1]` 不加时间戳；后续镜头以严格递增的切点时间开头（如 `At 00:03.500`）。
- 运镜按"类型 + 幅度 + 速度"书写（如 push in、pan、truck、arc 等）。
- 对白使用稳定说话人 ID（如 `(S1)`），内容放在 `<d>[语言] 原文</d>` 标签内，原文保留、不翻译。
- 屏幕文字（横幅、招牌、字幕等）原样放在英文双引号内。

## 3. minimax-ref2va-prompt

**功能**：将参考图标签与最终场景描述转换为 MiniMax H3-Base Ref2VA 全参考模式（本地 H3-Base 或 ComfyUI 执行）的严格 6 块英文 Context-IR 提示词，用于把源视频中的主体替换为参考图定义的角色。

**输入要求**：
- 参考图标签：描述角色外观的标签，对应 1 或 2 张参考图；角色外观以此为准。
- 最终场景描述：目标环境、动作、运镜以及目标视觉风格。
- 输入缺失、不完整或存在歧义时，会先向你确认，不会自动补全。

**输出规则**：
- 纯英文、单一代码块，严格按以下 6 块顺序输出：`subject_definitions`、`summary`、`retention_analysis`、`detailed_description`、`overall_soundscape`、`non_diegetic_music`。
- 索引从 1 开始：`<Video 1>`、`<Picture 1>`；只有提供第二张图时才出现 `<Picture 2>`。
- `summary` 必须以 `[video editing + reference generation] The target video is an edited version of <Video 1>.` 开头。
- `detailed_description` 第一句声明目标视觉风格，`[Shot 1]` 无时间戳；主体必须执行动作并带动 1-2 个具体物理特征（如头发飘动），运镜按"类型 + 幅度 + 速度"融入。
- 标签与场景冲突时：角色外观以标签为准，场景描述决定环境、动作与运镜。
- 纯音频/无替换任务时，在 `subject_definitions` 中写 `no replacement`，并省略主体替换逻辑。
