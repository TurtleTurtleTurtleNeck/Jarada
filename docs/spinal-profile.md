# Spinal Profile — Calibration & Registration Spec

> A **Face-ID-style** one-time (re-runnable) setup that teaches the vest *this wearer's* body: their range of motion, their neutral, their target posture, and baseline activity values. Without it, "good posture" is a textbook average; with it, it's *theirs*.

---

## Why per-wearer calibration

People differ in resting neck angle, torso length, shoulder height, and natural range. A fixed threshold ("neck_pitch > 20° = 거북목") mislabels someone whose neutral is already 15°. The spinal profile makes every metric in [data-contract.md](data-contract.md) **relative to the wearer's neutral**, so labels and dead-zones are personalized.

---

## What the profile captures

| Block | Captured | Used by |
|-------|----------|---------|
| **Range of motion (ROM)** | min/max `neck_pitch`, `neck_roll`, `trunk_pitch`, `trunk_roll` the wearer can reach | normalize `severity`; sanity-bound metrics |
| **Neutral** | the wearer's relaxed, aligned orientation (per-metric zero point) | reference offset for all derived metrics |
| **Target posture** | the "good" posture to coach toward (may differ slightly from neutral) | Layer 2 reference / Layer 3 dead-zone center |
| **Activity baselines** | `accel_mag` / `gyro_mag` signatures for walk / sit / stand | Layer 1 threshold tuning ([pipeline.md](pipeline.md)) |

---

## Registration flow (guided by the app)

> Each step is a guided prompt with a hold timer; the app shows live feedback. Steps and durations are **placeholders — TBD**.

1. **Fit check** — confirm both IMUs report `cal_status ≥ 2` and are on-body.
2. **Neutral capture** — "Sit/stand relaxed and look ahead." Hold `Ns`; average orientation → neutral.
3. **Target capture** — "Now sit up in your best comfortable posture." Hold `Ns` → target.
4. **ROM sweep** — guided: look down/up (pitch), tilt ear-to-shoulder L/R (roll), gentle rotate (yaw). Record min/max per metric.
5. **Activity baselines** — short **sit**, **stand**, and **walk** samples → store `accel_mag`/`gyro_mag` ranges for Layer 1.
6. **Confirm & store** — review summary, save profile (see schema below).

**Re-calibration:** prompt periodically or on demand (vest re-fit, posture changes over weeks).

---

## Stored profile schema (draft)

```json
{
  "profile_version": "v0.1.0",
  "wearer_id": "<opaque-id>",
  "created_ts": "<ISO-8601>",
  "neutral":  { "neck_pitch": 0.0, "neck_roll": 0.0, "trunk_pitch": 0.0, "trunk_roll": 0.0,
                "rel_pitch": 0.0, "rel_roll": 0.0 },
  "target":   { "neck_pitch": 0.0, "rel_pitch": 0.0, "trunk_pitch": 0.0 },
  "rom": {
    "neck_pitch":  { "min": -40.0, "max": 60.0 },
    "neck_roll":   { "min": -35.0, "max": 35.0 },
    "trunk_pitch": { "min": -20.0, "max": 45.0 },
    "trunk_roll":  { "min": -30.0, "max": 30.0 }
  },
  "dead_zone": { "pitch_deg": 8.0, "roll_deg": 6.0 },
  "activity_baseline": {
    "sit":   { "accel_mag": [9.6, 10.0], "gyro_mag": [0, 5] },
    "stand": { "accel_mag": [9.6, 10.0], "gyro_mag": [0, 8] },
    "walk":  { "accel_mag": [8.5, 12.0], "gyro_mag": [10, 60] }
  }
}
```

> Values above are illustrative placeholders. Field names follow [data-contract.md](data-contract.md).

---

## Where it lives

- **Capture / UI:** companion app (design-only in v1 — [../app/README.md](../app/README.md)).
- **Apply:** firmware references the stored profile to offset metrics and set Layer 1 thresholds + Layer 3 dead-zones — [../firmware/src/calibration/](../firmware/src/calibration/).
- **B2B note:** the profile + over-time history is the data product for posture clinics / 한방병원.
