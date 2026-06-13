# firmware/

On-device firmware for the Jarada vest: reads the two IMUs, runs the full three-layer pipeline, and drives the 9 haptic nodes.

## Purpose

Everything that runs on the wearable itself — sensing, metric derivation, Layer 1 gating, Layer 2 inference (embedded model), Layer 3 haptics, and BLE comms to the app.

## Toolchain

- **Framework:** [PlatformIO](https://platformio.org/) (Arduino/ESP-IDF) — see [`platformio.ini`](platformio.ini)
- **MCU:** ESP32 (prototype, BLE built in) → **nRF52840** (production roadmap)
- **Sensors:** 2× BNO085 over I²C
- **Build / flash:** `pio run` / `pio run -t upload` · monitor: `pio device monitor`

## Module map (`src/`)

| Module | Responsibility | Connects to |
|--------|----------------|-------------|
| [`imu/`](src/imu/) | BNO085 driver, quaternion read, → Euler/metrics | produces the [data contract](../docs/data-contract.md) |
| [`calibration/`](src/calibration/) | Apply the stored spinal profile (neutral/ROM/baselines) | [spinal-profile.md](../docs/spinal-profile.md) |
| [`layer1_gating/`](src/layer1_gating/) | Deterministic activity gate (still/walk/run/exception) | [pipeline.md](../docs/pipeline.md) Layer 1 |
| [`haptic/`](src/haptic/) | 9-node driver + directional mapping (Layer 3) | [pipeline.md](../docs/pipeline.md) Layer 3, [hardware](../hardware/README.md) |
| [`ble/`](src/ble/) | Comms to companion app | [data-contract.md](../docs/data-contract.md) §6 |

## How it connects to the rest

- **Speaks the [data contract](../docs/data-contract.md)** — field names/units for metrics and the BLE frame are defined there, not here.
- **Consumes the Layer 2 model from `ml/`.** The trained classifier is exported (TFLite Micro or a quantized C array) and embedded here — see the **model-export handoff** in [`../ml/README.md`](../ml/README.md). Firmware calls the embedded model with the fixed feature vector from [data-contract.md](../docs/data-contract.md) §5.
- **Applies the spinal profile** captured by the app to personalize metrics and thresholds.

## Status

🚧 Stubs only — no implementation yet. Each `src/` module has a README describing its intended interface.
