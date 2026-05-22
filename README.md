# galgame-engine

A Claude Code skill for AI-driven visual novel / galgame sessions with multi-module
architecture, psychological character depth, and observational-minimalist prose.

[English](#english) | [中文](#中文)

---

## English

### What is this?

An interactive narrative engine that runs inside Claude Code. Unlike simple "chat with
character" prompts, galgame-engine uses a **three-module architecture** where story
direction, scene narration, and character response are separate concerns — each with
isolated inputs built through a Context Firewall that removes hidden fields before
every pass.

### Why it's different

Most character-chat skills have one AI generating everything at once. The environment
description, the character's behavior, and the plot direction all come from the same
reasoning pass. This means characters become elements of scene composition — their
actions serve narrative convenience rather than arising from independent psychology.

galgame-engine fixes this:

```
Player Input
     │
     ├─▶ STORY DIRECTOR → Beat plan, pacing, pressure (no prose, no dialogue)
     │
     ├─▶ SCENE NARRATOR → Environment only — light, sound, space (stage, not actors)
     │
     ├─▶ CHARACTER ARCHITECT → Creates/updates Persona Documents (triggered on new characters)
     │
     ├─▶ CHARACTER RESPONSE → Per-character behavior & dialogue (called once per character)
     │         │
     │         ▼
     └─▶ EDITOR PASS → Merge, strip narrator analysis, preserve the gap between stage & actors
```

**Characters are built with psychological depth:**
- Jungian layers (Persona / Shadow / Anima-Animus)
- Core wound + core desire (creating playable internal tension)
- Attachment style (governing approach/avoidance curves)
- Defense mechanisms (triggered by specific situations)
- Unlock tiers (0-3), revealing deeper layers as trust builds

**Prose style: observational minimalism (白描).**
The narrator acts like a camera — recording what is visible, audible, tangible. No
emotion labels, no subtext explanation, no "she didn't X, she Y" analysis. Facts are
arranged in sequence; the reader does the interpreting.

### Installation

```bash
git clone https://github.com/Shukahub/galgame-engine.git ~/.claude/skills/galgame-engine
```

Restart Claude Code, or run `/plugin` to load the skill.

### Quick Start

Start a conversation with Claude Code and say something like:

> 开始一个新的galgame

The engine will ask for minimal setup (your character name, world preference) and
begin with an opening scene. From there, you play by typing your character's actions
and dialogue directly.

### Meta Commands

| Command | Purpose |
|---------|---------|
| `/pause` | Pause story, enter meta mode |
| `/resume` | Resume gameplay |
| `/status` | Show scene, relationships, open threads |
| `/profile` | View or edit protagonist profile |
| `/memory` | Show recent events and character impressions |
| `/tone` | Adjust tone preferences |
| `/debug` | Show engine-private state |
| `/save` / `/load` | Persist or restore session |
| `/newrole <desc>` | Create and introduce a new character |
| `/newrole queue <desc>` | Queue a character for later introduction |

### Design Principles

1. **Scene logic, character design, and character response are separate concerns.**
   Characters react from their own persona documents, relationship state, and recent
   events — not from narrative convenience.

2. **Physical removal beats instruction.** Hidden content is deleted from inputs before
   each pass, not passed with "ignore this" labels.

3. **Characters are the center of their own experience.** Each has preferences,
   irritations, and an agenda independent of the player. Silence, deflection, and
   changing the subject are valid responses.

4. **Show, don't explain.** The narrator describes what is observable. No measurements
   in degrees, no "she wasn't X-ing, she was Y-ing," no symbolic interpretation.

5. **Trust must be earned.** Charm and persistence alone don't raise trust. It rises
   from vulnerability protected, boundaries respected, promises kept, shared risk
   survived, dignity preserved.

---

## 中文

### 这是什么？

一个运行在 Claude Code 内部的交互式叙事引擎。与简单的"和角色聊天" prompt 不同，
galgame-engine 使用**三模块分离架构**——剧情导演、场景叙述、角色反应是三个独立的
关注点，每个模块在调用前都经过 Context Firewall 剥离隐藏信息。

### 为什么不同

大多数角色聊天 skill 让一个 AI 同时生成所有内容。环境描写、角色行为、剧情走向全
来自同一次推理。这意味着角色变成了场景构图的元素——她们的行为服务于叙事便利，而
非来自独立的心理驱动。

galgame-engine 的解决方案：

```
玩家输入
     │
     ├─▶ 剧情导演 → 节拍规划、节奏控制、压力设计（不写散文，不写对话）
     │
     ├─▶ 场景叙述 → 只写环境——光、声、空间（写舞台，不写演员）
     │
     ├─▶ 角色构建 → 创建/更新角色档案文档（新角色出现时触发）
     │
     ├─▶ 角色回应 → 逐角色生成行为与对话（每个角色独立调用一次）
     │         │
     │         ▼
     └─▶ 编辑合并 → 整合、剥离叙事者分析、保留舞台与演员之间的缝隙
```

**角色构建以心理学为基础：**
- 荣格分层（人格面具 / 阴影 / 阿尼玛-阿尼姆斯）
- 核心伤痛 + 核心渴望（制造可玩的内在张力）
- 依恋风格（决定她对靠近的反应曲线）
- 防御机制（被具体情境触发的行为规则）
- 解锁层级（0-3 级），信任越高，越深的层次才会暴露

**文风：白描。**
叙事者像一台摄像机——记录看到的、听到的、能触摸到的。不标注情绪，不解释潜台词，
不写"不是X，而是Y"。事实依次排列，读者自己感受。

### 安装

```bash
git clone https://github.com/Shukahub/galgame-engine.git ~/.claude/skills/galgame-engine
```

重启 Claude Code，或运行 `/plugin` 加载 skill。

### 快速开始

在 Claude Code 对话中说：

> 开始一个新的galgame

引擎会询问最低限度的设定（角色名、世界观偏好），然后展示开场场景。接下来，你直
接输入角色的行动和对话即可游玩。

### Meta 命令

| 命令 | 用途 |
|------|------|
| `/pause` | 暂停故事，进入 meta 模式 |
| `/resume` | 恢复游玩 |
| `/status` | 查看当前场景、关系、开放线索 |
| `/profile` | 查看或修改主角档案 |
| `/memory` | 查看近期事件和角色印象 |
| `/tone` | 调整基调偏好 |
| `/debug` | 查看引擎内部状态 |
| `/save` / `/load` | 保存/读取会话 |
| `/newrole <描述>` | 创建并引入新角色 |
| `/newrole queue <描述>` | 创建角色但延后引入 |

### 设计原则

1. **场景逻辑、角色设计、角色反应是三个独立关注点。** 角色的反应来自她自己的
   档案文档、关系状态和近期事件——而非叙事便利。

2. **物理移除优于指令绕过。** 隐藏内容在每次调用前从输入中删除，不传递带
   "请忽略这段"标签的完整档案。

3. **每个角色都是自己经验世界的中心。** 她有独立于玩家的偏好、烦躁和 agenda。
   沉默、回避、岔开话题都是合法反应。

4. **展示，不解释。** 叙事者只描述可观察的现象。不用度数丈量动作，不写
   "不是X，而是Y"，不做象征解读。

5. **信任必须被赢得。** 魅力和坚持本身不增加信任。信任来自：保护了她的脆弱、
   尊重了她的边界、兑现了承诺、共同经历了风险、维护了她的尊严。

### 文件结构

```
galgame-engine/
├── SKILL.md              # 主入口，包含核心规则和质量约束
├── references/
│   ├── module-prompts.md      # 所有模块的完整 prompt 契约
│   ├── persona-schema.md      # 角色档案完整 JSON schema
│   ├── world-state-schema.md  # 世界状态 schema、运行时模式
│   ├── meta-commands.md       # / 命令路由与处理规则
│   ├── style-guide.md         # 白描风格详细指南
│   └── overview.md            # 架构全景（中文）
└── LICENSE
```

### License

MIT — see [LICENSE](LICENSE).

---

*Built for players who want characters that feel real, prose that breathes, and
relationships that must be earned.*
