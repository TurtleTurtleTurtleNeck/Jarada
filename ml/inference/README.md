# ml/inference/ — On-device export & firmware handoff (stub)

**Responsibility:** convert a trained Layer 2 model into an MCU-ready artifact and produce the **handoff manifest** firmware needs to embed it. This directory *is* the ml→firmware seam (full contract in [`../README.md`](../README.md#-model-export-handoff-to-firmware)).

## Export paths

| Model type | Export | Embed in firmware as |
|------------|--------|----------------------|
| Small MLP | TFLite Micro (`.tflite`) → C array | included C array + TFLM interpreter |
| RandomForest | quantized C array / generated C (flattened thresholds) | included C, called directly |

## Handoff manifest (committed alongside artifact)

The artifact itself is gitignored (lands in `../models/`); a small manifest **is** committed so firmware/CI can verify what it's embedding:

```json
{
  "schema_version": "v0.1.0",
  "model_type": "random_forest | mlp",
  "feature_order": ["neck_pitch","neck_roll","trunk_pitch","trunk_roll","rel_pitch","rel_roll","rel_yaw"],
  "label_order": ["neutral","forward_head","shoulder_asymmetry","slouch","lateral_tilt"],
  "metrics": { "macro_f1": 0.0, "per_label": {} },
  "artifact": "model_v0.1.0.c",
  "sha256": "<hash>"
}
```

**Planned (stub):**
- `export.py` — model → TFLite/C array + manifest
- `verify.py` — assert feature/label order matches [`../../docs/data-contract.md`](../../docs/data-contract.md) §5 before handoff

**Invariant:** `feature_order` and `label_order` here must equal the data contract and the firmware embed. The future autonomous eval loop gates promotion on `metrics` ([../../docs/roadmap.md](../../docs/roadmap.md)).

🚧 No implementation yet.
