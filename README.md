# galgame-engine

A Claude Code skill for AI-driven visual novel sessions — built on the belief that
good stories emerge from constraints, not from narrative manipulation.

[中文](#中文) | [English](#english)

---

## 中文

### 我为什么做这个

我喜欢平实的、真实的叙事。

我认为一个好的故事不是被"编排"出来的——它是在一组约束条件下自然推演出的结果。
给定一个大环境，给定当前状况，给定每个人物不可违背的设定，让事件自然发生。故事
的技巧不在煽情，而在安排：事件的时机、人物之间的碰撞、那些看似平常但实际作用很
大的瞬间。

我喜欢法国新浪潮电影。它们的镜头语言平实，没有煽情导向。导演拍下女人推开窗户，
然后切到桌上的一杯水——他相信观众有足够的观察力，能从这些克制的、客观的镜头里
感受到人物的情感。这是对观众的尊重。

但这种审美在传统 AI 角色扮演中几乎不可能实现。

### 传统 AI Galgame 的缺陷

几乎所有的 AI 角色扮演 skill 和 prompt 都有同一个结构性问题：**环境描写、角色
行为和剧情推进由一个 AI 在同一次推理中完成。**

这导致了一个致命的后果：角色失去了主体性。

当叙事者和角色控制器是同一个思维过程时，角色的行为会被叙事逻辑"绑架"。她的动作、
她的台词、她的情绪反应——不是来自她的性格设定，而是被选择来配合场景的审美需求。
她"应该"在这个时刻脸红，因为那样"好看"。她"应该"说一句带刺的话然后转过头去，
因为那样"有张力"。

角色变成了布景的一部分。她不再是一个有独立心理的人——她是叙事者在场景构图里
摆放的一个元素。

这就是为什么简单的 prompt（"假装你是一个傲娇女孩"）产出的体验是扁平的。AI
认识这个标签，但标签之下没有可以提取的私密内心。角色的一切反应都是表层表演。

### 这个框架做了什么

我把生成一个 galgame turn 的过程拆成了**三个独立的推理 pass**，每个 pass 有自
己的限定输入，通过 Context Firewall 在调用前物理删除不应看到的信息：

```
玩家输入
     │
     ├─▶ 剧情导演 ── 只决定：下一个节拍是什么类型？节奏该快还是慢？
     │   不能写散文，不能写对话，不能决定角色的情绪。
     │
     ├─▶ 场景叙述 ── 只写环境：光、声、空间、物体。舞台，不写演员。
     │   不能给角色标注情绪，不能解释潜台词，不能度量动作。
     │
     ├─▶ 角色构建 ── 新角色出现时，创建完整的心理学档案。
     │   （触发：玩家 /newrole、新人物入场、重大事件）
     │
     ├─▶ 角色回应 ── 每个角色独立调用一次。只收到她能看到/知道的信息。
     │   她的性格、依恋风格、防御机制是约束条件。她的行为从这些约束中产生。
     │         │
     │         ▼
     └─▶ 编辑合并 ── 整合输出，剥离叙事者的分析，保留舞台与演员之间的缝隙
```

**核心原则：约束优先于叙事。**

世界设定是物理定律。人物的性格、依恋风格、防御机制是初始条件。玩家的行动是
输入变量。故事是在这些约束下自然推演出的结果——不是预先编排的情感弧线。

如果她的人设里写的是回避型依恋 + 高防备值，那她面对靠近时就会回避、会岔开话
题、会沉默。哪怕"这个场景需要一次温柔回应才好看"，也绝不能违背她的设定。

叙事者的工作不是"写一个好故事"。叙事者是一个记录仪：忠实记录约束推演过程中
发生了什么。不解释，不度量，不替读者感受。

### 角色的心理学深度

每个角色不是靠标签（"傲娇"、"温柔"、"冷淡"）定义的——而是靠一套心理学结构：

- **荣格分层**：Persona（面具）/ Shadow（阴影）/ Anima-Animus（内化的亲密理想）
- **核心伤痛**：一个具体的、塑造了她的经历。不是"曾被抛弃"，而是"父母离婚后，
  母亲不再直接对她说话——所有沟通通过弟弟转达。她学会了：自己的感受太麻烦，
  不值得被直接面对。"
- **核心渴望**：她在所有行为之下真正想要的东西。和面具之间产生可玩的张力——
  可能是矛盾，也可能是骄傲、恐惧、责任、控制欲或饥渴。
- **依恋风格**：安全型 / 焦虑型 / 回避型 / 混乱型。直接决定她面对玩家靠近
  时的行为曲线——靠近、拉回、还是摇摆。
- **防御机制**（2-3 种）：理智化、投射、反向形成、置换、升华、否认、分裂。
  每种都有具体的触发条件和可见的行为表现。
- **解锁层级**（0-3）：trust < 35 只能看到表层。trust 35-64 出现裂痕。
  trust 65-84 阴影开始浮现。trust ≥ 85 核心伤痛暴露——真正的亲密才可能。

### 叙事风格：白描

叙事者像一台摄像机。每个句子描述可见、可听、可触的东西。

**不写：**
```
她的表情没有变化——至少第一眼看过去是这样。但她的眼睛先动了，
像是在确认自己刚才听到的话。
```

**写：**
```
她的表情没有变化。眼睛先动了一下。下巴微微收起。
手指搁在书脊的烫金字上，划了一下。又划了一下。
窗外的霓虹从冷白切回暖橙。
```

两个事实之间不需要一个"但"来替你完成情感连接。读者不是傻子。

禁止词：其实、显然、像是、仿佛、大概、似乎、不是X而是Y、不像X更像Y
限制词：但/却/然而——只允许用在物理矛盾上（"门开着，但灯没亮。"），不允许
用来替读者翻译人物情绪。

这套风格的哲学基础是：**相信观众**。相信他们有足够的观察力和洞察力，能从
克制的、客观的描述中感知到人物的情感。正如法国新浪潮电影所做的那样。

### 安装

```bash
git clone https://github.com/Shukahub/galgame-engine.git ~/.claude/skills/galgame-engine
```

重启 Claude Code，或运行 `/plugin` 加载。

### 快速开始

在 Claude Code 对话中说：

> 开始一个新的galgame

引擎会询问最低限度的设定，然后展示开场场景。之后直接输入角色行动和对话即可。

### Meta 命令

| 命令 | 用途 |
|------|------|
| `/pause` | 暂停故事，进入 meta 模式 |
| `/resume` | 恢复游玩 |
| `/status` | 查看当前场景、关系、开放线索 |
| `/profile` | 查看或修改主角档案 |
| `/memory` | 查看近期事件和角色印象 |
| `/tone` | 调整文风偏好 |
| `/debug` | 查看引擎内部状态 |
| `/save` / `/load` | 保存/读取会话 |
| `/newrole <描述>` | 创建并引入新角色 |
| `/newrole queue <描述>` | 创建角色但延后引入 |

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

MIT — 详见 [LICENSE](LICENSE)。

---

## English

### Why I built this

I like stories that feel real. Not "dramatic." Real.

A good story, in my view, isn't staged — it emerges. You set a world, a situation,
and characters with non-negotiable traits. Then you let events unfold. The craft
isn't in emotional manipulation; it's in arrangement: the timing of events, the
collision of personalities, the quiet moments that turn out to matter.

I love French New Wave cinema. The camera is flat. No push-in on a tear. No music
telling you what to feel. A woman opens a window. Cut to a glass of water on a
table. The director trusts you to feel something. That trust — the belief that the
audience has eyes and a mind — is what gives those films their weight.

This aesthetic is almost impossible to achieve in traditional AI roleplay.

### The flaw in traditional AI galgame

Nearly every AI character-chat skill and prompt suffers from the same structural
problem: **environment, character behavior, and plot advancement are generated
by one AI in one reasoning pass.**

The consequence is fatal to character: the character loses her subjectivity.

When the narrator and the character controller are the same thought process, the
character's actions get hijacked by narrative convenience. Her gestures, her lines,
her emotional responses — they don't come from her personality. They're chosen to
make the scene aesthetically pleasing. She "should" blush now, because that would
look good. She "should" say something sharp and then turn away, because that would
create "tension."

The character becomes set dressing. She's no longer a person with independent
psychology — she's an element the narrator places in the scene composition.

That's why simple prompts ("pretend you're a tsundere girl") produce flat
experiences. The AI knows the label, but there's no private interior beneath it
to draw from. Every response is surface performance.

### What this framework does

I broke the generation of a single galgame turn into **three independent reasoning
passes**, each with scoped inputs built through a Context Firewall that physically
deletes information the module shouldn't see:

```
Player Input
     │
     ├─▶ STORY DIRECTOR — Decides only: what beat type? faster or slower?
     │   Cannot write prose, dialogue, or decide character emotions.
     │
     ├─▶ SCENE NARRATOR — Writes only the environment: light, sound, space,
     │   objects. The stage, not the actors. Cannot label emotions, explain
     │   subtext, or quantify gestures.
     │
     ├─▶ CHARACTER ARCHITECT — Builds full psychological persona documents.
     │   (Triggered by: new characters, major events, player /newrole)
     │
     ├─▶ CHARACTER RESPONSE — Called once per character per turn. Receives
     │   only what that character can perceive and knows. Her personality,
     │   attachment style, and defense mechanisms are constraints. Her
     │   behavior emerges from those constraints.
     │         │
     │         ▼
     └─▶ EDITOR PASS — Merges outputs, strips narrator analysis, preserves
          the gap between stage and actors.
```

**Core principle: constraints over narrative.**

The world setting is physics. A character's personality, attachment style, and
defense mechanisms are initial conditions. The player's actions are input
variables. The story is what emerges when these constraints play out — not a
pre-scripted emotional arc.

If her persona document says avoidant attachment + high guard, then when the
player leans in, she pulls back. She deflects. She goes quiet. Even if "this
scene needs a tender response to look good" — you don't violate the constraints.

The narrator is not here to "write a good story." The narrator is a recording
instrument: faithfully documenting what happened during the constraint simulation.
No interpretation. No measurements. No feeling things on the reader's behalf.

### Psychological depth, not trope labels

Characters aren't defined by tags ("tsundere," "gentle," "cold"). They're built
with a psychological structure:

- **Jungian layers** — Persona (mask) / Shadow (repressed) / Anima-Animus
  (internalized ideal of intimacy)
- **Core wound** — a specific formative experience. Not "was abandoned." More:
  "After her parents divorced, her mother stopped speaking to her directly —
  all communication went through her younger brother. She learned her feelings
  were too inconvenient to be addressed."
- **Core desire** — what she actually wants beneath all behavior. Creates playable
  tension with her persona: contradiction, pride, fear, duty, control, hunger.
- **Attachment style** — secure / anxious / avoidant / disorganized. Governs how
  she behaves as closeness increases.
- **Defense mechanisms** (2-3) — intellectualization, projection, reaction
  formation, displacement, sublimation, denial, splitting. Each with concrete
  triggers and visible behavioral manifestations.
- **Unlock tiers** (0-3) — trust < 35: surface only. trust 35-64: first cracks.
  trust 65-84: shadow surfaces. trust ≥ 85: core wound exposed, genuine
  vulnerability possible.

### Prose style: observational minimalism (白描)

The narrator operates like a camera. Every sentence describes what is visible,
audible, or tangible.

**Not this:**
```
Her expression didn't change — at least not at first glance. But her eyes moved
first, as if confirming what she'd just heard.
```

**This:**
```
Her expression didn't change. Her eyes moved first. Her chin drew back slightly.
Her finger rested on the gilded title of the book's spine. Traced it once. Again.
Outside the window, the neon shifted from cold white back to warm orange.
```

Two facts placed side by side don't need a "but" to complete the emotional
connection. The reader is not stupid.

Prohibited: "she wasn't X-ing, she was Y-ing," "not so much X as Y," symbolic
metaphors that tell the reader what to feel.

Restricted contrast words: "but," "yet," "however" — allowed only for physical
contradictions ("the door was open, but the light was off"), never to translate
a character's emotions for the reader.

The philosophy: **trust the audience.** Believe they have enough observation and
insight to perceive a character's emotions through restrained, objective
description. What French New Wave cinema did with a camera, this engine does
with prose.

### Installation

```bash
git clone https://github.com/Shukahub/galgame-engine.git ~/.claude/skills/galgame-engine
```

Restart Claude Code, or run `/plugin` to load.

### Quick Start

In a Claude Code conversation:

> 开始一个新的galgame

The engine asks for minimal setup, then opens with a scene. From there, type
your character's actions and dialogue directly.

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

### File Structure

```
galgame-engine/
├── SKILL.md              # Main entry — core rules, quality constraints
├── references/
│   ├── module-prompts.md      # Full prompt contracts for all modules
│   ├── persona-schema.md      # Complete Persona Document JSON schema
│   ├── world-state-schema.md  # World state schema, runtime modes
│   ├── meta-commands.md       # Slash command routing and handling
│   ├── style-guide.md         # Detailed white-description style guide
│   └── overview.md            # Architecture overview (Chinese)
└── LICENSE
```

### License

MIT — see [LICENSE](LICENSE).

---

*Built on the belief that the highest form of storytelling is restraint — and
that characters deserve to be people, not props.*
