# The `runs:` Palette

> **Status**: Conceptual examples throughout. These shapes illustrate the intended
> vocabulary; none has an implementation behind it, and none is blessed. Syntax
> follows the [Record Schema](https://github.com/enduring-game-standard/runs-spec/blob/main/RECORD_SCHEMA.md)
> and [DIGS](https://github.com/enduring-game-standard/runs-spec/blob/main/DIGS_EXPRESSION_LANGUAGE.md)
> specifications.

## Why a Shared Palette

Without shared shapes, every Processor needs adapters to meet every other: one
developer's position is `{x, y, z}`, another's is a three-element list, and the two
compose only through translation glue. The palette removes that friction at the
points where games most often meet — spatial state, time, input, and the
platform boundary. Everything here is optional; mixing palette shapes with custom
Fields is the normal case, not a compromise.

The palette is organized by where a shape sits relative to a Network's boundary:
**boundary Records** cross the game/platform line; **game-logic shapes** live
inside it.

---

## Foundation Shapes

```
record runs:vec2
  fields:
    x: float
    y: float

record runs:vec3
  fields:
    x: float
    y: float
    z: float

record runs:quat
  fields:
    x: float
    y: float
    z: float
    w: float    default: 1.0

record runs:transform
  fields:
    position: runs:vec3
    rotation: runs:quat
```

These use the primitive `float` (fully pinned IEEE 754 binary64). A game pinning
different arithmetic — fixed-point, a historical machine's representation —
declares its own types and shapes; the palette is a vocabulary, not a constraint on
the type system.

---

## Boundary Records

### Inbound (`requires:`) — the host writes, game logic reads

```
record runs:tick
  fields:
    number:  uint32      # monotonic tick counter
    quantum: float       # tick duration — a CONSTANT, identical every tick
```

The `quantum` field is the fixed timestep: the same value every tick of a given
build, baked as a game constant and echoed here for convenience. It is **never** a
measured wall-clock delta — measured time varies per run and destroys replay
determinism for everything downstream (see
[PATTERNS §Fixed Timestep](./PATTERNS.md#fixed-timestep)). Variable-rate rendering
interpolation is platform-side business and never enters game logic.

```
record runs:input_intent
  fields:
    move: runs:vec2
    look: runs:vec2
    jump: bool
```

Player *intent*, not device state — a gamepad, a keyboard, and a replay file all
fill the same shape. Games extend it under their own prefix
(`spacewar:player_controls` adds a hyperspace boolean) rather than asking the
palette to grow.

### Outbound (`produces:`) — game logic writes, the host reads

```
record runs:render_transform
  fields:
    position: runs:vec3
    rotation: runs:quat

record runs:audio_trigger
  fields:
    event:    uint32          # game-defined event id
    position: runs:vec3

record runs:match_result
  fields:
    finished: bool
    scores:   int32[]
```

Outbound shapes carry *what* crosses the line — the spatial and event data a
platform needs — never *how* to present it. Rendering technique, mixing, and
display timing live beyond the boundary.

---

## Game-Logic Shapes

```
record runs:velocity
  fields:
    linear:  runs:vec3
    angular: runs:vec3
```

| Shape | Field | Type | Note |
|-------|-------|------|------|
| `runs:velocity` | `linear`, `angular` | `runs:vec3` | Rate of change of transform |
| `runs:health` | `value` | `float` | Generic damageable quantity |
| `runs:team_id` | `value` | `uint32` | Affiliation grouping |

These are `state:`-side shapes: the host never reads or writes them directly. The
test for which side a Field belongs on: if any game-logic Processor reads it, it is
game logic; if it exists solely for presentation or originates from the platform,
it is a boundary Field.

---

## Primitive Processor Signatures

Conceptual signatures for the smallest shared verbs. Bodies shown are
illustrations in legal DIGS, not reference implementations.

```
#! runs-prim 1.0

processor runs:add_vec3
  inputs:
    a: runs:vec3
    b: runs:vec3
  outputs:
    result: runs:vec3

  output result.x = a.x + b.x
  output result.y = a.y + b.y
  output result.z = a.z + b.z
```

```
#! runs-prim 1.0

processor runs:scale_vec3
  inputs:
    v: runs:vec3
    factor: float
  outputs:
    result: runs:vec3

  output result.x = v.x * factor
  output result.y = v.y * factor
  output result.z = v.z * factor
```

```
#! runs-prim 1.0

processor runs:integrate_velocity
  inputs:
    position: runs:vec3
    velocity: runs:vec3
    quantum:  float
  outputs:
    position: runs:vec3

  let step = runs:scale_vec3(velocity, quantum)
  output position = runs:add_vec3(position, step.result).result
```

The same-name input/output on `integrate_velocity` is the standard idiom: the
Processor receives a value and produces its successor version — write once, a fresh
value in a fresh place.

Composition upward uses Sub-Networks, exactly as specified in
[Network Topology](https://github.com/enduring-game-standard/runs-spec/blob/main/NETWORK_TOPOLOGY.md#sub-network)
— there is no separate "bundle" syntax. A movement Sub-Network wiring
`runs:integrate_velocity` after an input-mapping Processor is itself a Processor
from the outside, with the provenance of its parts preserved by ID inside it.

---

## Naming Conventions

- Record and Field names: `snake_case`, singular nouns (`transform`, `velocity`).
- Processor names: `snake_case` verb phrases (`integrate_velocity`, `add_vec3`).
- Enum variants: `snake_case`, lowercase.
- Umbrella prefixes: short, lowercase, the game or project's natural name
  (`spacewar:`, `doom:`). Prefixes are convenience, not identity — see the
  [README](./README.md#the-runs-namespace).

## Manifest Format

The manifest — the artifact that pins a game's exact dependency closure by content
hash for baking — is owned by this layer but **not yet designed**. It will be
specified here when the first build tool forces its shape. Until then, nothing in
the ecosystem depends on a particular manifest layout, and any sketch would be
theater.

---

*MIT License*
