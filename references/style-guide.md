# Style Guide: Observational Minimalism

Use this when rendering player-facing prose. The target style is plain observation:
facts arranged like camera shots. Emotion should be inferred from what is present, not
explained by the narrator.

## Goal

Write scenes as if the camera records:
- body movement;
- object position;
- light, sound, temperature, smell, texture;
- distance and orientation;
- pauses, repeated actions, and changes in rhythm;
- spoken lines.

Do not tell the player what those facts mean. Let the sequence carry feeling.

## Core Rules

1. Describe phenomena, not interpretation.
2. Prefer short declarative sentences.
3. Put actions in physical order.
4. Keep the narrator out of the scene.
5. Remove emotional translation after an action.
6. Remove symbolic explanation unless the player can literally perceive it.
7. Dialogue may be ambiguous; do not add a narrator key that solves it.
8. End on a concrete observable state, not a thematic sentence.

## Avoid

Avoid narrator guidance:
- "其实"
- "显然"
- "像是"
- "仿佛"
- "大概"
- "似乎"
- "不是 X，而是 Y"
- "不像 X，更像 Y"
- "她没有真的..."
- "她把门框还给你了"
- "这不是故事，是规格参数"
- "那句话还浮在你们之间"

Avoid contrast words when they only explain subtext:
- "但"
- "却"
- "然而"
- "只是"
- "偏偏"

These words are allowed only when they describe a concrete visible contradiction:
"门开着，但灯没亮。" Do not use them to translate emotion:
"她说要看书，但手指没有翻页。"

Avoid interpretive adjectives unless directly visible:
- "动摇的"
- "不确定的"
- "认真地"
- "温柔地"
- "防备地"
- "狼狈地"
- "暧昧地"

Replace them with observable facts:
- "手指停在页边。"
- "她看了你一秒。"
- "杯子放回桌面时碰出一声轻响。"
- "她把视线移到门上。"

## Preferred Sentence Shape

Use:
```text
她的表情没有变化。眼睛先动了一下。下巴微微收起。
手指搁在书脊的烫金字上，划了一下。又划了一下。
窗外的霓虹从冷白切回暖橙。
```

Avoid:
```text
她的表情没有变化——至少第一眼看过去是这样。但她的眼睛先动了，
像是在确认自己刚才听到的话。
```

Use:
```text
她看着书页说。手指停在页码上。没有翻页。
```

Avoid:
```text
她这句话是看着书页说的，不是对着你说的。但她的手指还停在页码上。
```

Use:
```text
她把书从左手换到右手。封面朝下，手指夹在某一页。
然后换回左手，放回柜台上，封面朝上。
```

Avoid:
```text
她先把书从左手换到了右手，像是给自己找了一个物理缓冲。
```

## Decision Point Style

Decision points should not explain the emotional meaning of the moment.

Use:
```text
▎ 她看着你。书页停在拇指下。
```

Use:
```text
▎ 门口的铜铃还在轻轻晃。她没有继续翻页。
```

Avoid:
```text
▎ 她没关门。但她把门框还给你了——你来推。
```

Avoid:
```text
▎ 那个词还浮在你们之间。你要不要继续靠近她的内心？
```

## Editing Pass

Before final output, scan and cut:
- narrator explanations after actions;
- "but/however/yet" clauses that only mark subtext;
- metaphors that tell the reader what to feel;
- sentences that label a line as "not a question", "not a story", "not a joke";
- implied攻略 prompts such as "你要不要推开她的门".

When cutting, do not remove the underlying physical fact. Keep the action, object,
light, sound, and line.
