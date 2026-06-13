# ml/

Layer 2 of the pipeline — **posture classification**. Data, training, and the on-device export that hands a trained model to firmware.

## Purpose

Train a lightweight classifier (RandomForest baseline, or a small MLP) that labels posture from IMU metrics, then export it to run on the MCU. This is the *only* learned component in Jarada — gating (Layer 1) and haptics (Layer 3) stay deterministic.

## Toolchain

- **Python** (3.10+), `scikit-learn` (RandomForest) and/or a small NN stack (PyTorch/TF) for the MLP.
- Notebooks for exploration (`notebooks/`), scripts for reproducible training (`training/`).
- Setup (once first code lands):
  ```bash
  python -m venv .venv && source .venv/bin/activate
  pip install -r requirements.txt   # added with first training code
  ```

## Layout

| Path | What |
|------|------|
| `data/raw/` | raw captures — **gitignored** (large/private); only `.gitkeep` tracked |
| `data/sample/` | small, committed, reproducible sample for smoke tests |
| `notebooks/` | exploration / EDA |
| `training/` | reproducible training scripts (RF / MLP) — [training/README.md](training/README.md) |
| `models/` | exported artifacts — **binaries gitignored**, dir kept |
| `inference/` | on-device export + the firmware handoff — [inference/README.md](inference/README.md) |

## Input contract

Features come straight from [`../docs/data-contract.md`](../docs/data-contract.md) §5 — the **fixed-order feature vector**:

```
[ neck_pitch, neck_roll, trunk_pitch, trunk_roll, rel_pitch, rel_roll, rel_yaw ]
```

Labels: `forward_head`, `shoulder_asymmetry`, `slouch`, `lateral_tilt`, `neutral`. If the feature vector changes, change the data contract first.

## 🤝 Model-export handoff to firmware

This is the critical seam between `ml/` and `firmware/`. The contract:

1. **Train** → `training/` produces a model + metrics report.
2. **Export** → `inference/` converts it to an MCU-ready form:
   - **MLP:** TFLite Micro (`.tflite`) → C array, **or**
   - **RandomForest:** emit as a quantized C array / generated C (e.g. flattened tree thresholds).
3. **Artifact + manifest** → exported file(s) land in `models/` (gitignored) with a small committed **manifest** recording: schema version (from the data contract), feature order, label order, model type, accuracy/metrics, and a hash.
4. **Embed** → `firmware/` includes the generated C and calls it with the §5 feature vector. Firmware never reshapes features — order/units are fixed by the data contract.
5. **Promotion rule:** a new model only ships if it beats the current baseline on the agreed metric (precision/recall per label). This is where the future **autonomous eval loop** plugs in — see [../docs/roadmap.md](../docs/roadmap.md).

> Keep feature order and label order **identical** in training, the manifest, and firmware. A silent reorder is the classic way this breaks.

## Status

🚧 Stubs only — no training/inference code yet.
