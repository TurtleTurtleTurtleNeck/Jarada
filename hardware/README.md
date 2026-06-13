# hardware/

Physical build of the Jarada vest — components, schematics, wiring, and the haptic-node layout.

## Purpose

Document what the vest is made of and how it's wired, so firmware and assembly stay in sync. The **9-node haptic layout** defined here is the physical counterpart to the Layer 3 mapping ([../docs/pipeline.md](../docs/pipeline.md)).

## Contents

| File | Contents |
|------|----------|
| [`BOM.md`](BOM.md) | Bill of materials — parts, quantities, sourcing |
| *(add)* `schematic.*` | wiring schematic (export placeholder) |
| *(add)* `node-layout.*` | 9 haptic-node positions on the vest + IMU mount points |

## Key specs (from project context)

- **IMUs:** 2× BNO085 — mounted at **cervical** (neck) and **sacrum** (base of spine), I²C.
- **MCU:** ESP32 (prototype) → nRF52840 (production).
- **Haptics:** 9 nodes, directional push/pull layout (positions TBD — define `node-layout` here).
- **Power:** ~1000 mAh LiPo, target 16–24 h.

## How it connects to the rest

- IMU mount points + axes must match the sign conventions in [../docs/data-contract.md](../docs/data-contract.md) §1.
- The 9-node layout is referenced by [../firmware/src/haptic/](../firmware/src/haptic/) for the directional mapping.

## Status

🚧 Stub — BOM and layout to be filled in.
