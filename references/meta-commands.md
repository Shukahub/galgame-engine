# Meta Commands

Use this reference when the player sends a slash command or an explicit out-of-story
request such as "pause", "show status", "change my profile", "create a new role",
or "export her card".

## Contents

- [Router](#router)
- [Commands](#commands)
- [Command Details](#command-details)
- [Non-Slash Meta Requests](#non-slash-meta-requests)

---

## Router

Run the Meta Command Router before Story Director.

```json
{
  "raw_input": "",
  "is_meta_command": true,
  "command": "/status",
  "args": "",
  "mode": "inspect | update | debug | persistence | resume | story_control",
  "advances_story": false
}
```

Rules:
- Pure meta commands do not call Story Director, Scene Narrator, Character Architect, or
  Character Response.
- `/newrole` is not pure meta. It is a story-control command: parse it first, then route
  to `full_story` with `trigger: player_newrole`.
- Meta updates change engine state, not in-world facts, unless the command explicitly
  says the player acts or announces something in the scene.
- State visibility still applies. `/status` and `/memory` default to player-safe output.
  `/debug` is the explicit engine-private mode.
- Character knowledge is not magically updated by meta edits. If `/profile` changes a
  visible trait, characters learn it only when they observe it or already know it.
- After handling a command, keep the session paused if `world_state.meta.paused = true`.

---

## Commands

| Command | Mode | Advances story | Purpose |
|---------|------|----------------|---------|
| `/pause` | inspect | no | Pause story mode and enter meta mode |
| `/resume` | resume | optional | Leave meta mode; continue from the current scene |
| `/status` | inspect | no | Show player-safe scene, relationship, and pacing summary |
| `/profile` | inspect/update | no | View or modify protagonist profile |
| `/memory` | inspect | no | Show recent events, open threads, and known impressions |
| `/tone` | update | no | Adjust desired tone and pacing preferences |
| `/debug` | debug | no | Show engine-private state slices when explicitly requested |
| `/save` | persistence | no | Persist current session state if file access exists |
| `/load` | persistence | no | Load a previous session state if file access exists |
| `/newrole` | story_control | yes | Explicitly create and introduce or queue a new persistent character |
| `/import-card` | update | no | Import Character Card V2-style data into a Persona Document |
| `/export-card` | persistence | no | Export a character through the compatibility mapping |

---

## Command Details

### `/pause`
Set `world_state.meta.paused = true`. Reply out of story. Do not advance turn count.

### `/resume`
Set `world_state.meta.paused = false`. If the user includes an in-world action after
`/resume`, process that action as the next story turn; otherwise provide a short
player-safe recap and wait.

### `/status`
Show:
- current location/time/mood;
- present characters and visible relationship posture;
- open threads in spoiler-free wording;
- qualitative relationship bands, never raw stats unless `/debug` is also requested.

Do not reveal hidden motives, private notes, locked-tier content, exact numeric stats,
or mature-route affection thresholds.

### `/profile`
Without args, show protagonist profile fields and each character's known/misread view
in player-safe form.

With args, update `world_state.player.profile`:

```json
{
  "profile_patch": {},
  "locked_fields_added": [],
  "visibility_effect": "private_engine_update | visible_next_scene | already_visible"
}
```

Do not update `known_by_characters` unless the change was already visible or explicitly
declared in-world.

### `/memory`
Show recent player-visible events, unresolved open threads, and character-specific
known impressions. Hide engine-private archival refs and private character notes unless
`/debug` is requested.

### `/tone`
Update `world_state.meta.tone_preferences` and optionally `world_state.memory.core.active_tone`.
Use this for pacing flavor, not immediate plot control.

Examples:
- `/tone more romantic, less violent`
- `/tone slower, more sensory, more psychological`
- `/tone dangerous but no gore`

### `/debug`
Explicit engine-private mode. May show selected raw state slices:
- `director_plan`
- `response_safe_persona`
- `player_visible_to_character`
- `world_state` patch
- raw stat values
- unlock tiers

Keep debug output structured and concise. Do not enter debug mode unless the command is
explicit.

### `/save` and `/load`
Use file-backed state only on explicit `/save`, `/load`, autosave opt-in, or testing
requests. Ordinary story turns should not write files.

If filesystem access is unavailable, summarize the current serializable state and
explain that persistence is inline only.

Suggested behavior:
- `/save`: reconstruct current `world_state` from inline state and recent
  `GALGAME_STATE_DELTA` blocks. Use the latest per-character total stats as canonical,
  write world state and persistent character files, then set
  `world_state.meta.persistence.last_save_ref`.
- `/load`: load a prior save into inline state.
- `/save autosave on`: set `world_state.meta.persistence.autosave = true`.
- `/save autosave off`: set `world_state.meta.persistence.autosave = false`.

### `/newrole`
Explicitly create a persistent character from player-provided specs.

Examples:
- `/newrole 酷酷的鲻鱼头大姐姐，秋叶原同人店店主`
- `/newrole queue 之后在图书馆出现：沉默寡言的研究生，银色短发`

Behavior:
- Parse supplied appearance, role, temperament, relationship, location, and entrance
  details into `player_provided_specs`.
- Treat supplied specs as locked visible canon unless unsafe.
- If `queue` is present, store the request in `queued_threads` and let Story Director
  introduce her later.
- Otherwise route to `full_story`, run Character Architect with
  `trigger: player_newrole`, then render her entrance through Scene Narrator and
  Character Response.
- Update director cooldowns so Story Director does not immediately stack another major
  introduction unless the player asks.

### `/import-card`
Route to Character Architect with `source_character_card`. Preserve player-provided
locked fields and safe adult constraints.

### `/export-card`
Use `persona-schema.md` → `card_compatibility.export_hint`. Store galgame-engine private
fields under namespaced `extensions`, not in player-visible card prose.

---

## Non-Slash Meta Requests

Treat natural language as meta if the player clearly asks out of story:
- "pause"
- "show her current state"
- "change my profile"
- "what does she know about me"
- "make the tone darker"

If the input could be either in-world dialogue or meta, prefer story mode unless it uses
a slash command or explicitly says "out of story", "meta", or "pause".
