# Architecture

> System overview for Jarada — a vest-style posture-correction wearable.
> For the data schema every component shares, see [data-contract.md](data-contract.md). For pipeline internals, see [pipeline.md](pipeline.md).

## System overview

Jarada is a body-worn vest that senses spinal orientation with two IMUs, classifies posture on-device, and corrects the wearer with directional haptics — all in a tight, low-latency loop.

```
            ┌────────────────────────── VEST ──────────────────────────┐
            │                                                            │
  cervical  │   ┌──────────┐                              ┌──────────┐  │
   (neck) ──┼──▶│  BNO085  │──┐                       ┌──▶│ 9 haptic │  │
            │   └──────────┘  │  I²C (quaternion)     │   │  nodes   │  │
            │                 ▼                       │   └──────────┘  │
            │           ┌───────────┐   3-layer       │                 │
   sacrum   │   ┌──────────┐  │ MCU  │   pipeline ─────┘                 │
  (base) ───┼──▶│  BNO085  │─▶│ESP32 │                                   │
            │   └──────────┘  │/nRF52│──── BLE ───▶  companion app       │
            │                 └───────────┘            (design-only v1)   │
            │                      ▲                                      │
            │                ~1000mAh LiPo                                │
            └────────────────────────────────────────────────────────────┘
```

> **Diagram placeholder.** Replace with the rendered system diagram (export to `app/design/` or `hardware/`). Suggested: a labeled vest illustration showing the two IMU mount points and the 9 haptic-node layout.

## Components

| Block | Choice | Why | Notes |
|------|--------|-----|-------|
| **Sensing** | 2× BNO085 IMU | Onboard sensor fusion → stable quaternion output, no host-side filtering | Mounted at **cervical** (neck) and **sacrum** (base of spine) |
| **Compute** | ESP32 (prototype) | BLE built in, cheap, fast to iterate | Production roadmap → **nRF52840** for power/BLE efficiency |
| **Feedback** | 9 haptic points | Directional correction (which way to move), not a single buzz | Driven by a deterministic lookup table (Layer 3) |
| **Power** | ~1000 mAh LiPo | Target **16–24 h** runtime | Budget tracked in [../hardware/BOM.md](../hardware/BOM.md) |
| **Link** | BLE | Stream metrics + receive spinal profile | Frame schema in [data-contract.md](data-contract.md) |
| **App** | Companion app | Onboarding, spinal-profile calibration, history | **Design-only in v1** (Figma) — see [../app/README.md](../app/README.md) |

## Sensor placement & frames

- **Cervical IMU** — upper back / base of neck. Captures head-forward flexion (거북목) and neck lateral tilt.
- **Sacrum IMU** — base of spine. Captures lower-trunk lean and acts as the **reference body frame**: subtracting sacrum orientation from cervical gives spinal *alignment* independent of how the whole torso is leaning.

This two-IMU "absolute + relative" design is what lets Jarada tell *"you're slouching"* (relative) apart from *"you're leaning back in a chair"* (both absolute, but aligned).

## Data flow

1. Both IMUs stream quaternions over I²C to the MCU.
2. The MCU derives the metrics in [data-contract.md](data-contract.md) (absolute Euler per IMU + relative cervical−sacrum).
3. The **3-layer pipeline** runs on-device (see [pipeline.md](pipeline.md)).
4. Haptic nodes fire; metrics + events stream to the app over BLE.

## On-device vs off-device

| Runs on the vest (MCU) | Runs off-device |
|------------------------|-----------------|
| IMU read + metric derivation | Model **training** (`ml/training/`) |
| Layer 1 activity gating | Data collection / labeling |
| Layer 2 inference (exported model) | App UI & history (companion app) |
| Layer 3 haptic mapping | — |

The trained Layer 2 model is exported and embedded into firmware — see the **model-export handoff** in [../ml/README.md](../ml/README.md).

## Positioning (non-medical)

Jarada is a **posture monitoring & feedback** device, not a diagnostic or therapeutic medical device. This is a deliberate design constraint to stay outside 식약처 medical-device classification: no diagnosis, no medical claims, feedback framed as wellness/training. Keep this boundary in mind when writing copy, metrics, and the app.
