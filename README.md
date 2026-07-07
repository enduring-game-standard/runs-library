# RUNS Library

🏠 **[EGS Overview](https://github.com/enduring-game-standard)** · 📦 **[AEMS](https://github.com/enduring-game-standard/aems-schema)** · 🎯 **[AEMS Conventions](https://github.com/enduring-game-standard/aems-conventions)** · 🔧 **[RUNS](https://github.com/enduring-game-standard/runs-spec)** · 📖 **[RUNS Library](https://github.com/enduring-game-standard/runs-library)** · ⚡ **[WOCS](https://github.com/enduring-game-standard/wocs-protocol)** · 🎼 **[MAPS](https://github.com/enduring-game-standard/maps-notation)** · 🎶 **[MAPS Library](https://github.com/enduring-game-standard/maps-library)** · ❓ **[FAQ](https://github.com/enduring-game-standard/.github/blob/main/profile/FAQ.md)** · 🔤 **[Glossary](https://github.com/enduring-game-standard/.github/blob/main/profile/README.md#glossary)**

> **Status**: Draft. Everything here is a **conceptual example** — unimplemented,
> possibly wrong, not blessed. Real implementations, once they exist on the commons,
> will be indexed here by ID; these examples hold no privileged claim to become them.

## What This Repo Is

The [RUNS protocol](https://github.com/enduring-game-standard/runs-spec) is the
kernel: four primitives (Records, Fields, Processors, Networks) plus DIGS.
Compliance is those and nothing more. This repo is the **conventions layer** above
that kernel — optional, recommended, and where shared practice lives:

- **The `runs:` palette** ([PALETTE.md](./PALETTE.md)) — recommended common shapes
  (Records, Fields) and Processor signatures. Targeting them buys
  interoperability: two components that speak `runs:transform` compose without
  adapters. Ignoring them costs nothing but adapters.
- **The patterns** ([PATTERNS.md](./PATTERNS.md)) — the architecture conventions
  that make a RUNS game endure: the Rules/Realization split, the Variant/Port
  taxonomy, fixed timestep, threading, dispatch discipline, and the decomposition
  gradient.
- **Namespace and naming conventions, and the manifest format** — governance of the
  one reserved prefix, and the shapes of the ecosystem's packaging metadata.

This repo is a **curation index, not a code vault**. It owns interop *contracts*
(shapes and signatures), naming, and — once the commons holds real bodies — a
blessed-by-ID index mapping convention names to the content hashes of reference
implementations. The bodies themselves live on the commons, where anyone can
publish a rival and improvement needs nobody's permission. The repo points; it does
not host.

## The Three Tiers

| Tier | Lives in | Binding? |
|------|----------|----------|
| **Protocol** — Records, Fields, Processors, Networks, DIGS, the `runs:` reservation | [runs-spec](https://github.com/enduring-game-standard/runs-spec) | Mandatory for compliance |
| **Conventions** — the `runs:` palette, patterns, naming, manifest format | this repo | Optional, recommended |
| **Ecosystem** — community bundles, games, variants | the commons (Nostr events) | Permissionless |

## The `runs:` Namespace

Three properties, kept distinct:

- **Reserved** — only the standard defines `runs:` keys. Anti-squatting.
- **Semantically fixed** — if you reference a `runs:` key, it carries exactly the
  standard's keys, types, and semantics. You may not redefine it.
- **Optional to use** — targeting `runs:` shapes buys interop; it is never required.
  A game built entirely on its own umbrella prefix (`spacewar:`, `pong:`) is fully
  compliant.

`runs:` is deliberately small — the coreutils of the ecosystem, not its catalog. It
stays minimal and foundational; the ecosystem grows in umbrella prefixes, not by
swelling `runs:`.

Umbrella prefixes themselves need no governance: identity in this ecosystem is
cryptographic (author keys and content hashes), references resolve by ID, and two
authors publishing `pong:object` simply produces two distinct artifacts. The prefix
is readable convenience, never identity. `runs:` is the lone governed prefix because
its *string* carries fixed shared semantics that must mean the same thing for
everyone.

## Contributing

This layer is the ecosystem's open door: proposals for shared shapes, signatures,
and patterns are welcome as issues and pull requests. Decisions here are currently
made by the standard's author — there is no committee process, and additions are
weighed for neutrality, minimality, and composability. The base protocol next door
is closed; this is where outside input lands.

**MIT License**
