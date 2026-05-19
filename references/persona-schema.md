# Persona Document Schema

Full JSON schema for a character's Persona Document.
Used by: Character Architect AI (writes). Character Response AI reads only a physically
filtered `response_safe_persona` view, not this full document.

## Contents

- [Complete Schema](#complete-schema)
- [Field Writing Guidelines](#field-writing-guidelines)
- [Archetype Anti-Patterns](#archetype-anti-patterns)

---

## Complete Schema

```json
{
  "meta": {
    "name": "string — character's name",
    "character_id": "string — stable lowercase id used in world_state",
    "introduced_turn": "integer — which turn she first appeared",
    "introduced_by": "story | player",
    "current_unlock_tier": "0 | 1 | 2 | 3",
    "romanceable": "boolean",
    "adult_confirmed": "boolean — must be true for romanceable characters"
  },

  "card_compatibility": {
    "source_format": "none | chara_card_v2 | custom",
    "source_ref": "string — optional source file/id",
    "export_hint": {
      "name": "map from meta.name",
      "description": "map from surface + tier_0",
      "personality": "map from psychology.jungian.persona + defense mechanisms",
      "scenario": "map from current scene/world premise",
      "first_mes": "optional opening line/scene",
      "mes_example": "optional dialogue examples",
      "extensions": "store galgame-engine private schema here when exporting"
    }
  },

  "surface": {
    "age": "integer — must be >= 18 for romanceable or sexualized characters",
    "social_role": "string — e.g. university student, teacher, detective, idol, rival",
    "appearance": {
      "build": "string",
      "hair": "string",
      "eyes": "string",
      "distinguishing_features": "string",
      "typical_expression": "string — her default resting face",
      "movement_quality": "string — how she carries herself physically"
    },
    "voice": "string — tone, pace, volume tendencies",
    "speech_patterns": [
      "string — recurring verbal habits, vocabulary choices, things she never says"
    ],
    "typical_behavior": "string — what she does when nothing notable is happening",
    "observable_tells": {
      "stress": "string — what an observer sees when she is under pressure",
      "interest": "string — subtle cue that she is engaged",
      "discomfort": "string — how she signals she wants to disengage"
    }
  },

  "visibility": {
    "known_to_player": [
      "string — facts the player has directly observed or been told"
    ],
    "suspected_by_player": [
      "string — plausible player-facing clues, not confirmed truth"
    ],
    "hidden_from_player": [
      "string — private facts that cannot be revealed until an event or tier unlock"
    ]
  },

  "psychology": {

    "jungian": {
      "persona": "string — the mask she presents; what she wants the world to believe about her",
      "shadow": "string — what she denies or represses; the part of herself she cannot accept",
      "anima_animus": "string — her internalized ideal of intimacy; what she unconsciously seeks in a partner"
    },

    "core_wound": "string — the formative experience (loss, betrayal, neglect, shame) that shaped her defenses. Specific, not generic.",
    "core_desire": "string — what she actually wants beneath all behavior. Should create playable tension with her persona through contradiction, conflict, pride, control, duty, safety, hunger, or fear.",

    "attachment_style": {
      "type": "secure | anxious | avoidant | disorganized",
      "description": "string — how this manifests specifically for her",
      "approach_curve": "string — how she behaves as closeness increases: does she lean in, pull back, oscillate?"
    },

    "defense_mechanisms": [
      {
        "mechanism": "intellectualization | projection | reaction_formation | displacement | sublimation | denial | splitting",
        "trigger": "string — what specifically activates this defense",
        "manifestation": "string — what it looks like in behavior or speech"
      }
    ],

    "fears": [
      "string — specific fears, not abstract (not 'abandonment' but 'being left mid-sentence')"
    ],
    "pleasures": [
      "string — what genuinely lights her up, even if she hides it"
    ]
  },

  "unlock_tiers": {
    "tier_0": {
      "visible_to_response_ai": true,
      "content": "string — surface personality summary. What any observer would conclude after one encounter."
    },
    "tier_1": {
      "unlock_condition": "trust >= 35",
      "visible_to_response_ai": "boolean — computed at runtime",
      "content": "string — first crack in the persona. One specific tension between surface and interior that begins to show."
    },
    "tier_2": {
      "unlock_condition": "trust >= 65",
      "visible_to_response_ai": "boolean — computed at runtime",
      "content": "string — shadow begins surfacing. A moment where her core wound influences behavior visibly."
    },
    "tier_3": {
      "unlock_condition": "trust >= 85",
      "visible_to_response_ai": "boolean — computed at runtime",
      "content": "string — core desire exposed. She becomes capable of genuine vulnerability. What that looks like for her specifically."
    }
  },

  "stats": {
    "affection": "integer 0-100 — how much she cares for the player",
    "trust": "integer 0-100 — depth of psychological openness; determines unlock tier",
    "guard": "integer 0-100 — active resistance to vulnerability; high = clipped, deflecting behavior"
  },

  "stat_bands": {
    "affection_band": "cold | curious | warm | attached | devoted",
    "trust_band": "closed | testing | tentative | open | intimate",
    "guard_band": "unguarded | watchful | defended | armored | locked",
    "behavior_state": "string — current qualitative state passed to Character Response instead of raw stats"
  },

  "mature_minimum_affection": "integer 0-100 — engine-private necessary affection threshold before mature intimacy may be considered",

  "subjective_relationships": {
    "<other_character_id>": {
      "surface_relation": "string — what outsiders would think",
      "private_read": "string — what this character privately believes about the other",
      "trust": "integer 0-100",
      "tension": "integer 0-100",
      "leverage": "integer 0-100 — pressure, obligation, blackmail, debt, status imbalance",
      "recent_shift": "string — most recent meaningful change"
    }
  },

  "memory_model": {
    "core_memory": [
      "string — stable facts always needed for this character to stay herself"
    ],
    "salient_memories": [
      {
        "turn": "integer",
        "summary": "string",
        "emotional_charge": "integer 0-100",
        "visibility": "player_visible | character_private | engine_private"
      }
    ],
    "archival_refs": [
      {
        "id": "string",
        "tags": ["string"],
        "summary": "string"
      }
    ],
    "open_loops": [
      "string — unresolved promise, threat, secret, desire, or question"
    ],
    "boundary_rules": [
      "string — lines this character resists crossing without a strong reason"
    ]
  },

  "locked_fields": [
    "string — field names provided by the player that cannot be overwritten by the Architect"
  ],

  "event_log": [
    {
      "turn": "integer",
      "event_type": "vulnerability_moment | rejection | boundary_crossed | trust_built | secret_revealed | guard_raised",
      "summary": "string — one sentence description",
      "private_note": "string — short internal meaning for this character; never player-facing",
      "stat_deltas": { "affection": 0, "trust": 0, "guard": 0 },
      "tier_changed_to": "null or 0-3"
    }
  ]
}
```

---

## Field Writing Guidelines

### `core_wound` — do not be generic
Bad: "She was abandoned as a child."
Good: "Her mother stopped speaking directly to her after her parents divorced — all communication
went through her younger brother. She learned that her feelings were too inconvenient to address."

### `core_desire` — create playable tension
If her persona is `coldly self-sufficient`, her core desire might be `to be chosen
without having to ask`. Contradiction is useful, but tension can also come from pride,
duty, fear, control, shame, ambition, loyalty, or hunger pulling against what she shows.

### `observable_tells` — behavior, not inference
Bad: "She looks nervous."
Good: "She stops making eye contact and starts straightening objects nearby."

### `defense_mechanism.manifestation` — concrete and speakable
Bad: "She intellectualizes to avoid emotion."
Good: "When conversations turn personal, she pivots to analysis: 'Statistically, people
in situations like yours tend to…' — her voice becoming clipped and academic."

### `attachment_style.approach_curve`
This is the behavioral rule for how she handles increasing intimacy. It directly
governs the Response AI's behavior as affection/trust rises.

Examples:
- Anxious: "She leans in fast, then panics at her own vulnerability and creates
  distance to 'test' whether the player will pursue."
- Avoidant: "The closer the player gets, the more she finds reasons to be busy,
  critical, or elsewhere — not from disinterest but from fear of being seen."
- Disorganized: "She oscillates unpredictably — one moment seeking closeness,
  the next saying something deliberately cutting. There is no pattern the player
  can reliably exploit."

### `visibility`
Use this to prevent accidental spoilers. A fact can move from `hidden_from_player`
to `suspected_by_player` only after a clue appears in the scene, and from
`suspected_by_player` to `known_to_player` only after confirmation.

### `response_safe_persona`
Character Response reads a physically filtered view, not this full document. Build
`response_safe_persona` and `player_visible_to_character` using
`world-state-schema.md` → `Context Firewall Views`.

### `subjective_relationships`
These are this character's beliefs, not objective truth. If A thinks B betrayed her
but B did not, keep that mismatch; it creates playable tension. Mirror only major
changes into `world_state.relationship_graph`.

### `memory_model.salient_memories`
Keep only high-impact memories here. Routine turn history belongs in `event_log`.
Use `emotional_charge` to decide what the character remembers under pressure.

### `stat_bands`
Raw stats stay engine-private. Before calling Character Response, convert numbers into
bands and one short `behavior_state`.

Suggested bands:

| Stat | 0-19 | 20-39 | 40-64 | 65-84 | 85-100 |
|------|------|-------|-------|-------|--------|
| affection | cold | curious | warm | attached | devoted |
| trust | closed | testing | tentative | open | intimate |
| guard | unguarded | watchful | defended | armored | locked |

`guard` is inverted emotionally: higher values mean more resistance, shorter answers,
more deflection, and stronger boundary defense.

### `mature_minimum_affection`
This hidden value is a necessary threshold, not a promise of a scene. Set it from the
character's personality and defenses:
- Open or secure characters may use `70-80`.
- Guarded, avoidant, proud, duty-bound, or control-oriented characters usually use
  `80-90`.
- Anxious or disorganized characters may show attraction earlier, but mature intimacy
  still requires the threshold plus a fitting scene, consent, trust/tier, and low guard.

Never expose this exact value in normal play, `/status`, or player-facing narration.
Only `/debug` may show it.

### `card_compatibility`
Use Character Card V2-style fields only as an import/export bridge. The galgame engine's
Persona Document remains the source of truth because it needs unlock tiers, hidden
fields, relationship state, and private memory. When exporting, store engine-specific
private fields under a namespaced `extensions` object rather than flattening them into
player-visible card text.

---

## Archetype Anti-Patterns

Avoid these combinations — they produce flat characters:

| Don't do this | Why it fails |
|--------------|-------------|
| Tsundere persona + anxious attachment | Too on-the-nose; no discovery layer |
| Cold persona + avoidant attachment | Redundant — the exterior and interior say the same thing |
| Cheerful persona + secure attachment | No tension arc; she'll be easy to pursue, which the player said they don't want |
| Trauma backstory + disorganized attachment | Cliché — overused in the genre |

**Good tension combinations**:
- Cold/controlled persona + anxious attachment (she hates that she needs reassurance)
- Aggressive/competitive persona + secret desire to be protected
- Gentle/compliant persona + suppressed rage (reaction_formation)
- Hyper-independent persona + terror of being truly alone
