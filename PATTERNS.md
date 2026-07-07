# Enduring-Game Patterns

> **Status**: Conventions, not compliance. Everything in this document is a
> recommended pattern layered on the
> [RUNS protocol](https://github.com/enduring-game-standard/runs-spec); none of it
> is a base-protocol primitive, and a game that ignores all of it can still be
> fully compliant. These patterns exist because they are what make a game cheap to
> port, safe to vary, and possible to verify — the properties enduring games need.

## Rules and Realization

The base protocol guarantees determinism structurally inside any **deterministic
region** — a subgraph whose Processors all have DIGS bodies (see
[Network Topology §Where determinism holds](https://github.com/enduring-game-standard/runs-spec/blob/main/NETWORK_TOPOLOGY.md#where-determinism-holds-deterministic-regions)).
The enduring-game pattern is to make that split deliberate, total, and named:

- **Rules Processors** — the game logic: everything that decides what happens.
  Written in DIGS, hence platform-agnostic and deterministic. Together they form
  the **Rules region** — the part of the game that is the game, and the part that
  endures.
- **Realization Processors** — the layer that realizes the game on hardware, in
  both directions: timing and input (inbound), rendering and audio (outbound), and
  multiplayer transport (remote inputs arrive as inbound boundary Records exactly
  like local ones). Verb-pure Processors, full citizens of the Network — but
  platform-tailored, determinism-exempt, and written in **whatever the target
  demands**: a throwaway script on a forgiving platform, a GPU pipeline in a
  systems language, console homebrew. The standard deliberately specifies no
  language for them. Together they form the **Realization region** — infinitely
  important, and deliberately disposable per target.

The two regions meet **only at boundary Records** — the `requires:`/`produces:`
contract. The boundary is Records-only by necessity, not preference: a noun is
paradigm-neutral and can sit on the seam between deterministic logic and platform
machinery; a verb always picks a side. The boundary Records are the *entire*
contract between the regions: a Realization Processor's only obligation to the rest
of the game is the shape of the boundary Records it fills and drains.

The names are functional, not implementational — Rules and Realization, never
"the DIGS part" and "the native part" — because the point is what each region *is
for*, not what it happens to be written in.

How much Realization code a game even has depends on the platform: a fantasy
console exposing `draw_sprite` absorbs most of it into the host; a raw framebuffer
leaves it all game-authored. Both are conforming; the boundary contract is
identical either way.

The deterministic Rules region is what makes lockstep and rollback netcode
possible — same inputs, same state, on every peer — while containing no transport
at all. One care point: the core determinism guarantee is *evaluation-order*
determinism (identical results on one build). **Cross-machine** lockstep
additionally needs *arithmetic determinism* — fully pinned numeric types — which
every DIGS primitive type and every game-defined type provides under strict
evaluation, and hardware-native substitution forfeits (see Variants, below).

## The Two Swap Axes: Variant and Port

The Rules/Realization split creates two orthogonal ways to derive a new build from
a game, and the distinction is load-bearing for the whole ecosystem:

- **Variant** — open the Network, swap or change **Rules Processors**, bake. The
  rules changed; the realization may be untouched. This is how enduring games have
  always evolved — chess and soccer accrue house rules; variation is first-class
  and permissionless, never second-class derivation. (The word is *variant*. The
  act of starting one may be called forking; the result is a variant.)
- **Port** — re-author the **Realization region** wholesale for new hardware; the
  Rules Processors and boundary Records are untouched. Same game, new machine.

A **pure Port** preserves the Rules region's three reach-contracts bit-for-bit —
**arithmetic** (identical numeric behavior), **execution** (identical algorithms),
and **memory** (identical bounds). Its Rules-region output is bit-identical to the
original's: provably the same game.

A re-target that *cannot* hold those contracts — hardware-native floating point
whose results differ, shrunken entity caps to fit RAM — changes Rules Processors,
and is therefore a **Variant Port**: a variant made to fit a target. This is a
distinction the industry conflates: two console builds with divergent arithmetic
are conventionally both called "ports," but they are different games by output,
and RUNS requires the split to be named. Likewise:

- A **re-vibe** — reinterpreting source "close enough," reducing fidelity — is a
  variant, never a port.
- A **divergent compilation** — a build substituting non-bit-identical algorithms
  under a deviation manifest (see
  [DIGS §Divergent Compilation](https://github.com/enduring-game-standard/runs-spec/blob/main/DIGS_EXPRESSION_LANGUAGE.md#divergent-compilation))
  — is a variant, with the manifest as its machine-readable diff from canon.

"Any rules on any hardware" is mechanical exactly where the target can hold the
identical Rules region: porting then touches only Realization. Where the target
cannot, the tradeoff lands in the Rules and the result is a variant. Some targets
are out of a game's reach by its own declared contracts, and the type declarations
say so.

## Fixed Timestep

Rules-region time is **data, at a fixed rate**: a constant tick quantum (a game
constant) or a tick counter, delivered as an inbound boundary Record
([PALETTE §runs:tick](./PALETTE.md#inbound-requires--the-host-writes-game-logic-reads)).
A measured wall-clock delta varies per run and silently destroys replay
determinism for every Processor downstream of it. Render interpolation,
substepping against real elapsed time, and frame pacing are Realization-region
concerns; they never enter Rules-region inputs.

## Thread Shared Things

The single-assignment law (*write once, never overwrite; pass shared things
hand-to-hand*) is base protocol; the pattern here is recognizing what counts as a
shared thing and threading it on purpose:

- The **PRNG** is the canonical case: thread its state through every consumer —
  `prng` in, `prng` out, hand to hand — and the consumption order becomes a derived
  fact of the wiring instead of a scheduling accident. Determinism follows; so does
  replayability.
- **Accumulators** (scores, damage tallies), **queues** (spawn requests), and
  **pools** (object tables) are the same shape: fold them through their writers.
- The smell that something needs threading: the wiring seems to need an *authored*
  execution order, or a place is written twice per tick. Both mean a shared
  resource is being overwritten instead of passed. Source material ported from
  register machines is full of this — overwriting and hand-scheduling were *that
  hardware's* realization workarounds, not game rules, and they do not survive
  translation into the Rules region.

## Perceive → Route → Act

Routing discipline for dispatch (the base laws are in
[Network Topology §Guards](https://github.com/enduring-game-standard/runs-spec/blob/main/NETWORK_TOPOLOGY.md#guards-and-the-two-laws)):

- **Perceive**: a Processor computes the fact and writes it as a Field — a state
  tag, `can_see_player`.
- **Route**: arc guards read that Field and pick exactly one target. Guards never
  compute.
- **Act**: the target Processor does the behavior, ignorant of the state machine
  that routed to it — which is what keeps it composable into other games.

Use **closed enums as discriminants** wherever the original logic has a state
machine. A closed enum makes the partition checkable — a tool proves every state
is handled exactly once — where a string or integer discriminant can never be
proven complete and drags an `else` arc everywhere. A state machine in RUNS is a
Network of guarded arcs, never a dispatcher Processor.

## The Decomposition Gradient

Compliance never depends on Processor granularity: a monolith re-expressing a
whole foreign routine verb-for-verb in DIGS and an atomic `add_vec3` are equally
valid, and the monolith that passes its test vectors is flawlessly the game —
merely undecomposed. The convention is a direction of travel, not a gate:

1. **Fat body** — one Processor, one foreign routine, literal translation. The
   correct first station for every conversion: port now, decompose later.
2. **Split** — factor the body into sub-Processor calls
   (`let r = ns:proc(x)`).
3. **Lift** — promote routing decisions to arc guards; publish the pieces to the
   commons, referenced by ID.
4. **Bundle** — group the pieces as a Sub-Network: a Network used as a Processor
   from the outside, with the provenance of its atoms preserved by ID inside it.
   The popular unit is usually the bundle; the atoms stay addressable, the way a
   package ecosystem keeps both.

Composition follows behavior, not original intent: pieces compose on what they
compute, not on what any of them was once for. The far end of the gradient — games
assembled largely from shared, commons-published primitives — is the ecosystem's
aspiration, and no game is required to arrive there to be correct.

## Hardware Timing as Game Logic

Some historical games used CPU performance itself as a mechanic — entity updates
speeding up as the entity count drops, because the loop finished faster. In RUNS
that behavior is **Rules-region data**: a lookup table or formula in a `state:`
Record reproducing the speed curve, derived from profiling the original hardware.
The timing profile becomes explicit, portable game logic; nothing couples the
Rules region to how fast any actual machine runs.

---

*MIT License*
