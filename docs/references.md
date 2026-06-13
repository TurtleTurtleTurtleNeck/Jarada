# References

> Five research references grounding Jarada's design. Each entry: **title · authors · year · venue**, then a one-line *why it matters to this design*. Bibliographic details are placeholders — fill in `[…]`.

---

### 1. Forward head posture (거북목) — prevalence & musculoskeletal impact
- **Title:** *[fill in]*
- **Authors:** *[fill in]*
- **Year:** *[fill in]*
- **Venue:** *[fill in]*
- **Link / DOI:** *[fill in]*
- **Why it matters:** Establishes the problem — that forward head posture is common and causes measurable musculoskeletal harm — justifying a continuous, preventive intervention.

---

### 2. IMU-based posture monitoring wearables — validity
- **Title:** *[fill in]*
- **Authors:** *[fill in]*
- **Year:** *[fill in]*
- **Venue:** *[fill in]*
- **Link / DOI:** *[fill in]*
- **Why it matters:** Evidence that body-worn IMUs can track posture accurately enough to be useful — validates the core sensing approach vs cameras or pressure mats.

---

### 3. Sensor fusion for body-segment orientation (quaternion / BNO085-class)
- **Title:** *[fill in]*
- **Authors:** *[fill in]*
- **Year:** *[fill in]*
- **Venue:** *[fill in]*
- **Link / DOI:** *[fill in]*
- **Why it matters:** Supports relying on onboard quaternion fusion for stable orientation — justifies the BNO085 choice and the absolute+relative two-IMU metric design.

---

### 4. Haptic biofeedback for posture correction — efficacy
- **Title:** *[fill in]*
- **Authors:** *[fill in]*
- **Year:** *[fill in]*
- **Venue:** *[fill in]*
- **Link / DOI:** *[fill in]*
- **Why it matters:** Shows real-time haptic cues actually change posture behavior — justifies Layer 3 (directional vibration) over passive bracing or visual-only feedback.

---

### 5. ML classification of posture/activity from IMU data (RandomForest / MLP)
- **Title:** *[fill in]*
- **Authors:** *[fill in]*
- **Year:** *[fill in]*
- **Venue:** *[fill in]*
- **Link / DOI:** *[fill in]*
- **Why it matters:** Precedent for classifying sitting/posture states from IMU features with lightweight models — supports the Layer 2 model choice and on-device feasibility.
