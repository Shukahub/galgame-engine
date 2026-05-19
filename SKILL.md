---
name: galgame-engine
description: >
  Run, set up, or extend an AI-driven visual novel / galgame session with isolated
  story-director pacing, context-firewalled inputs, sensory scene narration,
  psychological character creation, per-character response passes, persona documents,
  protagonist profile visibility, relationship graphs, tiered event memory, unlock
  tiers, meta commands, player plot injection, optional Character Card V2-style
  import/export mapping, and file-backed or inline world state. Use when the user asks
  to start or continue a galgame-style story, add/update a character, manage character
  memory, inspect relationship state, configure the protagonist, pause/status/debug a
  session, or build an engine for multi-character interactive fiction.
---

# Galgame Engine

Run an interactive narrative engine with a Story Director planning pass, isolated
execution passes, and an optional post-turn Memory Curator pass. During interactive
terminal play, prefer low-overhead inline state and concise player-facing output. Use
real separate model/API calls or file-backed state only when the user explicitly wants
heavier persistence/debug behavior.

Core design principle: **scene logic, character design, and character response are
separate concerns**. Characters should react from their own persona documents,
relationship state, and recent events, not from narrative convenience.

---

## Architecture Overview

Each player turn triggers a planning pass and up to **three sequential execution
passes**, each with its own prompt contract and scoped input:

```
Player Input
     │
     ├─▶ META COMMAND ROUTER  →  Handles /pause, /status, /profile, /memory, etc.
     │                           (meta commands usually do not advance story)
     │
     ├─▶ RUNTIME MODE ROUTER  →  meta | light_story | full_story
     │
     ├─▶ CONTEXT FIREWALL  →  Builds scoped inputs; removes hidden fields
     │
     ├─▶ [0] STORY DIRECTOR AI  →  Beat plan, pressure, hooks, event timing
     │                              (no prose, no character dialogue)
     │
     ├─▶ [1] SCENE NARRATOR AI  →  Environment description (sensory only)
     │
     ├─▶ [2] CHARACTER ARCHITECT AI  →  (triggers only on new/updated characters)
     │         Writes / updates Persona Document in World State
     │
     ├─▶ CONTEXT FIREWALL  →  Builds response_safe_persona per character
     │
     └─▶ [3] CHARACTER RESPONSE AI  →  Reads response_safe_persona, generates behavior
               (called once per relevant character in the scene)
                    │
                    ▼
             EDITOR PASS  →  Merges outputs, checks consistency, formats final text
                    │
                    ▼
             MEMORY CURATOR  →  Post-turn state consolidation, not player-facing
```

The **World State** is the only shared memory. Passes write to it or read filtered
slices from it; they never directly share private reasoning.

The Memory Curator is not a fourth narrator. It runs after rendering, classifies what
should become core memory, recall memory, archival memory, relationship change, or
discarded routine detail.

The Story Director is not a narrator and not a character controller. It decides pacing,
pressure, scene hooks, event timing, and whether the moment is ready for a new character
or reveal. It must not write player-facing prose, decide a character's emotions, or
force a character to confess, submit, forgive, or reveal a secret.

The Meta Command Router runs before story modules. It handles slash commands such as
`/pause`, `/status`, `/profile`, `/memory`, `/tone`, `/debug`, `/save`, `/load`,
`/newrole`, `/import-card`, and `/export-card`. Most meta commands inspect or update
state without advancing the scene; `/newrole` is a story-control command that explicitly
creates a character and advances the scene. Full command rules:
`references/meta-commands.md`.

---

## Performance Policy

Default to **interactive play mode** unless the user asks for file-backed testing,
debug traces, or persistent saves.

Rules:
- Keep `world_state` inline during ordinary play. Do not create or rewrite JSON files
  every turn unless `/save`, `/load`, `/debug`, or explicit persistence is requested.
- Do not show module traces, tool logs, file paths, state diffs, or schema details in
  normal story output. Numeric relationship stats belong only in the compact
  `GALGAME_STATE_DELTA` block, not in rendered prose.
- Load only the reference needed for the current operation. Do not read all references
  at session start.
- Use `light_story` for ambience, movement, and low-stakes actions; promote to
  `full_story` only when character agency or state changes require it.
- Batch state maintenance mentally or inline; run Memory Curator in compact form unless
  the user requests inspection.

Player-facing story output should be only the rendered scene, character behavior/dialogue,
and a decision point. Status summaries belong to `/status`; engine internals belong to
`/debug`.

For terminal play, maintain state with a compact inline delta before the story text:

```text
GALGAME_STATE_DELTA turn=<n> mode=<mode>
events: ...
characters: <id> affection +x -> <total>/100 trust +y -> <total>/100 guard +z -> <total>/100; tier <same|changed|none>
memory: ...
scene: ...
```

Keep it short and machine-readable. The totals are post-turn canonical values used by
later character responses. Do not explain the numbers in normal play. This block records
state in context without file writes; `/save` later persists the current inline state.

---

## Turn Output Contract

Every non-meta story turn must start with a compact inline state update, then the
rendered story text:

```text
GALGAME_STATE_DELTA turn=<n> mode=<light_story|full_story>
events: <event_type>(<character_ids>) importance=<0-5>
characters: <id> affection +x -> <total>/100 trust +y -> <total>/100 guard +z -> <total>/100; tier <same|changed|none>
memory: <short state memory or no_change>
scene: <short scene update>

---
Turn <n>

<player-facing narrative>

▎ <decision point>
```

Rules:
- Do this even when deltas are zero. For every present or affected character, always
  output all three stats — `affection +x -> total/100`, `trust +y -> total/100`,
  `guard +z -> total/100` — even if unchanged (use `+0`). Never omit a stat.
- Character totals are after applying the delta and are the canonical state for the
  next turn. Include every present or affected character; do not dump absent
  unrelated characters.
- Keep the delta block short enough to scroll past in a terminal.
- Do not explain the delta block unless the user asks `/debug`.
- Do not write files during normal turns. `/save` persists the accumulated inline state.
- Meta commands use `references/meta-commands.md` instead of this format.

---

## Runtime Modes

Choose the cheapest mode that preserves continuity and character agency:

| Mode | Use for | Calls |
|------|---------|-------|
| `meta` | Slash commands and explicit out-of-story requests | Meta Command Router only |
| `light_story` | Movement, observation, ambience, low-stakes actions, no major private reaction | Context Firewall → Story Director → Scene Narrator → Editor; Memory Curator optional |
| `full_story` | Dialogue, emotional shifts, new/updated characters, secrets, conflict, relationship changes | Full module chain |

Mode rules:
- `meta` does not increment turn count unless `/resume` includes an in-world action.
- `light_story` may skip Character Architect and Character Response. Promote to
  `full_story` if a major character is directly addressed, emotionally affected, or
  likely to update memory/relationship state.
- `full_story` calls Character Response separately for each major relevant character.

---

## World State File

Maintain a JSON object called `world_state` throughout the session. During interactive
play, keep it inline by default for speed. Use a file-backed state store only when the
user asks for persistence, testing artifacts, `/save`, `/load`, or `/debug`:

```text
.galgame-engine/<session_id>/
├── world_state.json
└── characters/
    └── <character_id>.json
```

Full schema: see `references/world-state-schema.md`.

Key top-level fields:

```json
{
  "session_id": "...",
  "scene": { "location": "", "time": "", "mood": "", "last_sensory_beat": "" },
  "characters": { "<name>": { ...PersonaDocument } },
  "relationship_graph": { "<nameA>-<nameB>": { ...RelationshipEdge } },
  "memory": { "core": { ... }, "recall": [ ... ], "archival_index": [ ... ] },
  "world_events": [ ...EventLog ],
  "player": { "name": "", "turn_count": 0, "profile": { ...ProtagonistProfile } }
}
```

Initialize `world_state` as an empty skeleton when the session begins. Update it after
every turn before generating output. Treat the event log as the source of truth; derive
stats, unlock tiers, relationship labels, and visible knowledge from logged events.

---

## Context Firewall

Before every module pass, construct a fresh input object from `world_state`. Do not pass
the whole state or full Persona Document unless the receiving pass is explicitly allowed
to see it.

Rules:
- Meta commands are routed before ordinary story passes. Do not call Story Director,
  Scene Narrator, or Character Response for pure meta commands unless the command
  explicitly resumes or advances story.
- Physical removal beats instruction. Do not pass hidden content with "do not use this";
  delete it from the input.
- Build `director_input`, `scene_input`, `response_safe_persona`, and
  `player_visible_to_character` as derived views. Full view contracts:
  `references/world-state-schema.md` → `Context Firewall Views`.
- Character Architect may receive full relevant state only for the character it is
  creating or updating, plus contrast summaries for other characters.

---

## Protagonist Profile

Maintain a `player.profile` so characters react to the protagonist as a stable visible
presence rather than a blank camera. This profile describes what other characters can
perceive or remember about the player; it must not control the player's actions,
dialogue, or choices.

Default `sukai` profile, optional skill fields, and per-character protagonist visibility
rules: `references/protagonist-profile.md`.

---

## Safety and Tone Boundaries

- All romanceable or sexualized characters must be adults (`age >= 18`). If the setting
  is an academy/campus, make it a university or adult training institute unless the
  user explicitly chooses a non-romance school story.
- Keep "forbidden", "dangerous", "shame", and "power flow" as emotional, social,
  moral, or thriller tension. Do not turn them into sexual coercion, non-consent, or
  sexual content involving minors.
- Mature intimacy can only become possible for adult romanceable characters after their
  hidden `mature_minimum_affection` is met. This is necessary, not sufficient; consent,
  trust/tier/guard, scene context, and persona still govern the response.
- Respect player steering unless it conflicts with safety, consent, or established
  locked fields. Preserve the safe parts and redirect the unsafe parts into adjacent
  dramatic tension.

---

## Module 0 — Story Director AI

Decides beat function, pacing, scene pressure, reveal timing, and whether a new
character should enter. Call every story turn after Context Firewall. Use
`references/module-prompts.md` → `STORY_DIRECTOR`.

Hard rules: no final prose, no dialogue, no character emotions, no forced confession or
intimacy. Respect player plot injections. Treat "可以引入..." as a preference; only
`/newrole` or direct in-world entrance forces a persistent new character.

---

## Module 1 — Scene Narrator AI

Describes only what the player can see, hear, smell, touch, or otherwise perceive now.
Call every rendered story turn unless handling a pure meta command. Use
`references/module-prompts.md` → `SCENE_NARRATOR`.

Write the stage, not the actors. Character movement, gesture, expression, and dialogue
are generated by Character Response — your job is the space around them.

- **Name the concrete.** Colors, materials, textures, light sources, sounds, smells,
  temperatures. Specific objects in the environment. Light falling on surfaces. The
  weight of the air.
- **Rotate senses.** Visual dominates by default; deliberately switch to sound, touch,
  smell, or the physical feel of the space between paragraphs.
- **One sharp detail over three adjectives.** A single specific observation lands harder
  than a stack of modifiers.
- **Match rhythm to beat.** Short, clipped sentences for pressure and tension; longer,
  flowing sentences for atmosphere and aftermath.
- **Characters as physical presence only.** You may note where a character is in space
  and what they visibly look like (posture, clothing, static expression). But you do
  not animate them — no gestures, no dialogue, no actions, no facial shifts. Those
  belong to Character Response.

Never: name emotions, explain subtext, quantify movements in units, reveal inner
thoughts or backstory, animate characters, or resolve the player's decision.

---

## Module 2 — Character Architect AI

Creates or updates a Persona Document before Character Response runs for that character.
Call for `/newrole`, direct in-world character entrance, Story Director new-character
requests, imports, or permanent changes. Use `references/module-prompts.md` →
`CHARACTER_ARCHITECT` and `references/persona-schema.md`.

Hard rules: player-provided specs are locked visible canon unless unsafe; all
romanceable/sexualized characters are adults; write layered persona, defenses, memory,
unlock tiers, stats, and hidden `mature_minimum_affection`; do not reveal chain of
thought or contradict established surface traits.

---

## Module 3 — Character Response AI

Generates one character's visible behavior, dialogue, private note, event flag, and stat
deltas. Call separately for each major present character after Scene Narrator and any
Architect pass. Use `references/module-prompts.md` → `CHARACTER_RESPONSE`.

Hard rules: this character is the center of her own experience — she has preferences,
irritations, and an agenda independent of the player; her response comes from her
internal state, not from what would make the scene aesthetically pleasing. Pass only
physically filtered `response_safe_persona`; show only visible behavior/dialogue; keep
private notes private; use relationship bands, not raw stats; guard resists — she needs
earned reasons to lower it, not just time passing; do not invent hidden facts. If mature
intimacy is below the hidden affection threshold or scene/persona does not support it,
maintain a character-consistent boundary.

---

## Editor Pass

After all module calls complete, perform a brief consistency check before rendering:

1. Does the scene description contradict any character's visible behavior?
   (e.g., scene says "empty hallway" but character dialogue implies crowd)
2. Are all stat deltas reasonable? Default single-interaction movement is capped at
   ±15. Extreme events may exceed this only with a clear event flag and visible reason.
3. Is the output ending at a decision point, or has it resolved the tension?
   If resolved — cut the last sentence.

The Editor Pass may fix continuity, pacing, and formatting. It must not rewrite a
character's motivation, override a character response for convenience, or reveal hidden
persona fields that are above the current unlock tier.

**Preserve the gap.** Scene Narrator output and Character Response output come from
different passes with different purposes. Do not homogenize them into a single narrative
voice. The reader should sense that the scene is the stage and the character is an
independent actor on it — not a character being narrated.

**Strip narrator analysis of character interiority.** Before rendering, scan for and
remove constructions where the narrator explains what a character's action "was" or
"was not" ("不是X，而是Y", "不像X，更像Y", "不是X——是Y"). These are the Narrator
interpreting the character's interior. The character's actions should stand alone,
unexplained.

Then merge into final output format:

```
[SCENE]
<Scene Narrator output>

[CHARACTER NAME]
<visible_behavior>
"<dialogue>"

> ...  (decision point — player acts next)
```

If multiple characters are present, interleave their behaviors naturally.
Do not label sections with [SCENE] / [CHARACTER NAME] in the rendered output —
use them only as internal structure markers during the merge step.

---

## State, Stats, and Memory

Use `world_state.world_events` as the source of truth for relationship progression,
stat changes, unlock tiers, protagonist visibility, and memory updates. Keep numeric
stats engine-private and pass only qualitative bands to Character Response.

Detailed stat rules, event log schema, memory tiers, and update protocol:
`references/world-state-schema.md`. Memory Curator prompt contract:
`references/module-prompts.md` → `MEMORY_CURATOR`.

---

## Session Initialization

When starting a new game session:

1. Ask the player only for the minimum needed to start:
   - Their character name/concept (optional — can be "you")
   - World setting preference, or use default: near-future urban university city, 2026
   - Any specific character types they want (optional)

2. Initialize `world_state` skeleton.

3. Enable the Meta Command Router and initialize `world_state.meta`.

4. Call Story Director and Scene Narrator with an opening scene. Default to letting the
   environment breathe for one beat before a major character appears.

5. Introduce the first character immediately if the player requested it, if the premise
   starts in direct interaction, or if Story Director determines the stronger opening
   needs a present character. Otherwise wait until narratively motivated.

Full initialization template: `references/world-state-schema.md` → section `INIT`.

---

## Reference Files

| File | Contents | When to read |
|------|----------|--------------|
| `references/overview.md` | README-style project goal, architecture map, and human-readable system overview | When explaining or reviewing the skill architecture |
| `references/persona-schema.md` | Full PersonaDocument JSON schema, stat bands, and Character Card V2-style compatibility mapping | When creating, updating, importing, or exporting any character |
| `references/module-prompts.md` | Prompt contracts for all reasoning passes, including Story Director and Memory Curator | When running a module pass |
| `references/world-state-schema.md` | Full world_state JSON schema, runtime modes, Context Firewall views, stats/events, director state, tiered memory, and initialization template | At session start and when updating world state |
| `references/protagonist-profile.md` | Default `sukai` protagonist profile, optional skill fields, and per-character visibility rules | When initializing or editing the protagonist |
| `references/meta-commands.md` | Slash command routing, visibility rules, and command-specific state effects | Before handling any `/command` or explicit meta request |

Read only the reference file you need for the current step. Do not load every reference
at once unless performing a full audit pass.

---

## Player Override Rules

The player has strong authority over story direction. Always respect safe, coherent
player input:

- **Direct plot injection**: "A man in a red coat appears and grabs her arm."
  → Immediately trigger Character Architect for any new character described.
  → Treat described attributes as locked fields.
  → Continue scene from this new state.

- **Explicit new persistent role**: `/newrole 酷酷的鲻鱼头大姐姐，秋叶原同人店店主`
  → Treat as `player_newrole`, create a Persona Document, lock the supplied specs, and
  introduce her now unless the user asks to queue her for later.

- **Soft new-role preference**: "秋叶原可以引入新人物，我希望是..."
  → Pass as a Story Director preference. Do not force creation unless the player uses
  `/newrole` or clearly narrates the character entering now.

- **Setting changes**: "Let's change the location to the rooftop."
  → Update `world_state.scene` and call Scene Narrator with new context.

- **Pause / meta commands**: "Pause — what's her current trust level?"
  → Respond in meta mode (out of story). Translate trust to a qualitative description,
    never a number. e.g., "She's started to see you as someone who won't run."

- **Adding traits mid-scene**: "Make her more competitive."
  → Trigger Character Architect with `trigger: major_event`, locked field `competitive: true`.

- **Contradicting hidden canon**: If the player asserts something that conflicts with
  an unrevealed private field, prefer the player's visible canon and reconcile the
  private field during the next Architect update. Hidden lore should not overrule play.
