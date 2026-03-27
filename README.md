# RUNS Library

🏠 **[EGS Overview](https://github.com/enduring-game-standard)**
· 🏃 **[RUNS](https://github.com/enduring-game-standard/runs-spec)**
· 📦 **[AEMS](https://github.com/enduring-game-standard/aems-schema)**
· ⚡ **[WOCS](https://github.com/enduring-game-standard/wocs-protocol)**
· 🎼 **[MAPS](https://github.com/enduring-game-standard/maps-notation)**
· ❓ **[FAQ](https://github.com/enduring-game-standard/.github/blob/main/profile/FAQ.md)**

> **Status**: Community-Curated Examples / RFC
> **Version**: 0.1.0

---

## The Shared Palette for RUNS

The [RUNS Protocol](https://github.com/enduring-game-standard/runs-spec) defines the mandatory rules for composable game logic: Records, Fields, Processors, and Networks. Any runtime that implements these rules is fully compliant. No additional vocabulary is required.

The problem is practical. Without a shared vocabulary of data shapes, every Processor needs adapters to connect with every other Processor. Two developers who independently define a position field — one as `{x, y, z}`, the other as `{pos: [float, float, float]}` — produce components that cannot compose without translation. Shared shapes eliminate that friction.

The **RUNS Library** is an optional, community-curated collection of recommended Fields and Processors that solve this problem. It provides exact data shapes — `runs:transform`, `runs:velocity`, `runs:input_intent` — that any developer can target for instant interoperability. A movement Processor from one bundle drives transforms in another without adapters, variants compose with games they were not originally built for, and new projects inherit working primitives instead of reinventing them.

The Library is not part of the RUNS Protocol. You can ignore it entirely and build fully compliant RUNS implementations with custom schemas. But targeting Library shapes is a strong convention that makes decentralized composition practical.

All Library primitives are plain-text, self-describing definitions published as Nostr events by convention. Nostr is not incidental. It is the open commons through which Library primitives become discoverable, inheritable, and remixable across developers, projects, and generations — without gatekeepers or platform dependencies.

## Recommended Fields

These exact Field schemas provide the starting vocabulary for interoperable RUNS components. They are organized by where they sit relative to a Network's [Runtime Interface](https://github.com/enduring-game-standard/runs-spec#the-mental-model).

**Boundary Fields** cross the line between game logic and runtime. Inbound Fields are what a runtime provides before each tick (timing, player input); outbound Fields are what game logic produces for the runtime to present (spatial state, match lifecycle). Runtime authors implement inbound shapes and consume outbound shapes. Game authors target boundary shapes for instant portability across runtimes.

**Game Logic Fields** are internal to Networks — runtimes never read or write them directly. They carry the state that Processors transform during the gameplay tick. Games extend Library boundary Fields with game-specific Records when needed (e.g., `spacewar:player_controls` extends `runs:input_intent` with a hyperspace boolean; `doom:mobj` extends `runs:transform` with sector linkage). The distinction follows the spec's decision test: if a Field's output feeds back into game state that other Processors read, it is game logic; if it originates from the platform or exists solely for presentation, it is a boundary Field.

### Boundary Fields (Runtime Interface)

**Inbound** — Runtime provides, game logic reads:

| Prefix  | Field Name   | Type                                          | Description                      |
|---------|--------------|-----------------------------------------------|----------------------------------|
| `runs`  | delta_time   | float                                         | Frame timestep                   |
| `runs`  | input_intent | struct { move: vec2, look: vec2, jump: bool } | Player intent                    |

**Outbound** — Game logic produces, runtime reads:

| Prefix  | Field Name       | Type                                      | Description                              |
|---------|------------------|-------------------------------------------|------------------------------------------|
| `runs`  | render_transform | struct { position: vec3, rotation: quat } | Spatial state projected for rendering    |

### Game Logic Fields

| Prefix  | Field Name       | Type                                      | Description                     |
|---------|------------------|-------------------------------------------|---------------------------------|
| `runs`  | transform        | struct { position: vec3, rotation: quat } | Spatial placement               |
| `runs`  | velocity         | vec3                                      | Linear velocity                 |
| `runs`  | angular_velocity | vec3                                      | Rotational velocity             |
| `runs`  | health           | float                                     | Generic damageable value        |
| `runs`  | team_id          | u32                                       | Affiliation grouping            |

Schemas are versioned plain-text for serialization and distribution through the Nostr commons:

```json
{
  "runs:transform": {
    "position": { "type": "vec3", "default": [0, 0, 0] },
    "rotation": { "type": "quat", "default": [0, 0, 0, 1] }
  }
}
```

Custom Fields remain fully supported. Mix freely with exact `runs:` shapes.

## Primitive Processors

Granular, pure operations suggested in a simple declarative plain-text format (`.runs-prim`) for readability and distribution through the Nostr commons.

Example format:

```text
processor add_vec3
inputs:
  a: vec3
  b: vec3
outputs:
  result: vec3

result.x = a.x + b.x
result.y = a.y + b.y
result.z = a.z + b.z
```

Suggested primitives:
- `mul_vec3_scalar` — Scale vectors
- `integrate_velocity` — Euler integration: `transform.position += velocity * delta_time`
- `apply_input_intent` — Map intent to acceleration/velocity
- `query_entities` — Basic selection (has_field, etc.)

These are the smallest operational units. Wire them in Networks or bundle them into higher-scale Processors.

## Bundling: Multi-Scale Composition

Bundles are sub-Networks packaged as reusable meta-Processors, with provenance to their underlying primitives.

Example simple movement bundle (`.runs-bundle` graph):

```yaml
bundle basic_movement
inputs:
  transform: runs:transform
  velocity: runs:velocity
  delta_time: runs:delta_time
outputs:
  transform: runs:transform

wires:
  - scaled: mul_vec3_scalar(velocity, delta_time)
  - new_pos: add_vec3(transform.position, scaled.result)
output:
  transform.position = new_pos.result
```

Higher levels chain further: character controllers bundle movement and grounding, physics systems bundle multiple controllers and resolution. Every bundle remains a uniform Processor — recursively composable with full note ID lineage through the Nostr commons.

## Processor Authoring Styles

To enable long-term durability alongside runtime performance, the Library distinguishes two conventions — both plain-text, both targeting exact Fields.

**Gameplay Logic Processors (Recommended for Library Primitives)**
- Style: Constrained, declarative SSA-like syntax.
- Focus: Pure semantic rules — no hardware assumptions.
- Horizon: The logic is hand-reimplementable on any future platform.
- This is Tier 1 in the RUNS compilation model: the enduring artifact. Gameplay logic Processors are expressed in a formal language, compiled by runtimes, and produce identical behavior on every platform.

Example (`integrate_velocity.runs-prim`):

```text
processor integrate_velocity
inputs:
  position: runs:vec3
  velocity: runs:vec3
  delta_time: float
outputs:
  position: runs:vec3

position += velocity * delta_time
```

**Execution Realization Processors (Optional for Optimization)**
- Style: Extended declarative with hint sections.
- Focus: Platform guidance (SIMD, approximations, offload) — safely ignored by runtimes that do not support them.
- Horizon: Evolves with hardware; the core logic remains identical.
- This is Tier 2 guidance: hints for the runtime compiler. The `core` section is the enduring Tier 1 source; the `hints` section is platform-specific optimization that varies by runtime.

Example (`integrate_velocity_realized.runs-prim`):

```text
processor integrate_velocity_realized
inputs:
  position: runs:vec3
  velocity: runs:vec3
  delta_time: float
outputs:
  position: runs:vec3

core:
  position += velocity * delta_time

hints:
  vectorize: simd
  approximate: fast_mul
  target: gpu_compute if_available
```

Prefer pure semantic style for Library contributions. Realizations belong in ecosystem packages where runtime-specific optimization is warranted.

## Connection to Notation and Craft

Library Fields are the concrete shapes that [MAPS notation](https://github.com/enduring-game-standard/maps-notation) targets. The connection is direct: a `runs:transform` Field is the implementation of a MAPS State node describing spatial placement. A `basic_movement` bundle is the implementation of a MAPS Verb describing how position changes over time. A designer who sketches a combat system in MAPS notation is writing the blueprint from which Library-compatible RUNS source is built.

This bridge between notation and runtime is what makes cumulative craft practical. A designer's intent, captured in notation, maps onto shared data shapes that any compliant runtime can execute. The notation survives because the shapes it targets are plain-text, self-describing, and maintained in an open commons.

## Integration with EGS

| Component | Role | Library Relationship |
|-----------|------|---------------------|
| [RUNS Protocol](https://github.com/enduring-game-standard/runs-spec) | Mandatory execution rules | Library extends the Protocol with optional recommended schemas and Processors |
| [AEMS](https://github.com/enduring-game-standard/aems-schema) | Persistent entities | AEMS Entity/Manifestation definitions are compiled into Records carrying Library Fields; Asset/State events persist player data at lifecycle boundaries |
| [MAPS](https://github.com/enduring-game-standard/maps-notation) | Design notation | Library Fields are the concrete shapes that MAPS notation States and Verbs target |
| [WOCS](https://github.com/enduring-game-standard/wocs-protocol) | Coordination and services | WOCS coordinates bounties for Library contributions, curation, and relay hosting |

## What the Library Deliberately Excludes

The Library maintains the same restraint discipline as the RUNS Protocol:

- **No genre-specific schemas** — The Library provides neutral primitives (transform, velocity, health). Genre-specific data shapes (inventory systems, dialogue trees, faction graphs) belong in ecosystem packages.
- **No rendering or non-interactable simulation primitives** — Visual representation, non-interactable ragdolls, decorative cloth, particle effects, and other non-gameplay simulations are runtime concerns, not shared data shapes. (Boundary Fields like `runs:render_transform` define *what* crosses the game-logic/runtime line — the spatial data the runtime needs to render — but not *how* to render it. Rendering implementation remains a runtime concern.) Simulations whose output feeds back into game state (physics objects the player can manipulate, ragdolls that block doorways or can be picked up) are gameplay — they belong in Processors.
- **No networking or transport** — The Library defines local data shapes. Multiplayer synchronization is handled by implementations and coordinated through WOCS.
- **No runtime requirements** — Using Library shapes is a convention, not a compliance gate. Any runtime that implements the RUNS Protocol is fully compliant regardless of Library adoption.
- **No implementation language** — Processor definitions are declarative specifications of pure transformations. Runtimes implement them in whatever language suits their platform.

## Namespace Conventions

All Library primitives use the reserved `runs:` prefix. Implement `runs:` schemas exactly (keys, types, semantics) when targeting Library compatibility. See the [RUNS Protocol § Namespace Conventions](https://github.com/enduring-game-standard/runs-spec#namespace-conventions) for full rules and third-party prefix guidance.

## Contributing

This Library grows through community input:

- **Propose new primitives** via issues or pull requests. Library additions undergo community review to ensure neutrality and composability.
- **Coordinate through WOCS** for bountied additions, curation sprints, or contested decisions. Work Orders provide transparent funding and settlement for Library development.
- **Breaking changes** to existing `runs:` schemas require an RFC process with community review period.

Target exact `runs:` shapes for sharing; innovate beyond them for uniqueness.

## Summary

The RUNS Library provides optional, curated data shapes and Processors that make the RUNS ecosystem practically composable. Recommended Fields define the shared vocabulary. Primitive Processors demonstrate granular operations. Bundles show multi-scale composition. All definitions are plain-text Nostr events — discoverable, inheritable, and remixable through the open commons. Combined with the RUNS Protocol for execution rules, AEMS for persistent entities, MAPS for design notation, and WOCS for coordination, the Library bridges the gap between a composable architecture and a working ecosystem.

**MIT License** — Fork, extend, share freely.