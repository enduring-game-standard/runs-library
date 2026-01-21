# GERS Standard Library ("The Vocabulary")

**Layer 2 of the Trinity Model**

The GERS Standard Library provides semantic agreement on data shapes. Interoperability starts here.
This library is **Modular**. A compliant engine MAY implement only a subset of modules. There is NO "Core" that mandates Time or Space.

## Modules

### 1. Module: `std:graph` (Structural)
**Concept**: Root anchors for traversing the state graph.
**Schema**: `gers:root`
- `children` (List): The top-level nodes of the simulation.

### 2. Module: `std:temporal` (Time)
**Concept**: Continuous or delta-based progression.
**Schema**: `gers:time`
- `dt` (float64): Delta duration.
- `elapsed` (float64): Accumulator.
- `ticks` (uint64): Discrete steps.
**Usage**: Required for Physics/Real-time. Omitted for Correspondence Chess.

### 3. Module: `std:spatial` (3D Euclidean)
**Concept**: Cartesian positioning in 3-space.
**Schema**: `gers:transform`
- `position` (vec3)
- `rotation` (quat)
- `scale` (vec3)
- `parent` (ref)
**Usage**: Required for 3D Games. Omitted for Spreadsheets, Text UIs, Card Games.

### 4. Module: `std:input` (HID)
**Concept**: Human Interface Devices.
**Schema**: `gers:input`
- `axis` (Map<String, Float>): 'Horizontal', 'Vertical'.
- `buttons` (Set).
