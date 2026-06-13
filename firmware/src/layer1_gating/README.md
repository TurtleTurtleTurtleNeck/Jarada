# src/layer1_gating/ — Activity gating, Layer 1 (stub)

**Responsibility:** deterministic, threshold-based activity detection — `still` / `walking` / `running` / `exception`. Gates the rest of the pipeline: posture is only evaluated/corrected when appropriate (typically `still`).

**Spec:** [`../../../docs/pipeline.md`](../../../docs/pipeline.md) Layer 1. **No AI** — pure thresholds on `accel_mag` / `gyro_mag`, tuned against the wearer's activity baselines from the spinal profile.

**Planned interface (stub):**
- `activity_update(const ImuFrame*) -> Activity` — current state from accel/gyro magnitudes
- `gate_open(Activity) -> bool` — whether Layer 2/3 should run

**Notes:** thresholds are TBD; seed from `activity_baseline` in the spinal profile.

🚧 No implementation yet.
