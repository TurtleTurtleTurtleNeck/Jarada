# app/ — Companion app

> **Design-only for now.** v1 of the app is built in **Figma**; this directory holds design exports and screen specs, **not** code. Implementation comes later (see [../docs/roadmap.md](../docs/roadmap.md)).

## Purpose

The companion app is how a wearer onboards, runs **spinal-profile calibration**, and reviews posture history. For B2B (posture clinics / 한방병원) it's also where practitioners view a wearer's profile and trends.

## What lives here

| Path | Contents |
|------|----------|
| `design/` | Figma exports (PNG/SVG screen specs), flows, component notes |

> Heavy raw `.fig` source files are gitignored — commit exported specs/images, not binaries. Keep a link to the live Figma file below.

**Figma file:** *[add link]*

## Key screens (to spec in Figma)

- Onboarding / fit check
- **Spinal-profile registration** — the Face-ID-style flow ([../docs/spinal-profile.md](../docs/spinal-profile.md))
- Live posture / feedback view
- History & trends (the B2B data view)

## How it connects to the rest

- Talks to the vest over **BLE** using the frame in [../docs/data-contract.md](../docs/data-contract.md) §6.
- Captures the spinal profile that firmware later **applies** ([../firmware/src/calibration/](../firmware/src/calibration/)).
- All metric names/units in mockups must match the [data contract](../docs/data-contract.md) — one vocabulary across design, firmware, and ml.

## Status

🎨 Design-only. No app code in v1.
