# src/imu/ — BNO085 driver & metric derivation (stub)

**Responsibility:** read both BNO085 IMUs (`cervical`, `sacrum`) over I²C, get fused quaternions, convert to Euler, and derive the absolute + relative metrics.

**Produces:** the fields in [`../../../docs/data-contract.md`](../../../docs/data-contract.md) §2–§4 (raw stream, per-IMU Euler, relative cervical−sacrum).

**Planned interface (stub):**
- `imu_init()` — bring up both sensors, configure rotation-vector + accel/gyro reports
- `imu_read(ImuFrame* out)` — latest quaternion, accel/gyro magnitude, cal status per IMU
- `imu_metrics(const ImuFrame*, Metrics* out)` — quaternion → Euler → absolute + relative metrics

**Notes:** units/sign conventions are defined by the data contract — don't redefine them here. Two sensors share the bus; addressing TBD.

🚧 No implementation yet.
