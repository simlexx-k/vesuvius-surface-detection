# Vesuvius Challenge — Surface Detection

Build a model to virtually unwrap ancient scrolls by segmenting the papyrus surface in 3D CT scans from the Villa dei Papiri.

## Competition overview
Ancient Herculaneum scrolls are too fragile to physically open. The goal is to trace the scroll surface through folds, gaps, and distortions so later stages of the virtual unwrapping pipeline can recover text. The tightest and most tangled regions are where segmentation quality matters most.

## Task
Given 3D CT scan chunks, predict the papyrus surface as a 3D binary mask.

Key guidance:
- The ideal target is the **recto** surface (the side facing the umbilicus/center of the scroll), which lies on the layer with horizontal fibers.
- It is acceptable if the segmentation approximates a sheet even if it encompasses both recto and verso.
- Avoid topological mistakes: artificial mergers between different sheets and holes that split one sheet into multiple disconnected components.

## Data
- 3D chunks of binary-labeled CT scans of closed, carbonized Herculaneum scrolls.
- Chunk dimensions are not fixed and can vary across the dataset.
- Data acquired at the ESRF synchrotron (BM18) and the DLS synchrotron (I12).

The host team may release **additional labeled data** during the competition. These new labels are expected to be less curated than the original training set; use discretion when incorporating them.

## Evaluation
Submissions are scored using a **weighted average** of three segmentation metrics:
- Surface Dice
- TopoScore
- VOI

Scoring can take several hours to complete.

### Metric summary (topology-aware 3D surface segmentation)
We reward surface proximity, instance consistency, and topological correctness so good unwrapping is not only accurate but **topologically right**.

Leaderboard formula (higher is better):
```
Score = 0.30 × TopoScore + 0.35 × SurfaceDice@τ + 0.35 × VOI_score
```

Key defaults and edge cases:
- SurfaceDice tolerance: `τ = 2.0` (in physical spacing units for each case).
- All terms are in `[0, 1]`.
- Empty-mask convention: both empty → all metrics = 1.0; one empty → SurfaceDice = 0, TopoScore = 0 on k=0, VOI_score near 0.

**SurfaceDice@τ** — Are the two surfaces within tolerance?
- Measures the fraction of surface points within distance `τ` between prediction and ground truth (both directions).
- Tolerant to small boundary offsets while still enforcing correct geometry.

**VOI_score** — Do instances split/merge correctly?
- Variation of Information on connected components (default 26-connectivity).
- `VOI_total = VOI_split + VOI_merge` (lower is better).
- Converted to a bounded score: `VOI_score = 1 / (1 + 0.3 × VOI_total)`.

**TopoScore** — Are topological features preserved?
- Compares Betti features across k=0 (components), k=1 (tunnels/handles), k=2 (cavities).
- Weighted average over active dimensions (default weights `w0=0.34`, `w1=0.33`, `w2=0.33`).

Practical metric tips:
- Favor **continuity** within a wrap; avoid **bridges** across adjacent wraps.
- Watch for spurious holes/handles; TopoScore penalizes them even if Dice stays high.

## Submission format
Submit a **zip** containing one `.tif` volume mask per test image:
- Each mask must be named `[image_id].tif`.
- Dimensions must **exactly match** the source image.
- Data type must match the train mask.
- The zip must be named `submission.zip`.

## Code requirements (Kaggle)
Submissions must be made via Kaggle **Notebooks** with:
- Runtime ≤ 9 hours (CPU or GPU).
- Internet access **disabled**.
- External data must be freely and publicly available.
- The submission file must be named `submission.zip`.

## References
- The model outputs are designed to plug directly into the Vesuvius Challenge digital unwrapping pipeline.
