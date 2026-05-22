# Galgame Engine Overview

这是一份 README 风格的总览文档，用来帮助人快速理解 `galgame-engine`
这个 skill 的目标、架构、模块边界和运行方式。执行时仍以 `SKILL.md` 和各 reference
文件为准。

## Contents

- [Project Goal](#project-goal)
- [Core Design Idea](#core-design-idea)
- [Architecture Map](#architecture-map)
- [Runtime Modes](#runtime-modes)
- [Module Responsibilities](#module-responsibilities)
- [State And Memory](#state-and-memory)
- [Information Isolation](#information-isolation)
- [Protagonist Model](#protagonist-model)
- [Meta Commands](#meta-commands)
- [File Guide](#file-guide)
- [Quality Principles](#quality-principles)

---

## Project Goal

`galgame-engine` 的目标是把普通 AI 聊天改造成一个更像 galgame / visual novel
引擎的互动叙事系统。

它追求的不是一次性写出一段剧情，而是长期运行时保持这些能力：

- 剧情有节奏：知道什么时候安静、施压、打断、给线索、制造冲突或引入新角色。
- 角色有主体性：角色不是为了剧情方便而立刻配合玩家，而是根据自己的档案、关系、记忆和防御机制做反应。
- 信息有边界：玩家、角色、引擎各自知道的信息不同，秘密不能因为同一个模型读过就提前泄漏。
- 互动可探索：玩家通过行动、对话、试探、保护边界、共同事件逐渐解锁角色更深层。
- 主角可被感知：主角不是空白镜头，而是有外貌、声音、气质和可见特质；但角色只能知道自己观察到的部分。
- 长线可维护：世界状态、角色档案、关系图、事件记忆和 meta 命令都能被持续管理。

默认基调是近未来成人校园都市：浪漫、危险、反差、心理张力、开放探索。

---

## Core Design Idea

核心思想是把“讲故事”和“角色反应”拆开。

普通 AI 写 galgame 容易出现一个问题：同一个模型同时负责环境、剧情推进、角色行为，
于是角色会被叙事目标绑架。她本来应该抗拒、误解、沉默或转移话题，却为了剧情推进突然配合、告白或揭露秘密。

这个 skill 用模块边界解决这个问题：

- Story Director 只负责节奏和机会，不写角色台词。
- Scene Narrator 只负责可感知场景，不写角色内心。
- Character Architect 只负责创建和更新角色深层档案。
- Character Response 只负责一个角色当下的行为和台词。
- Editor Pass 只负责合并和剪辑，不改角色动机。
- Memory Curator 只负责事后整理状态，不补写剧情。

---

## Architecture Map

```text
Player Input
     |
     v
META COMMAND ROUTER
     |-- meta command -> inspect/update state, usually stop here
     |
     v
RUNTIME MODE ROUTER
     |-- meta
     |-- light_story
     |-- full_story
     |
     v
CONTEXT FIREWALL
     |-- builds director_input
     |-- builds scene_input
     |-- builds response_safe_persona
     |-- builds player_visible_to_character
     |
     v
STORY DIRECTOR
     |-- beat type
     |-- spotlight
     |-- pressure / clue / interruption
     |-- new character request
     |
     v
SCENE NARRATOR
     |-- sensory scene only
     |-- visible / audible / smell / touch
     |
     v
CHARACTER ARCHITECT
     |-- only on new or updated characters
     |-- writes Persona Document
     |
     v
CHARACTER RESPONSE
     |-- one major character per call
     |-- uses filtered persona only
     |
     v
EDITOR PASS
     |-- merge outputs
     |-- remove contradictions
     |-- end at decision point
     |
     v
MEMORY CURATOR
     |-- update events
     |-- classify memory
     |-- update relationships
```

---

## Runtime Modes

运行模式用于节省 token 和调用成本。

| Mode | Use Case | Calls |
|------|----------|-------|
| `meta` | `/status`, `/profile`, `/memory`, `/debug`, out-of-story requests | Meta Command Router only |
| `light_story` | 移动、观察、氛围、低风险行动 | Context Firewall, Story Director, Scene Narrator, Editor; Memory Curator optional |
| `full_story` | 对话、情绪变化、冲突、秘密、新角色、关系变化 | Full module chain |

`light_story` 不是低质量模式，而是低成本模式。只要主要角色被直接影响，就提升到
`full_story`。

---

## Module Responsibilities

### Story Director

负责“下一拍的功能”：

- 现在该安静还是升压？
- 谁应该被聚光？
- 是否适合给线索？
- 是否需要外部事件打断？
- 是否到了引入新角色的时机？

它不能写最终 prose、不能替角色决定感情、不能强迫角色揭露秘密。

### Scene Narrator

负责“玩家当前能感受到什么”：

- 光、声、气味、触感、空间、距离、天气、人群、物体。
- 只写可观察表现，不写内心、动机、背景解释。

### Character Architect

负责角色档案：

- 表层外貌、声音、动作习惯。
- 荣格层：Persona、Shadow、Anima/Animus。
- 核心伤口、核心欲望、依恋模式、防御机制。
- 解锁层级、记忆、关系、边界。

### Character Response

负责单个角色的当下行为：

- 她看见了什么？
- 她目前怎样理解主角？
- 她和主角/其他人的关系是什么？
- 以她当前解锁层级，她会说什么、避开什么、误读什么？

它不能读取完整角色档案，只能读取 `response_safe_persona`。

### Editor Pass

负责把多个模块的输出剪辑成玩家读到的一回合文本。

它可以修连续性和节奏，但不能为了好看而改掉角色动机。

### Memory Curator

负责回合结束后的状态整理：

- 追加事件。
- 更新关系。
- 记录私有角色记忆。
- 把记忆分成 core / recall / archival。

---

## State And Memory

核心状态叫 `world_state`。交互游玩时默认内联保存，速度更快；只有在 `/save`、
`/load`、`/debug`、长线存档或测试复现时才建议文件化：

```text
.galgame-engine/<session_id>/
├── world_state.json
└── characters/
    └── <character_id>.json
```

主要内容：

- `player`: 主角档案、当前 turn、各角色对主角的认知。
- `scene`: 当前地点、时间、氛围、在场角色。
- `characters`: 角色 Persona Documents。
- `relationship_graph`: 角色之间的关系、张力、信任、压力。
- `director_state`: 节奏、聚光历史、事件冷却、开放剧情线。
- `memory`: core / recall / archival。
- `world_events`: 事件日志。
- `meta`: 是否暂停、debug、tone 偏好、最近命令。

事件日志是关系推进的事实来源。数值可以存在，但玩家默认只看到行为化描述。

---

## Information Isolation

信息隔离是这个 skill 的关键。

不要把完整世界状态直接塞给所有模块。每个模块都应该收到自己的派生视图：

- `director_input`: 给 Story Director，只含可见剧情状态和非剧透压力摘要。
- `scene_input`: 给 Scene Narrator，只含当前可感知场景。
- `response_safe_persona`: 给 Character Response，只含当前 unlock tier 及以下的角色信息。
- `player_visible_to_character`: 给 Character Response，只含该角色能感受到的主角信息。

原则：物理删除优先于提示忽略。不要传隐藏内容再写“不要使用”；直接不传。

---

## Protagonist Model

主角通过 `player.profile` 管理。

默认主角是 `sukai`：

- 23 岁男性。
- 185cm / 70kg，高挑清瘦。
- 冷白皮，黑色长发，后脑勺扎一个炸开的马尾。
- 死鱼眼，淡漠，有距离感。
- 中性感、少年感、自由随性的阿宅气质。
- 说话随意，带慵懒和调侃。

但角色不能读取完整主角档案。每个角色只读取
`player_visible_to_character`，也就是她当前能看到、听到、确认、误读或怀疑的主角。

详见 `references/protagonist-profile.md`。

---

## Meta Commands

Meta commands 用于管理游戏；多数不推进剧情。`/newrole` 是例外，它是显式造角并推进
剧情的 story-control 命令。

常用命令：

- `/pause`: 暂停剧情。
- `/resume`: 恢复剧情。
- `/status`: 查看当前状态的玩家安全摘要。
- `/profile`: 查看或修改主角档案。
- `/memory`: 查看近期事件、开放剧情线、角色已知印象。
- `/tone`: 调整基调。
- `/debug`: 查看引擎私有状态。
- `/save` / `/load`: 保存和读取。
- `/newrole`: 显式创建并引入或排队一个新角色。
- `/import-card` / `/export-card`: 角色卡兼容层。

默认 `/status` 和 `/memory` 不泄露隐藏字段。只有 `/debug` 才进入引擎私有视角。

---

## File Guide

```text
galgame-engine/
├── SKILL.md
└── references/
    ├── overview.md
    ├── module-prompts.md
    ├── world-state-schema.md
    ├── persona-schema.md
    ├── protagonist-profile.md
    ├── meta-commands.md
    └── style-guide.md
```

文件用途：

- `SKILL.md`: 轻量入口、模块边界、调用顺序、引用导航。
- `overview.md`: 人类可读的 README 风格架构说明。
- `module-prompts.md`: 各模块输入输出契约。
- `world-state-schema.md`: 世界状态、运行模式、视图过滤、记忆和更新协议。
- `persona-schema.md`: 角色档案 schema 和角色构建规范。
- `protagonist-profile.md`: 主角默认档案和主角可见性规则。
- `meta-commands.md`: 斜杠命令和 meta 模式规则。
- `style-guide.md`: 白描文风、事实镜头、去解释化的输出规范。

---

## Quality Principles

1. 角色不能轻易被攻略。好感、信任和防备必须有事件支撑。
2. 玩家可以引导剧情，但隐藏设定不能强压玩家已经建立的可见事实。
3. 角色只暴露行为，不提前解释深层心理。
4. 剧情服务感觉，但不牺牲角色主体性。
5. 秘密需要线索、时机和关系状态，不靠突然揭秘。
6. 长线记忆要压缩，核心记忆要小，归档记忆要可检索。
7. 用最便宜的 runtime mode 达成当前回合目标。
8. 终端游玩默认不要每回合写文件；每回合用短 `GALGAME_STATE_DELTA` 记录 delta 和 post-turn total，需要持久化时用 `/save`。
9. 成熟亲密路线由角色专属隐藏好感门槛控制；达标只是必要条件，不是自动同意。
