# Data Contract — IMU Metric Schema

> **The single source of truth** for the metrics that flow between **firmware**, **ml**, and **app**. If you add, rename, or re-unit a field, change it *here first*, then update consumers. Don't redefine fields locally.

---

## 1. Conventions

| Aspect | Decision |
|--------|----------|
| **Sensors** | 2× BNO085, IDs `cervical` (neck) and `sacrum` (base of spine) |
| **Orientation source** | BNO085 rotation-vector (onboard sensor fusion) → unit quaternion |
| **Angle unit** | **degrees (°)** everywhere downstream (quaternions converted to Euler on-MCU) |
| **Angle convention** | intrinsic Tait–Bryan, order **pitch → roll → yaw**, right-handed |
| **Sign convention** | `pitch` + = flexion **forward/down** · `roll` + = lean to the wearer's **right** · `yaw` + = rotate **left (CCW from above)** |
| **Reference** | derived angles are reported relative to the wearer's **neutral** from their spinal profile (see [spinal-profile.md](spinal-profile.md)) |
| **Sample rate (sensor)** | `100 Hz` *(TBD — BNO085 fusion rate)* |
| **Decision rate (pipeline)** | `5 Hz` *(TBD — Layer 2 evaluation cadence)* |
| **Timestamp** | `ts_ms` — uint32, milliseconds since device boot |

> Items marked *(TBD)* are placeholders to be locked during prototyping.

---

## 2. Raw sensor stream (per IMU)

Emitted by each BNO085. `{imu}` ∈ `{cervical, sacrum}`.

| Field | Source | Type | Unit | Notes |
|-------|--------|------|------|-------|
| `{imu}.quat_w` | IMU | absolute | unitless | quaternion scalar (normalized) |
| `{imu}.quat_x` | IMU | absolute | unitless | quaternion i |
| `{imu}.quat_y` | IMU | absolute | unitless | quaternion j |
| `{imu}.quat_z` | IMU | absolute | unitless | quaternion k |
| `{imu}.accel_mag` | IMU | absolute | m/s² | linear-accel magnitude → **Layer 1** |
| `{imu}.gyro_mag` | IMU | absolute | °/s | angular-rate magnitude → **Layer 1** |
| `{imu}.cal_status` | IMU | absolute | enum 0–3 | BNO085 calibration accuracy (0=unreliable, 3=fully calibrated) |

---

## 3. Derived per-IMU orientation (absolute)

Quaternion → Euler, then offset by the wearer's neutral. These describe each segment on its own.

| Metric | Source | Type | Unit | Description |
|--------|--------|------|------|-------------|
| `neck_pitch` | cervical | absolute | ° | head forward-flexion. **Primary 거북목 metric.** + = forward/down |
| `neck_roll` | cervical | absolute | ° | head lateral tilt. + = right |
| `neck_yaw` | cervical | absolute | ° | head rotation (drift-prone; informational) |
| `trunk_pitch` | sacrum | absolute | ° | lower-trunk forward lean. + = forward |
| `trunk_roll` | sacrum | absolute | ° | lower-trunk lateral lean. + = right |
| `trunk_yaw` | sacrum | absolute | ° | lower-trunk rotation (informational) |

---

## 4. Relative metrics (cervical − sacrum)

Cervical orientation expressed **relative to** the sacrum frame. This isolates *spinal alignment* from whole-body lean and is the heart of the design.

| Metric | Source | Type | Unit | Description |
|--------|--------|------|------|-------------|
| `rel_pitch` | cervical − sacrum | relative | ° | sagittal spine curvature (upper vs lower). Slouch indicator |
| `rel_roll` | cervical − sacrum | relative | ° | upper-vs-lower lateral offset. **Primary shoulder-asymmetry metric** |
| `rel_yaw` | cervical − sacrum | relative | ° | torsional twist between segments |

---

## 5. Layer 2 posture features → labels

What the classifier consumes (subset/combination of §3–§4, all referenced to neutral) and the labels it emits. **This feature vector is the firmware↔ml contract** — see export handoff in [../ml/README.md](../ml/README.md).

**Feature vector (v1, order fixed):**
`[ neck_pitch, neck_roll, trunk_pitch, trunk_roll, rel_pitch, rel_roll, rel_yaw ]`

| Posture label | Primary metric(s) | Type | Korean |
|---------------|-------------------|------|--------|
| `forward_head` | `neck_pitch` ↑ | absolute | 거북목 |
| `shoulder_asymmetry` | `rel_roll` | relative | 어깨 비대칭 |
| `slouch` | `neck_pitch` ↑ **and** `trunk_pitch` ↑ (+ `rel_pitch`) | absolute + relative | 굽은 등 |
| `lateral_tilt` | `trunk_roll` (and/or `neck_roll`) | absolute | 측면 기울기 |
| `neutral` | all within per-wearer dead-zone | — | 정상 |

Each non-`neutral` label also carries:

| Field | Type | Unit | Notes |
|-------|------|------|-------|
| `label` | enum | — | one of the labels above |
| `confidence` | float | 0.0–1.0 | classifier probability |
| `direction` | enum | — | corrective direction for Layer 3 (e.g. `back`, `left`, `right`, `up`) |
| `severity` | float | 0.0–1.0 | normalized deviation from neutral (drives haptic intensity) |

---

## 6. BLE frame (shared payload)

Canonical JSON shape streamed device→app and used as the schema name across components. Field names match the tables above.

```json
{
  "ts_ms": 1532210,
  "activity": "still",
  "imu": {
    "cervical": { "quat_w": 0.99, "quat_x": 0.01, "quat_y": -0.04, "quat_z": 0.02,
                  "accel_mag": 9.79, "gyro_mag": 1.2, "cal_status": 3 },
    "sacrum":   { "quat_w": 1.00, "quat_x": 0.00, "quat_y": -0.01, "quat_z": 0.00,
                  "accel_mag": 9.81, "gyro_mag": 0.4, "cal_status": 3 }
  },
  "metrics": {
    "neck_pitch": 18.5, "neck_roll": 2.1, "neck_yaw": -4.0,
    "trunk_pitch": 3.2, "trunk_roll": 1.0, "trunk_yaw": 0.5,
    "rel_pitch": 15.3, "rel_roll": 1.1, "rel_yaw": -4.5
  },
  "posture": {
    "label": "forward_head", "confidence": 0.91,
    "direction": "back", "severity": 0.62
  }
}
```

> On-device, the wire format will likely be packed binary for efficiency; this JSON is the **logical contract**. Keep field names identical across binary, JSON, and the app.

---

## 7. Change policy

1. Edit this file first.
2. Bump the version below.
3. Update consumers: `firmware/src/imu/`, `firmware/src/ble/`, `ml/` feature builder, app data layer.
4. Note the change in the PR description.

**Schema version:** `v0.1.0` (draft — pre-implementation)
