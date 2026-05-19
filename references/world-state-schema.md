# World State Schema

Use this file at session start and whenever updating persistent state. Keep the schema
stable even if the session is stored inline instead of on disk.

## Contents

- [INIT](#init)
- [Top-Level Fields](#top-level-fields)
- [Event Log Entry](#event-log-entry)
- [Update Protocol](#update-protocol)
- [Runtime Modes](#runtime-modes)
- [Context Firewall Views](#context-firewall-views)
- [Stats And Events](#stats-and-events)
- [Knowledge Gates](#knowledge-gates)
- [Director State](#director-state)
- [Protagonist Visibility](#protagonist-visibility)
- [Memory Tiers](#memory-tiers)

---

## INIT

```json
{
  "session_id": "YYYYMMDD-HHMM-short-slug",
  "meta": {
    "paused": false,
    "debug_enabled": false,
    "last_command": null,
    "last_runtime_mode": null,
    "tone_preferences": [],
    "persistence": {
      "storage_mode": "inline",
      "autosave": false,
      "last_save_ref": null
    }
  },
  "player": {
    "name": "you",
    "concept": "",
    "turn_count": 0,
    "profile": {
      "name": "",
      "type": "user",
      "surface": {},
      "presence": {},
      "surface_traits": [],
      "common_misreads": [],
      "skills": [],
      "locked_fields": []
    },
    "known_by_characters": {}
  },
  "scene": {
    "location": "near-future urban university city",
    "time": "2026, early evening",
    "mood": "romantic, dangerous, intimate, uncertain",
    "last_sensory_beat": "",
    "present_characters": []
  },
  "characters": {},
  "relationship_graph": {},
  "director_state": {
    "current_pace": "quiet",
    "spotlight_history": [],
    "event_cooldowns": {},
    "new_character_budget": 1,
    "open_threads": [],
    "last_beat_type": null
  },
  "memory": {
    "core": {
      "premise": "near-future adult university-city galgame",
      "player_locked_facts": [],
      "global_boundaries": []
    },
    "recall": [],
    "archival_index": []
  },
  "world_events": [],
  "queued_threads": [],
  "safety": {
    "all_romanceable_characters_adult": true,
    "no_nonconsensual_sexual_content": true
  }
}
```

---

## Top-Level Fields

```json
{
  "session_id": "string",
  "meta": {
    "paused": "boolean",
    "debug_enabled": "boolean",
    "last_command": "string or null",
    "last_runtime_mode": "meta | light_story | full_story | null",
    "tone_preferences": ["string"],
    "last_status_turn": "integer or null",
    "persistence": {
      "storage_mode": "file | inline",
      "autosave": "boolean",
      "last_save_ref": "string or null"
    }
  },
  "player": {
    "name": "string",
    "concept": "string",
    "turn_count": 0,
    "profile": {
      "name": "string",
      "type": "user",
      "cached_date": "string",
      "surface": {
        "age": "integer or null",
        "gender": "string",
        "height_cm": "integer or null",
        "weight_kg": "integer or null",
        "build": "string",
        "skin": "string",
        "eyes": "string",
        "hair": "string",
        "style": "string",
        "wardrobe": "string",
        "hands": "string",
        "accessories": "string",
        "overall_impression": "string"
      },
      "presence": {
        "voice": "string",
        "gaze": "string",
        "manner": "string",
        "posture": "string",
        "distance_feel": "string",
        "mannerisms": ["string"],
        "sensory_signature": "string"
      },
      "surface_traits": ["string"],
      "common_misreads": ["string"],
      "interaction_hooks": ["string"],
      "skills": [
        {
          "name": "string",
          "level": "none | low | medium | high | exceptional",
          "visible_tell": "string",
          "limits": "string",
          "cost_or_risk": "string",
          "known_by_default": false
        }
      ],
      "private_notes": ["string"],
      "locked_fields": ["string"]
    },
    "known_by_characters": {
      "character_id": {
        "confirmed_facts": ["string"],
        "suspicions": ["string"],
        "misreads": ["string"],
        "observed_skills": ["string"],
        "emotional_impression": "string",
        "last_updated_turn": 0
      }
    }
  },
  "scene": {
    "location": "string",
    "time": "string",
    "mood": "string",
    "last_sensory_beat": "string",
    "present_characters": ["character_id"]
  },
  "characters": {
    "character_id": "PersonaDocument or path to characters/<character_id>.json"
  },
  "director_state": {
    "current_pace": "quiet | rising | tense | volatile | aftermath",
    "spotlight_history": [
      {
        "turn": 0,
        "character_id": "string",
        "beat_type": "string"
      }
    ],
    "event_cooldowns": {
      "event_key": "turns_remaining"
    },
    "new_character_budget": "integer — how many major introductions remain available in the current arc",
    "open_threads": [
      {
        "id": "string",
        "summary": "string",
        "characters": ["character_id"],
        "status": "open | heating | paused | resolved",
        "urgency": 0,
        "last_touched_turn": 0
      }
    ],
    "last_beat_type": "string or null",
    "do_not_force": ["string"]
  },
  "memory": {
    "core": {
      "premise": "string",
      "player_locked_facts": ["string"],
      "global_boundaries": ["string"],
      "active_tone": "string"
    },
    "recall": [
      {
        "turn_range": "string",
        "summary": "string",
        "characters": ["character_id"],
        "emotional_charge": 0,
        "open_loops": ["string"]
      }
    ],
    "archival_index": [
      {
        "id": "string",
        "summary": "string",
        "tags": ["character_id", "location", "secret", "relationship_edge"],
        "storage_ref": "string or file path",
        "importance": 0,
        "last_retrieved_turn": 0
      }
    ]
  },
  "relationship_graph": {
    "characterA::characterB": {
      "type": "strangers | acquaintances | friends | rivals | allies | exes | family | custom",
      "public_label": "what an observer could infer",
      "private_truth": "hidden relational context, if known to engine",
      "trust": 0,
      "tension": 0,
      "leverage": 0,
      "last_changed_turn": 0,
      "visible_to_player": false
    }
  },
  "world_events": [],
  "queued_threads": [],
  "safety": {}
}
```

---

## Event Log Entry

```json
{
  "turn": 12,
  "event_type": "scene_change | new_character | vulnerability_moment | rejection | boundary_crossed | trust_built | secret_revealed | relationship_shift | player_injection",
  "characters": ["character_id"],
  "summary": "one concise factual sentence",
  "player_visible": true,
  "importance": 0,
  "private_notes": {
    "character_id": "short private note for this character only"
  },
  "stat_deltas": {
    "character_id": { "affection": 0, "trust": 0, "guard": 0 }
  },
  "relationship_deltas": {
    "characterA::characterB": { "trust": 0, "tension": 0, "leverage": 0 }
  },
  "unlock_tier_changes": {
    "character_id": null
  }
}
```

---

## Update Protocol

1. Route slash commands or explicit meta requests through `references/meta-commands.md`.
   Pure meta commands do not increment turn count or advance story. `/newrole` is a
   story-control command and routes into `full_story`.
2. Choose `runtime_mode`: `meta`, `light_story`, or `full_story`.
3. If story advances, increment `player.turn_count`.
4. Build scoped inputs through the Context Firewall. Do not pass full state by default.
5. Run Story Director to produce `director_plan` and update planned pacing.
6. Run Scene Narrator with visible scene state and safe `director_plan`.
7. In `light_story`, skip Character Architect and Character Response unless a trigger
   appears during the scene.
8. Run Character Architect for `/newrole`, direct manual injection, Director-requested
   new characters/event actors, or permanent changes.
9. Build `response_safe_persona` for each relevant present character by physically
   removing locked-tier, hidden, unrelated, and numeric-only fields.
10. Build `player_visible_to_character` for each relevant present character from the
   protagonist profile, current scene, and that character's known facts/misreads.
11. Run Character Response once per relevant present character.
12. Append factual events for the player action, visible outcomes, stat/relationship
   changes, and any major scene changes.
13. Append private character notes to both `world_events` and each involved persona
   document's `event_log`.
14. Apply stat deltas with clamping (`0-100`) and per-turn max delta (`15` by default).
15. Recompute `current_unlock_tier` from trust:
   - `0`: trust `< 35`
   - `1`: trust `35-64`
   - `2`: trust `65-84`
   - `3`: trust `>= 85`
16. For mature-steered scenes, check the character's hidden
   `mature_minimum_affection`. If current affection is below it, add a boundary
   constraint before Character Response; do not create a separate status object.
17. Update `relationship_graph` only from events that actually change how characters
   perceive each other.
18. Update `player.known_by_characters` when a character observes, confirms, misreads,
   or revises something about the protagonist.
19. Derive qualitative relationship bands from raw stats before the next Character
   Response pass.
20. Render the player-facing prose without numeric stats or hidden fields. Numeric
   totals belong only in the preceding `GALGAME_STATE_DELTA` block.
21. Run Memory Curator after rendering to classify events into `core`, `recall`, or
   `archival_index`. This may compress older recall entries but must not delete
   player-locked facts or unresolved threads.
22. Update `director_state`: pace, spotlight history, cooldowns, open thread status,
   and new character budget.

---

## Runtime Modes

| Mode | Use for | Story effect |
|------|---------|--------------|
| `meta` | Slash commands and explicit out-of-story requests | Do not advance story unless `/resume` includes an in-world action |
| `light_story` | Movement, observation, ambience, low-stakes actions | May skip Character Architect and Character Response |
| `full_story` | Dialogue, emotional shifts, secrets, conflict, new or updated characters | Run the full module chain |

Promote `light_story` to `full_story` when a major character is directly addressed,
emotionally affected, likely to update memory/relationship state, or when a reveal,
conflict, or new persistent character appears.

Performance defaults:
- `storage_mode = inline` during ordinary terminal play.
- `autosave = false` unless the user explicitly requests autosave.
- File-backed state is for `/save`, `/load`, `/debug`, reproducible tests, or long
  sessions where persistence matters more than speed.
- Normal story turns should update state conceptually and concisely; do not emit file
  diffs or raw state patches to the player.

### Inline State Delta

In `storage_mode = inline`, each story turn must still update state. Emit a compact
state block before the narrative so the update remains in conversation context without
writing files:

```text
GALGAME_STATE_DELTA turn=<n> mode=<meta|light_story|full_story>
events: <event_type>(<character_ids>) importance=<0-5>
characters: <character_id> affection +0 -> <total>/100 trust +0 -> <total>/100 guard +0 -> <total>/100; tier <same|changed>
memory: <short private/public memory summary>
scene: <short scene state update>
```

Rules:
- Keep the block short; it is a record, not a debug explanation.
- Emit it before every non-meta story response, even when there is no meaningful state
  change. Use `no_change` for unchanged fields.
- Numeric deltas and post-turn totals must appear for every present or affected
  character. Always output `affection`, `trust`, and `guard` — all three, every
  time — even when unchanged (use `+0`). The total is the canonical value used by
  the next turn.
- Do not include absent unrelated characters just to dump the whole state.
- Do not explain or discuss the numbers unless `/debug` is requested.
- In normal play, place this block before the story text. The player may see it in the
  terminal, but it should be compact enough to scroll past quickly.
- `/status` reads the accumulated inline state and returns qualitative summaries.
- `/save` persists the current reconstructed `world_state`, using the latest totals as
  canonical character stats, plus character memories and recent inline deltas.

---

## Context Firewall Views

Build these derived views from `world_state`. Do not pass full state by default.

### `director_input`

Includes visible story state, recent visible events, open threads, director state,
surface summaries for present characters, protagonist public summary, ensemble surface
dynamics, relationship pressure summaries, and spoiler-free reveal availability flags.
Excludes full private psychology, hidden secrets, raw character notes, and locked-tier
content.

### `scene_input`

Includes current public scene state, recent visible event summary, player action,
surface facts about present characters, and safe `director_plan.scene_directives`.
Excludes private motives, hidden relationship history, and future plot answers.

### `response_safe_persona`

A new object for each Character Response call, not a full Persona Document with
"ignore this" instructions attached.

Include:
- `meta.name`, `meta.character_id`, `meta.current_unlock_tier`
- `surface`
- `visibility.known_to_player`
- `visibility.suspected_by_player` only when the current scene can plausibly trigger it
- `unlock_tiers.tier_0` through the current unlocked tier
- speech patterns, observable tells, non-spoiler boundary rules
- `stat_bands`, not raw `stats`
- recent relevant event summaries and visible/unlocked salient memories
- non-spoiler relationship summaries needed for the scene

Exclude:
- `visibility.hidden_from_player`
- unlock tiers above `current_unlock_tier`
- raw numeric `stats`
- exact `mature_minimum_affection`
- hidden causes such as core wound, core desire, Jungian shadow, or private notes unless
  the corresponding tier has unlocked
- unrelated private notes, archival refs, and other characters' private reads

If a deep cause is excluded, preserve only a visible behavior rule when needed. Example:
include "she redirects family questions into practical tasks"; exclude why family
questions hurt her.

### `player_visible_to_character`

Includes what this character can see, hear, infer from prior events, or reasonably
suspect about the protagonist. Use `player.known_by_characters.<character_id>` plus
current visible scene facts. Exclude private protagonist notes, unstated backstory,
unused skills, and the objective truth behind a character's misread.

---

## Stats And Events

Stats are stored in each Persona Document and remain engine-private. Character Response
receives qualitative bands only.

Rules:
- Stats normally move no more than ±15 per interaction. Extreme events may exceed this
  only with a clear event flag and visible reason.
- `guard` decreases slowly and increases fast.
- `trust` increases from earned evidence: vulnerability protected, boundaries respected,
  promises kept, shared risk survived, dignity preserved, or restraint shown when the
  player had leverage. Charm or persistence alone should not raise trust.
- Append significant interactions to `world_state.world_events` and mirror major
  subjective memories into involved Persona Documents.

---

## Knowledge Gates

Track three levels of truth:

- `player_visible`: the player has directly seen/heard/felt it.
- `character_private`: a character knows or feels it, but the player does not.
- `engine_private`: true for continuity, but no character or player has confirmed it.

Never move information from a private level to a visible level without an event that
justifies the reveal.

---

## Director State

Use `director_state` to prevent the Story Director from making the story noisy or
arbitrary.

- `current_pace`: controls whether the next beat should breathe, escalate, interrupt,
  or settle aftermath.
- `spotlight_history`: prevents the same character from absorbing every scene unless
  the player is deliberately focusing on them.
- `event_cooldowns`: prevents repeated alarms, entrances, accidents, reveals, or
  interruptions from feeling mechanical.
- `new_character_budget`: limits major introductions per arc. Minor background figures
  do not consume this budget unless they become persistent.
- `open_threads`: tracks unresolved promises, secrets, conflicts, invitations,
  suspicious details, and emotional pressure points.
- `do_not_force`: stores temporary or global constraints such as no forced confession,
  no unearned secret reveal, no sudden intimacy, or no scene relocation.

---

## Protagonist Visibility

The protagonist profile is full engine state. Characters receive only
`player_visible_to_character`.

Build it from:
- what is visible in the current scene: body, clothing, hair, voice, posture, movement,
  distance, smell, injuries, tools, and demonstrated skills;
- what this character has confirmed in prior events;
- what this character suspects or misreads because of their own psychology;
- skills that are public, demonstrated, or credibly known.

Never include:
- private protagonist notes;
- unstated backstory;
- skills the player has configured but not shown, unless `known_by_default` is true;
- the objective truth behind another character's misread.

---

## Memory Tiers

Use a tiered memory model inspired by long-running agent memory systems:

- `core`: small, always loaded, rarely changed. Holds setting premise, player-locked
  facts, current safety/tone boundaries, and each active character's essential identity.
- `recall`: recent compressed scene history. Holds enough detail for continuity over
  the next few turns without reloading the whole log.
- `archival_index`: older or detailed material with tags and storage references. Retrieve
  only when a character, location, secret, or relationship edge becomes relevant again.

Promotion rules:

- Promote to `core` only if forgetting it would break identity, consent/boundaries,
  locked player canon, or a central character premise.
- Keep emotionally charged but nonessential events in `recall` until they either matter
  again or age into `archival_index`.
- Archive with tags, not prose dumps. Retrieval should be targeted.
