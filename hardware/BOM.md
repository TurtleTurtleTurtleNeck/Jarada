# Bill of Materials (BOM)

> Stub — fill in part numbers, quantities, and sourcing as the build firms up.

| # | Component | Spec | Qty | Notes | Source / PN |
|---|-----------|------|-----|-------|-------------|
| 1 | IMU | BNO085 (9-DOF, onboard fusion) | 2 | cervical + sacrum, I²C | *[fill in]* |
| 2 | MCU (prototype) | ESP32 (BLE built in) | 1 | swap → nRF52840 for production | *[fill in]* |
| 3 | MCU (production) | nRF52840 | 1 | roadmap | *[fill in]* |
| 4 | Haptic actuators | ERM / LRA vibration motors | 9 | directional node array | *[fill in]* |
| 5 | Haptic driver | motor driver / multiplexer | *[TBD]* | drive 9 nodes | *[fill in]* |
| 6 | Battery | LiPo, ~1000 mAh | 1 | target 16–24 h runtime | *[fill in]* |
| 7 | Charging | LiPo charger / USB-C PMIC | 1 | | *[fill in]* |
| 8 | Vest / textile | wearable garment + mounts | 1 | houses IMUs + 9 nodes | *[fill in]* |
| 9 | Wiring / connectors | flex / conductive thread | — | | *[fill in]* |

## Power budget (target)

| Item | Draw | Notes |
|------|------|-------|
| Battery capacity | ~1000 mAh | |
| Target runtime | 16–24 h | drives MCU choice (ESP32 → nRF52840) |
| Avg system draw | *[TBD]* | back-calculate from runtime target |
