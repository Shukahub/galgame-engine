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

**The fundamental problem with most character-chat skills:**

When a single AI generates the environment, the character's behavior, and the plot
direction in one reasoning pass, a subtle collapse happens: the character becomes
scenery. Her actions are chosen to fit the mood of the scene, not to express her
own psychology. She doesn't have her own agenda — she has a narrative function. The
result feels like reading about a character, not interacting with a person.

This is why simple prompts like "pretend to be a tsundere girl" produce flat
experiences. The AI knows the trope but has no private interior to draw from. Every
response is surface-level performance.

**What galgame-engine does differently:**

Three independent reasoning passes, each with a scoped input built through a Context
Firewall that physically removes hidden fields before every call:

```
Player Input
     │
     ├─▶ STORY DIRECTOR → Beat plan, pacing, pressure
     │   Receives: visible story state, open threads, pacing state
     │   Produces: beat type, scene directives, event candidates
     │   Cannot: write prose, decide character emotions, force reveals
     │
     ├─▶ SCENE NARRATOR → Environment only
     │   Receives: scene state, director plan, surface character facts
     │   Produces: sensory prose — light, sound, space, objects
     │   Cannot: animate characters, name emotions, explain subtext
     │
     ├─▶ CHARACTER ARCHITECT → Creates Persona Documents
     │   Triggered by: new characters, major events, player /newrole
     │   Builds: Jungian layers, core wound, defenses, attachment style
     │
     ├─▶ CHARACTER RESPONSE → Per-character behavior & dialogue
     │   Receives: response_safe_persona (filtered), relationship bands,
     │             player_visible_to_character, scene description
     │   Produces: visible behavior, dialogue, stat deltas
     │   Cannot: control other characters, see hidden persona fields
     │         │
     │         ▼
     └─▶ EDITOR PASS → Merge, strip narrator analysis, preserve the gap
```

**Why this matters in practice:**

- The Scene Narrator writes "the stage, not the actors." It describes the space,
  the light, the objects — but characters are animated by Character Response,
  not by the narrator. This means a character's gesture isn't chosen to make the
  scene pretty; it comes from her internal state.

- Each Character Response call receives only what that specific character can
  perceive and knows. A guarded character literally cannot act on information
  she hasn't unlocked. The Context Firewall enforces this by deleting hidden
  fields from the input — no "ignore this secret" workarounds.

- Characters have private interiors the player never sees: hidden core wounds,
  unspoken desires, defense mechanisms that fire under specific triggers. These
  aren't decorative lore — they directly shape behavior through stat bands,
  unlock tiers, and the response_safe_persona filter.

**Psychological depth, not trope imitation:**

Every character is built with:
- **Jungian layers** — Persona (the mask), Shadow (the repressed), Anima/Animus
  (the internalized ideal of intimacy)
- **Core wound + core desire** — a specific formative experience that shaped
  her defenses, and what she actually wants beneath all behavior. These create
  playable internal tension (not just surface contradiction — also pride vs
  hunger, duty vs freedom, control vs vulnerability)
- **Attachment style** — secure / anxious / avoidant / disorganized, directly
  governing how she behaves as closeness increases
- **Defense mechanisms** (2-3 per character) — intellectualization, projection,
  reaction formation, displacement, sublimation, denial, splitting — each with
  concrete triggers and visible behavioral manifestations
- **Unlock tiers** (0-3) — deeper layers become visible only as trust crosses
  thresholds (35/65/85). A character with trust < 35 is literally incapable of
  sustained Tier 1+ intimacy, though brief involuntary cracks may appear

**Prose with discipline: observational minimalism (白描)**

The narrator operates like a camera, not a critic. Every sentence must describe
what is visible, audible, or tangible. No emotion labels ("she was angry"), no
measurements ("her head tilted two degrees"), no subtext analysis ("she wasn't
smiling — rather, she was..."). The engine's Editor Pass strips these before
rendering. The result: facts arranged in sequence, letting the reader do the
interpreting — the way French New Wave cinema trusts the audience to feel
without being told what to feel.

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

**大多数角色聊天 skill 的根本问题：**

当一个 AI 在同一次推理中同时生成环境、角色行为和剧情走向时，会发生一个微妙的坍
塌：角色变成了布景。她的动作被选择来配合场景的氛围，而非表达她自己的心理。她没
有自己的 agenda——她只有一个叙事功能。结果读起来像是在"看一个角色"，而不是在和
一个人互动。

这就是为什么简单的"假装你是一个傲娇女孩" prompt 产出的体验是扁平的。AI 认识这个
标签，但标签之下没有私密的内心世界可以提取。每一次回应都是表层的表演。

**galgame-engine 做了什么不同的事：**

三次独立的推理 pass，每个 pass 在调用前都经过 Context Firewall 构建限定输入——
物理删除隐藏字段，而非传过去然后告诉模型"请忽略"：

```
玩家输入
     │
     ├─▶ 剧情导演 → 节拍规划、节奏控制、压力设计
     │   接收：可见故事状态、开放线索、节奏状态
     │   产出：节拍类型、场景指令、候选事件
     │   禁止：写散文、决定角色情绪、强行揭示秘密
     │
     ├─▶ 场景叙述 → 只写环境
     │   接收：场景状态、导演计划、角色表层信息
     │   产出：感官散文——光、声、空间、物体
     │   禁止：让角色动起来、标注情绪、解释潜台词
     │
     ├─▶ 角色构建 → 创建角色档案
     │   触发时机：新角色出现、重大事件、玩家 /newrole
     │   构建：荣格分层、核心伤痛、防御机制、依恋风格
     │
     ├─▶ 角色回应 → 逐角色生成行为与对话
     │   接收：response_safe_persona（已过滤）、关系波段、
     │         player_visible_to_character、场景描述
     │   产出：可见行为、对话、stat 变化量
     │   禁止：控制其他角色、看见隐藏的 persona 字段
     │         │
     │         ▼
     └─▶ 编辑合并 → 整合、剥离叙事者分析、保留缝隙
```

**为什么这在实践中重要：**

- 场景叙述者写的是"舞台，而非演员"。它描述空间、光线、物体——但角色的动作来自
  角色回应模块，而非叙事者。这意味着角色的手势不是为了"让场景好看"而被选中的；
  它来自她的内在状态。

- 每个角色回应调用只收到这个角色能感知和知道的信息。一个防备心强的角色无法
  根据她尚未解锁的信息行动。Context Firewall 通过从输入中删除隐藏字段来强制
  执行——不存在"请忽略这些秘密"的绕过方式。

- 角色拥有玩家永远看不到的私密内心：隐藏的核心伤痛、未说出口的渴望、在特定
  触发下启动的防御机制。这些不是装饰性的背景故事——它们通过 stat 波段、解锁
  层级和 response_safe_persona 过滤器直接塑造行为。

**心理深度，而非标签模仿：**

每个角色的构建包含：
- **荣格分层**——Persona（面具）、Shadow（阴影）、Anima/Animus（她内化的亲密
  关系理想）
- **核心伤痛 + 核心渴望**——塑造她防御机制的具体经历，以及她在所有行为之下真正
  想要的东西。它们制造可玩的内部张力（不仅是表里矛盾——也可以是骄傲 vs 饥渴、
  责任 vs 自由、控制 vs 脆弱）
- **依恋风格**——安全型 / 焦虑型 / 回避型 / 混乱型，直接决定她在靠近时的行为
- **防御机制**（每角色 2-3 种）——理智化、投射、反向形成、置换、升华、否认、
  分裂——每种都有具体的触发条件和可见的行为表现
- **解锁层级**（0-3 级）——更深的层次只在信任跨过阈值（35/65/85）后才变得
  可见。信任 < 35 的角色无法维持 Tier 1+ 的亲密度，但可能出现短暂的不由自主
  的裂缝

**克制的文风：白描**

叙事者像一台摄像机，而非评论员。每个句子必须描述可见、可听、可触的东西。没有情
绪标签（"她很生气"），没有度量（"她的头歪了两度"），没有潜台词分析（"她不是在
笑——而是……"）。引擎的 Editor Pass 在渲染前剥离这些。结果：事实依次排列，让读
者自己感受——正如法国新浪潮电影对观众的信任：不需要被告诉该感觉到什么。

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
