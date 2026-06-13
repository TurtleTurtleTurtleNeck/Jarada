# src/ble/ — BLE comms to companion app (stub)

**Responsibility:** Bluetooth Low Energy link between vest and app — stream live metrics/posture, receive the spinal profile, and emit feedback events.

**Frame schema:** [`../../../docs/data-contract.md`](../../../docs/data-contract.md) §6. On the wire this will likely be packed binary; the JSON in the data contract is the **logical** contract — keep field names identical.

**Planned interface (stub):**
- `ble_init()` — advertise, set up GATT services/characteristics
- `ble_stream(const Frame*)` — push a metrics/posture frame to the app
- `ble_on_profile(SpinalProfile* out)` — receive a spinal profile from the app
- characteristics TBD (telemetry, profile write, events)

🚧 No implementation yet.
