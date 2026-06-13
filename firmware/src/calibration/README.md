# src/calibration/ — Spinal-profile application (stub)

**Responsibility:** load the wearer's stored **spinal profile** and apply it on-device — offset metrics by neutral, normalize severity by ROM, and feed activity baselines to Layer 1 and dead-zones to Layer 3.

**Spec:** [`../../../docs/spinal-profile.md`](../../../docs/spinal-profile.md). **Capture UI** lives in the app; this module only *applies* the result.

**Planned interface (stub):**
- `profile_load(SpinalProfile* out)` — read stored profile (received via BLE / persisted)
- `profile_apply_neutral(Metrics* m, const SpinalProfile*)` — offset to per-wearer neutral
- `profile_dead_zone(const SpinalProfile*)` — dead-zone params for Layer 3
- `profile_activity_baseline(const SpinalProfile*)` — thresholds for Layer 1

🚧 No implementation yet.
