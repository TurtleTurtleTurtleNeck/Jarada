# 자라다 (Jarada)

> AI와 웨어러블을 이용한 자세교정 솔루션, 자라다입니다.
> *A vest-style wearable that catches your slouch before your spine does.*

— by **Team TTNK** 🐢

---

## The 60-second pitch

거북목(forward head posture) and spinal imbalance are now everyday injuries — phones, desks, study. They build silently and you only notice once it hurts. Existing fixes are either passive (a brace you forget you're wearing) or after-the-fact (a clinic visit once the damage is done).

**Jarada is a posture-correction vest you actually feel.** Two IMUs — one at the neck (cervical), one at the base of the spine (sacrum) — continuously read how your spine is oriented in space. When you drift into 거북목, a slouch, or a side-lean, **9 haptic points on the vest nudge you back into alignment** — a directional push/pull you can correct by, not a generic buzz you learn to ignore.

What makes it trustworthy:

- **It knows when *not* to nag.** A deterministic activity gate (Layer 1) tells running from sitting from a stretch, so you don't get corrected mid-jog.
- **It's tuned to *you*.** A Face-ID-style "spinal profile" calibration captures your range of motion and *your* neutral, so "good posture" means yours — not a textbook average.
- **It's honest about what it is.** A non-medical monitoring & feedback device — not a diagnostic tool — by design.

**Target users:** posture clinics and 한방병원 (B2B), where the spinal-profile data and feedback logs give practitioners something to work with between visits.

---

## Architecture at a glance

```
        ┌──────────────┐        ┌──────────────┐
        │ Cervical IMU │        │  Sacrum IMU  │   2× BNO085, quaternion out
        │   (BNO085)   │        │   (BNO085)   │   (onboard sensor fusion)
        └──────┬───────┘        └──────┬───────┘
               └──────────┬────────────┘
                          ▼
                 ┌─────────────────┐
                 │  ESP32 (proto)  │   compute + BLE
                 │ nRF52840 (prod) │
                 └────────┬────────┘
                          ▼
   ┌──────────────────────────────────────────────────┐
   │  Layer 1 — Activity gating   (deterministic)       │  still / walking / running / exception
   │  Layer 2 — Posture classify  (AI: RF / MLP)        │  거북목 · shoulder asym · slouch · tilt
   │  Layer 3 — Haptic mapping     (deterministic LUT)  │  → directional vibration on 9 nodes
   └──────────────────────────────────────────────────┘
                          ▼
                 ┌─────────────────┐
                 │  9 haptic nodes │   push / pull, dead-zone + persistence-gated
                 └─────────────────┘
```

Full write-up: **[docs/architecture.md](docs/architecture.md)**.

---

## The three-layer pipeline

Jarada deliberately sandwiches **one AI layer between two deterministic layers** — the AI only ever *classifies posture*; it never decides when to act or how to buzz. That keeps behavior predictable and debuggable.

| Layer | Job | How | Deterministic? |
|------|-----|-----|----------------|
| **1 — Activity gating** | Decide if we should even be evaluating posture | accel/gyro thresholds → still / walking / running / exception | ✅ yes |
| **2 — Posture classification** | Label the current posture | RandomForest or small MLP over absolute + relative IMU metrics | 🤖 AI |
| **3 — Haptic mapping** | Turn a posture label into a correction | lookup table → directional node vibration, dead-zone + N-sec persistence gate | ✅ yes |

Details: **[docs/pipeline.md](docs/pipeline.md)**.

---

## Repository map

| Path | What lives here |
|------|-----------------|
| [`docs/`](docs/) | Architecture, pipeline, the shared **data contract**, spinal-profile spec, references, roadmap |
| [`firmware/`](firmware/) | ESP32/nRF52840 firmware — IMU, calibration, Layer 1 gating, haptics, BLE |
| [`ml/`](ml/) | Layer 2 — data, training (RF/MLP), and the **on-device model export → firmware handoff** |
| [`app/`](app/) | Companion app — **design-only for now** (Figma exports + screen specs) |
| [`hardware/`](hardware/) | BOM, schematics, wiring notes |

**One source of truth to know about:** [`docs/data-contract.md`](docs/data-contract.md) defines the IMU metric schema (field names + units) that firmware, ml, and app all speak. Change it there, nowhere else.

---

## Status

🚧 **v1 scaffold.** Structure, documentation skeletons, and stubs — no implementation yet. See [`docs/roadmap.md`](docs/roadmap.md) for what's next (incl. the v2 visuomotor loop and the autonomous eval loop).

---

## Setup

> Nothing to build yet — this is a documentation/structure scaffold. The commands below are the intended entry points as each component lands.

```bash
# clone
git clone https://github.com/TurtleTurtleTurtleNeck/Jarada.git
cd Jarada

# firmware (ESP32, PlatformIO) — see firmware/README.md
#   cd firmware && pio run

# ml (Python) — see ml/README.md
#   cd ml && python -m venv .venv && source .venv/bin/activate
#   pip install -r requirements.txt   # (added with first training code)
```

**Contributing:** `main` is protected — all changes go through a pull request with one approving review. Branch off `main`, open a PR.

## License

All rights reserved (no open-source license). © Team TTNK.
