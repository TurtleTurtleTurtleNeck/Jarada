# Roadmap

> v1 is the three-layer pipeline as built (see [pipeline.md](pipeline.md)). This file captures **post-v1 direction** — clearly separated so nothing here is mistaken for already-implemented behavior.

## v1 (current scope)

- ✅ Three-layer pipeline: deterministic gating → AI classification → deterministic haptics
- ✅ Two-IMU absolute + relative metrics ([data-contract.md](data-contract.md))
- ✅ Per-wearer spinal-profile calibration ([spinal-profile.md](spinal-profile.md))
- ✅ ESP32 prototype + companion app design

## Production hardening

- nRF52840 migration (power, BLE efficiency) — see [architecture.md](architecture.md)
- Hit the **16–24 h** battery target on real workloads
- Multi-label posture (e.g. `forward_head` + `lateral_tilt` simultaneously)
- App build-out (currently design-only)

## v2 — Visuomotor loop *(future / exploratory)*

Close the loop from *correction* to *learning*. Today Layer 3 fires a directional cue; v2 would track **whether the wearer actually responded** and adapt over time:

- Treat the haptic cue → posture-change as a control loop: did the nudge produce the intended correction within a window? Use the response (or non-response) to tune cue timing, intensity, and the persistence gate per wearer.
- Longer-term: richer sensing (e.g. visual/relative-position input) to disambiguate postures the two IMUs can't separate.
- Goal: corrections that *teach* posture, not just interrupt it — measurable reduction in time-in-bad-posture over weeks.

> Scope, sensing, and feasibility are open. Listed as direction, not commitment.

## Autonomous eval loop *(future / dev-process)*

A self-improving evaluation pipeline for Layer 2, so the model gets better without manual babysitting:

- Continuously score the classifier against labeled/held-out and field data (precision/recall per posture label, confusion between e.g. `slouch` vs `forward_head`).
- Surface failure clusters, propose retraining/data-collection targets, and gate model promotion on a metric bar before a new model is exported to firmware.
- Tie into the **ml→firmware export handoff** ([../ml/README.md](../ml/README.md)) so only models that beat the current baseline ship.

> Inspired by closed-loop eval/improve harnesses; adapt to Jarada's posture-classification metrics.
