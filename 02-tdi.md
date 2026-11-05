# Time-dependent inhibition — ADME-Fusion

> [!NOTE]
> **TL;DR — two endpoint-specific classifiers**
>
> **Molecular structure → CYP2D6 and CYP3A4 TDI calls for 750 compounds**
>
> - **CYP2D6:** a 100-tree random forest on Morgan fingerprints and 12 descriptors; positive at **p ≥ 0.25**.
> - **CYP3A4:** five fitted ensembles, each combining **15% LightGBM + 45% ExtraTrees + 20% graph networks + 20% ADME-encoder readouts**.
> - **ADME branch:** two CheMeleon encoders adapted to published property predictions, then frozen; each contributes **10%** through a logistic classifier.
> - **Final CYP3A4 call:** positive when **at least three of five ensembles** exceed their own training-selected cutoffs.

## Training data

Challenge supervision comes from the structures and Boolean TDI labels in the [official release](https://huggingface.co/datasets/openadmet/cyp-challenge-train-test/tree/d7eb3c973dca01aad8e33c91c29e5123fd925d8f). The endpoints use different training populations and fitting procedures:

| Endpoint | Labeled compounds | Fitting procedure |
|---|---:|---|
| CYP2D6 | 1,214, including 255 positives | Three grouped folds select forest leaf size and cutoff within a 3,859-compound TDI pool; the final forest uses all 1,214 endpoint labels |
| CYP3A4 | 2,748 within a 4,834-compound development pool | Five outer chemical-group folds, each with three inner folds for cutoff selection; the five outer-training-fold ensembles are retained for inference |

Reserved confirmation compounds are excluded from supervised fitting. Missing endpoint labels are omitted, not assigned to the negative class. Test compounds contribute structures and identifiers only. Imputation, scaling and classifier fitting use the applicable training fold.

## CYP2D6: fingerprint forest

The forest receives a radius-2, 2,048-bit **chiral Morgan fingerprint** and 12 descriptors covering charge, nitrogen/amine content, aromaticity, hydrogen bonding, lipophilicity, size, polar surface area and rotatable bonds. The pipeline applies median imputation, missingness indicators and StandardScaler before the forest.

The final model has **100 trees, minimum leaf size 5 and no class weighting**. A positive call requires **p ≥ 0.25**; no probability calibration is applied.

## CYP3A4: trees, molecular graphs and ADME encoders

Each of the five ensembles has the same composition. The four ExtraTrees models and four graph networks differ by random seed; each group is averaged before blending.

| Component | Final weight | Inputs |
|---|---:|---|
| LightGBM | 15% | 2,048 achiral Morgan bits and 217 RDKit descriptors |
| Four ExtraTrees models | 45% | Radius-2 Morgan counts and feature-Morgan counts, MACCS keys, 42 MQNs and 217 RDKit descriptors |
| Four graph networks | 20% | Atom–bond graph, RDKit descriptors and MQNs |
| Four-CYP encoder + logistic classifier | 10% | Frozen 2,048-dimensional CheMeleon embedding, plus an indicator for missing embeddings |
| ADME25 encoder + logistic classifier | 10% | Frozen 2,048-dimensional CheMeleon embedding, plus an indicator for missing embeddings |

**Graph networks.** Atom features encode element, degree, charge, hydrogen count, aromaticity, ring membership, hybridization and chirality; bond features encode bond type, conjugation, ring membership and stereochemistry. Two residual message-passing steps use 48 channels. Mean and square-root-normalized sum pooling, an atom-count feature and a separate descriptor branch feed the binary classifier. These networks are trained from scratch on CYP3A4 TDI labels and use 2D connectivity, without conformers or docking.

**ADME encoders.** Both start from the [public CheMeleon checkpoint](https://zenodo.org/records/15460715) and undergo additional training on the same **2,048 external molecules**. Supervision consists of **published Novartis model predictions**, not experimental measurements:

| Encoder | Adaptation targets |
|---|---|
| Four-CYP | Predicted log(kobs) and CYP3A4, CYP2C9 and CYP2D6 pIC50 |
| ADME25 | The same four targets plus clearance, fraction-unbound, lipophilicity, permeability and efflux properties: 25 targets in total |

Adaptation updates the encoder and a property-prediction head for **256 steps, equivalent to four passes over the source sample**. Source-standardized SmoothL1 losses are averaged within property families, then across families. Both encoders use identical source ordering, head initialization and training budgets. Chemical families containing official activity compounds are excluded from the external sample.

For TDI prediction, the property heads are discarded. Mean-pooled embeddings receive per-molecule layer normalization; each frozen encoder then feeds a fold-specific standardized logistic classifier. **Only the logistic classifiers learn from challenge labels.** The two encoder checkpoints are shared across all five ensembles.

**Combining the predictions.** Component probabilities are combined with the fixed weights above; weights are not fitted. The five ensembles apply cutoffs **0.30, 0.25, 0.40, 0.35 and 0.30**, selected from their respective inner-fold predictions. At least three positive votes produce a positive CYP3A4 call. No full-population refit replaces these five ensembles.

<details>
<summary><strong>Training settings</strong></summary>

- **CYP2D6 forest:** bootstrap sampling, square-root feature sampling, seed 20261103. Leaf size is selected from 2, 5 and 10 by pooled grouped out-of-fold average precision; ties favor the larger leaf.
- **LightGBM:** 400 trees, depth 4, 24 leaves, learning rate 0.03; training-only median imputation.
- **ExtraTrees:** 320 trees per model, minimum leaf size 4, feature fraction 0.35 and balanced class weights. Both count fingerprints have 2,048 dimensions and omit chirality.
- **Graph training:** 40 fixed epochs, batch size 128, unweighted binary cross-entropy; AdamW with initial learning rate 0.0015, cosine decay, weight decay 0.02 and dropout 0.2. Continuous descriptor inputs use training-only imputation/scaling and clipping to ±10. Graph and ExtraTrees ensembles use seeds 20261103–20261106.
- **ADME adaptation:** batch size 32; AdamW encoder/head learning rates 1e-5/1e-3, weight decay 1e-4, seed 20261103. A 2,048→256→25 ReLU property head is used for both encoders; the Four-CYP loss masks the other 21 targets.
- **ADME readouts:** StandardScaler followed by logistic regression, `C=0.01`, `liblinear`, `max_iter=1000`, seed 20261104. Embeddings have 2,048 dimensions plus one indicator: 1 when an embedding is available, 0 when missing. Layer normalization has no learned affine parameters.
- **Cutoffs:** maximize training out-of-fold MCC over 0.05–0.95 in 0.05 steps; ties favor proximity to 0.5, then the lower cutoff. CYP3A4 uses inner-fold predictions for each outer-fold ensemble.

</details>

## Development evaluation

The following comparison isolates the ADME branches on **five fixed chemical-group folds for CYP3A4**. Scores are equal-weight fold means.

The **tree/graph control** renormalizes LightGBM, ExtraTrees and graph weights to 18.75%, 56.25% and 25%. Each single-encoder variant uses 80% of that control and 20% of its encoder/readout. The original CheMeleon control uses the public checkpoint without ADME adaptation; ADME-Fusion uses the full 80/10/10 blend.

| CYP3A4 system | MCC ↑ | Average precision ↑ |
|---|---:|---:|
| Tree/graph control | 0.375454 | 0.509207 |
| Control + original CheMeleon | 0.380879 | 0.518196 |
| Control + Four-CYP | 0.377467 | 0.519425 |
| Control + ADME25 | 0.376064 | 0.519138 |
| **ADME-Fusion** | **0.378488** | **0.518979** |

ADME-Fusion improves mean MCC by **0.003034**, with gains in three of five folds. The conditional 95% interval for the difference is **[−0.016860, +0.023858]**, from 1,000 paired chemical-group resamples.

Each held-out compound is evaluated by the ensemble that excluded its fold. Consequently, these scores assess **individual fold ensembles, not the final five-ensemble majority vote**; they also do not evaluate CYP2D6. Development data were adaptively reused, and the intervals hold fitted models fixed, excluding retraining and search uncertainty.

> **Preparation note.** This submission was assembled under a tight time limit using completed model fits and training-selected cutoffs. Additional model fitting and threshold refinement were not performed before submission, and separate validation of the final majority-vote ensemble remained unfinished.

## Coverage and limitations

All **750 compounds** receive both binary calls: 85 positive for CYP2D6 and 324 for CYP3A4. The CYP2D6 imputer handles 13 entirely missing feature rows. Each ADME encoder supplies embeddings for **737/750 compounds**. The remaining 13 compounds have unsupported enhanced stereochemistry; their embeddings are filled with zeros and their indicator is set to 0. Their logistic branches still contribute to the blend.

External supervision consists of publicly released predictions from models with private training ancestry. No proprietary dataset is used directly, but overlap with the original CheMeleon or Novartis teacher training sets remains unresolved. Code, fitted models and provenance records remain private.

---

**Accepted:** 3 November 2026, 23:58:38 UTC.

**SHA-256 (`predictions.csv`):** `b8d7f8c20106ae065dc120e026379185cb26453fb0fa2356d1b1f3886021fd9f`.
