# src/haptic/ — 9-node haptic driver & directional mapping, Layer 3 (stub)

**Responsibility:** turn a posture label (+direction, +severity) into directional vibration across the 9 haptic nodes — deterministic lookup table, with a dead-zone and an N-second persistence gate before anything fires.

**Spec:** [`../../../docs/pipeline.md`](../../../docs/pipeline.md) Layer 3. Physical node layout: [`../../../hardware/README.md`](../../../hardware/README.md).

**Planned interface (stub):**
- `haptic_init()` — bring up the 9 node drivers
- `haptic_map(Posture) -> NodePattern` — LUT: (label, direction) → which nodes + intensity
- `haptic_apply(NodePattern, severity)` — drive nodes (intensity ∝ severity)
- persistence + dead-zone gating before `haptic_apply` (no nagging on brief/normal movement)

**Notes:** 9-node layout and per-label patterns are TBD with hardware. push/pull semantics = pattern guides the wearer *which way to move*.

🚧 No implementation yet.
