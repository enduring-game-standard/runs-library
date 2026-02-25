# RUNS Library

🏠 **[EGS Overview](https://github.com/enduring-game-standard)**  
· 🏃 **[RUNS Spec](https://github.com/enduring-game-standard/runs-spec)**  
· 📦 **[AEMS](https://github.com/enduring-game-standard/aems-schema)**  
· ⚡ **[WOCS](https://github.com/enduring-game-standard/wocs-protocol)**  
· 🎭 **[MAPS](https://github.com/enduring-game-standard/maps-notation)**  
· ❓ **[FAQ](https://github.com/enduring-game-standard/.github/blob/main/profile/FAQ.md)**

> **Status**: Community-Curated Examples / RFC  
> **Version**: 0.1.0

## The Shared Palette for RUNS

The **RUNS Library** provides a collection of **recommended, optional primitives**—data shapes (Fields) and granular Processor examples—that accelerate interoperability across games, mods, and runtimes.

It is **not** part of the mandatory RUNS Protocol. You can ignore it entirely and still build fully compliant RUNS implementations with custom schemas.

Targeting these exact primitives unlocks instant composability: a movement Processor from one bundle drives transforms in another, mods integrate seamlessly, and the ecosystem shares reusable "pigments" without prior negotiation.

Curated openly by the community, this library evolves organically. Widely adopted primitives become the de facto foundation for new projects—lowering barriers while preserving complete freedom.

For permissionless, enduring distribution, all examples here are plain-text: serializable schemas, declarative Processor definitions, and bundle graphs—published by convention as Nostr events with provenance chains.

## Why Use the Standard Library?

- **Instant Interoperability** — Exact shared Fields enable unrelated packages to compose without adapters.
- **Multi-Scale Remix** — Granular atoms wire into mid-level bundles, which bundle into systems—all uniform and provenance-chained.
- **Rapid Prototyping** — Beginners combine high-level bundles for playable sketches in hours.
- **Ecosystem Momentum** — Popular primitives attract more tools, optimizations, and extensions.
- **Centuries-Scale Endurance** — Plain-text, self-describing chains survive platforms and authors.

Using the library is a strong convention, not a requirement. It’s the shared palette that makes decentralized, timeless composition practical.

## Core Philosophy

- **Minimal and Neutral** — No genre bias, no baked-in performance assumptions.
- **Plain-Text Nostr-Native** — Human-readable, relay-distributable, tamper-proof.
- **Granular to Hierarchical** — Ultra-fine primitives bundle into composites, preserving explicitness to the atoms.
- **Community-Driven** — Proposals via PRs, issues, or WOCS coordination.
- **Exact for Interop** — Implement these `runs:` shapes precisely when targeting shared ecosystem packages.

## Namespace Best Practices

All library primitives use the reserved `runs:` prefix—see the [RUNS Spec Namespace Conventions](https://github.com/enduring-game-standard/runs-spec#namespace-conventions) for full rules.

- Implement `runs:` schemas **exactly** (keys, field names, types, semantics) for public packages claiming library compatibility.
- Do **not** deviate or rename in shared bundles—exact matching guarantees seamless composition.
- Ecosystem extensions: Use custom umbrella prefixes with bundle manifests (Nostr events anchoring version, note_id, dependencies).
- Publish primitives and bundles as plain-text events for permissionless replication and provenance chaining—from raw atoms to full systems.

## Recommended Fields (The Common Tongue)

These exact Field schemas provide the starting vocabulary. Define them on Records as needed.

| Prefix  | Field Name       | Type                                      | Description                     |
|---------|------------------|-------------------------------------------|---------------------------------|
| `runs`  | transform        | struct { position: vec3, rotation: quat } | Spatial placement               |
| `runs`  | velocity         | vec3                                      | Linear velocity                 |
| `runs`  | angular_velocity | vec3                                      | Rotational velocity             |
| `runs`  | delta_time       | float                                     | Frame timestep (runtime-provided)|
| `runs`  | input_intent     | struct { move: vec2, look: vec2, jump: bool } | Player intent                |
| `runs`  | health           | float                                     | Generic damageable value        |
| `runs`  | team_id          | u32                                       | Affiliation grouping            |

Schemas are versioned plain-text for serialization and Nostr distribution:

```json
{
  "runs:transform": {
    "position": { "type": "vec3", "default": [0, 0, 0] },
    "rotation": { "type": "quat", "default": [0, 0, 0, 1] }
  }
}
```

Custom Fields remain fully supported—mix freely with exact `runs:` shapes.

## Primitive Processors (The Pigments)

Granular, pure operations—suggested in a simple declarative plain-text format (`.runs-prim`) for readability and relay distribution.

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

These are the atomic pigments—wire them in Networks or bundle into higher-scale Processors.

## Bundling Examples (Mixing Paints)

Bundles are sub-Networks packaged as reusable meta-Processors, with provenance to underlying primitives.

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

Higher levels chain further: character controllers bundle movement + grounding, physics systems bundle multiple controllers + resolution. Every bundle remains a uniform Processor—recursively composable with full note ID lineage.

## Processor Authoring Styles (Semantic vs. Realization)

To enable extreme longevity alongside performance, the library distinguishes two **conventions**—both plain-text, both targeting exact Fields:

**Gameplay Logic Processors (Recommended for Library Primitives)**  
- Style: Constrained, declarative SSA-like syntax.  
- Focus: Pure semantic rules—no hardware assumptions.  
- Horizon: Millennia—hand-reimplementable centuries later.  

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
- Focus: Platform guidance (SIMD, approximations, offload)—safely ignored.  
- Horizon: Decades—evolves with hardware.  

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

Prefer pure semantic style for library contributions. Realizations belong in ecosystem packages.

## Contributing

This library lives on community input:

- Propose new primitives via issues/PRs.
- Use WOCS for bountied additions or curation.
- Fork and specialize—adoption and relay replication decide endurance.

Target exact `runs:` shapes for sharing; innovate beyond for uniqueness.

## Summary

The RUNS Library is your optional starter palette: exact shared Fields and granular primitives that make enduring games easier to remix and compose—from atomic pigments to timeless masterpieces.

Mix freely. Build something that outlives us all.

**MIT License** — Fork, extend, share freely.