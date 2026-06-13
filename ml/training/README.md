# ml/training/ — Layer 2 model training (stub)

**Responsibility:** reproducible training of the posture classifier — **RandomForest** (baseline) or a **small MLP**.

**Inputs:** feature vector + labels per [`../../docs/data-contract.md`](../../docs/data-contract.md) §5, built from `data/`.

**Outputs:** a trained model + a metrics report (per-label precision/recall, confusion matrix), handed to [`../inference/`](../inference/) for export. Artifacts go to `../models/` (gitignored).

**Planned (stub):**
- `train.py` — load features/labels, fit RF or MLP, emit model + metrics
- `evaluate.py` — held-out scoring, confusion matrix (esp. `slouch` vs `forward_head`)
- config for feature order / label order (must match the data contract)

**Keep fixed:** feature order and label order must match the data contract and the firmware embed exactly.

🚧 No implementation yet.
