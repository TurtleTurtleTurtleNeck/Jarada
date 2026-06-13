# The Three-Layer Pipeline

> Jarada's core design: **one AI layer sandwiched between two deterministic layers.** The AI only classifies posture — it never decides *when* to act or *how* to buzz. Everything safety- and behavior-critical stays deterministic and debuggable.

```
 IMU metrics (data-contract.md)
        │
        ▼
┌───────────────────────────┐
│ Layer 1 — Activity Gating │  deterministic (accel/gyro thresholds)
│  still / walking /        │  ── if not "still" (or relevant), short-circuit:
│  running / exception      │      don't evaluate posture, don't correct
└───────────┬───────────────┘
            │ gate open
            ▼
┌───────────────────────────┐
│ Layer 2 — Posture Classify│  AI (RandomForest / small MLP)
│  forward_head · shoulder  │  ── outputs a posture label (+ confidence)
│  _asym · slouch · tilt    │
└───────────┬───────────────┘
            │ posture label
            ▼
┌───────────────────────────┐
│ Layer 3 — Haptic Mapping  │  deterministic (lookup table)
│  label → directional node │  ── dead-zone + N-sec persistence gate
│  vibration (push/pull)    │      before anything fires
└───────────┬───────────────┘
            ▼
     9 haptic nodes
```

---

## Layer 1 — Activity gating (deterministic, no AI)

**Goal:** decide whether posture evaluation is even meaningful right now. You don't correct someone's neck angle while they're running.

**Inputs:** accelerometer + gyroscope magnitudes from both IMUs (`accel_mag_*`, `gyro_mag_*` — see [data-contract.md](data-contract.md)).

**Method:** threshold-based state machine. No learned model — fully deterministic and inspectable.

| State | Rough signature | Pipeline behavior |
|-------|-----------------|-------------------|
| `still` | low accel variance, low gyro | ✅ evaluate posture (the main case) |
| `walking` | periodic moderate accel | ⏸️ suppress correction (posture meaningless during gait) |
| `running` | high accel/gyro | ⏸️ suppress correction |
| `exception` | transient spikes / sensor fault / off-body | ⏸️ suppress; flag for review |

> Thresholds are **TBD** and tuned against the calibration baselines (walk/sit/stand) captured during spinal-profile registration — see [spinal-profile.md](spinal-profile.md).

**Stub:** [`../firmware/src/layer1_gating/`](../firmware/src/layer1_gating/)

---

## Layer 2 — Posture classification (AI)

**Goal:** given that we *should* evaluate, label the current posture.

**Model:** RandomForest (baseline) or a small MLP. Chosen for: tiny footprint, fast on-MCU inference, interpretable feature importances. Trained off-device, exported into firmware (see handoff in [../ml/README.md](../ml/README.md)).

**Inputs:** the absolute + relative IMU metrics defined in [data-contract.md](data-contract.md), referenced against the wearer's neutral from their spinal profile.

**Output labels (target set):**

| Label | Driven mainly by | Korean |
|-------|------------------|--------|
| `forward_head` | cervical pitch forward vs neutral | 거북목 |
| `shoulder_asymmetry` | relative roll (cervical − sacrum) | 어깨 비대칭 |
| `slouch` | cervical **and** sacrum pitch both forward | 굽은 등 |
| `lateral_tilt` | trunk roll (and/or neck roll) to one side | 측면 기울기 |
| `neutral` | within per-wearer dead-zone of target | 정상 |

> Labels are **multi-class for v1**; multi-label (e.g. forward_head *and* lateral_tilt at once) is a roadmap item — see [roadmap.md](roadmap.md). Each non-neutral label also carries a **direction** so Layer 3 knows which way to push.

**Stub:** [`../ml/training/`](../ml/training/) (training) → [`../ml/inference/`](../ml/inference/) (export)

---

## Layer 3 — Haptic mapping (deterministic)

**Goal:** turn a posture label into a *correction the wearer can act on* — a directional push/pull, not a generic alert.

**Method:** lookup table from `(label, direction)` → which of the 9 nodes fire, and how.

- **Directional push/pull:** nodes are positioned so a pattern says "move *this* way" (e.g. forward_head → posterior-neck nodes pull the head back).
- **Continuous feedback:** intensity can track severity rather than a single on/off pulse.
- **Dead-zone:** no correction while within the wearer's neutral tolerance — prevents nagging on tiny, normal movement.
- **Persistence gate:** a posture must hold for **N seconds** before any node fires — prevents reacting to a momentary reach or glance.

| Posture label | Correction intent | Node group (placeholder) |
|---------------|-------------------|--------------------------|
| `forward_head` | pull head/neck back | posterior cervical nodes |
| `shoulder_asymmetry` | lift the dropped side | the low-side shoulder nodes |
| `slouch` | extend upper back | mid/upper posterior nodes |
| `lateral_tilt` | counter the lean | contralateral nodes |
| `neutral` | (none) | — |

> The exact 9-node layout and per-label patterns are **TBD** — to be defined with the hardware node map. See [../firmware/src/haptic/](../firmware/src/haptic/) and [../hardware/README.md](../hardware/README.md).

**Stub:** [`../firmware/src/haptic/`](../firmware/src/haptic/)

---

## Why this split?

- **Predictability:** the only learned component is classification. *When* to act and *how* to buzz are deterministic — easy to test, audit, and reason about for a body-worn device.
- **Safety / positioning:** keeping correction logic deterministic supports the non-medical, no-surprises framing (see [architecture.md](architecture.md#positioning-non-medical)).
- **Debuggability:** if a correction feels wrong, you can isolate whether it was a bad *label* (Layer 2) or a bad *mapping* (Layer 3).
