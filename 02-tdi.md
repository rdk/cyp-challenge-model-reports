# Time-dependent inhibition — ADME-Fusion

> [!NOTE]
> **TL;DR**
>
> **Molecular structure → binary time-dependent inhibition calls**
>
> - **Targets:** CYP2D6 and CYP3A4 for **750 compounds**.
> - **CYP2D6:** fingerprint/descriptor random forest; positive when **p ≥ 0.25**.
> - **CYP3A4:** tree, graph-network and ADME-encoder probability blends; **majority vote across five fitted systems**.
> - **CYP3A4 weights:** 15% LightGBM, 45% four-model ExtraTrees average, 20% four-model graph-network average, and 10% from each of two frozen ADME encoders with logistic readouts.

## Inputs and training data

Both endpoints use structures and Boolean labels from the [official challenge release](https://huggingface.co/datasets/openadmet/cyp-challenge-train-test/tree/d7eb3c973dca01aad8e33c91c29e5123fd925d8f). Their training populations differ:

| Endpoint | Available labels | Training/evaluation design |
|---|---:|---|
| CYP2D6 | 1,214, including 255 positives | Final forest fitted on all available endpoint labels; three grouped folds select leaf size and cutoff within a 3,859-compound TDI pool |
| CYP3A4 | 2,748 within a 4,834-compound development pool | Five outer chemical-group folds; three inner folds select cutoffs |

Reserved confirmation compounds are excluded from supervised fitting, and missing responses remain unknown. Test data provide only structures and identifiers.

Two CYP3A4 encoders start from the [public CheMeleon checkpoint](https://zenodo.org/records/15460715). ADME (absorption, distribution, metabolism and excretion) pretraining uses 2,048 screened external molecules with **published Novartis model-generated targets**, not experimental labels:

| Encoder | Pretraining targets |
|---|---|
| Four-CYP | Predicted log(kobs) and CYP3A4, CYP2C9 and CYP2D6 pIC50 |
| ADME25 | All 25 published ADME targets |

Chemical families containing official activity compounds are excluded from this external pretraining set. The original encoder and private teacher training memberships remain unresolved. No proprietary dataset is used directly; external supervision consists of published model outputs.

## CYP2D6: random forest

The input is a radius-2, 2,048-bit chiral Morgan fingerprint plus 12 compact descriptors covering charge, nitrogen/amine content, aromaticity, hydrogen bonding, lipophilicity, size, polar surface area and rotatable bonds. Training-only median imputation, missingness indicators and StandardScaler precede a **100-tree random forest**, with square-root feature sampling, bootstrap sampling, minimum leaf size **5**, seed 20261103 and no class weighting.

Leaf size is selected from 2, 5 and 10 by pooled grouped out-of-fold average precision, breaking ties toward the larger leaf. The probability cutoff maximizes out-of-fold MCC on a 0.05–0.95 grid; ties favor proximity to 0.5, then the lower value. The deployed rule is **positive if p ≥ 0.25**. There is no probability calibration.

## CYP3A4: probability blend and majority vote

Each of five outer-fold systems uses the following **final probability weights**:

| Component | Weight | Representation/model |
|---|---:|---|
| LightGBM | 15% | Morgan bits and RDKit descriptors |
| Four ExtraTrees models, averaged equally | 45% | Morgan counts, feature-Morgan counts, MACCS, MQN and RDKit descriptors |
| Four graph networks, averaged equally | 20% | Molecular graph with a descriptor branch |
| Four-CYP encoder + logistic readout | 10% | Frozen ADME-pretrained CheMeleon embedding |
| ADME25 encoder + logistic readout | 10% | Frozen ADME-pretrained CheMeleon embedding |

The ADME encoders remain frozen during challenge-label fitting; only their logistic readouts are trained. Feature scaling uses training folds only. Unsupported enhanced-stereo embeddings use zero vectors and an availability flag.

The five systems apply cutoffs **0.30, 0.25, 0.40, 0.35 and 0.30**, each selected using its own inner-fold predictions. CYP3A4 is positive when **at least three systems vote positive**. These are outer-training-fold models; no full-population refit enters this vote.

<details>
<summary><strong>Technical settings</strong> — CYP3A4</summary>

- **LightGBM:** 400 trees, depth 4, 24 leaves, learning rate 0.03; 2,048 achiral Morgan bits plus 217 RDKit descriptors, with training-only median imputation.
- **ExtraTrees:** 320 trees per model, minimum leaf size 4 and feature fraction 0.35.
- **Graph networks:** two message-passing steps, 48 channels, mean/normalized-sum pooling and a descriptor branch. Graph and ExtraTrees ensembles use seeds 20261103–20261106.
- **ADME pretraining:** 256 AdamW steps; batch size 32; encoder/head learning rates 1e-5/1e-3. Readouts use 2,048-dimensional mean-pooled embeddings with per-molecule layer normalization and an availability indicator, StandardScaler, and logistic regression (`C=0.01`, `liblinear`, `max_iter=1000`, seed 20261104).
- **Threshold selection:** maximum inner out-of-fold MCC on 0.05–0.95 in 0.05 steps, ties nearest 0.5 and then lower. Blend weights are fixed.

</details>

## Development evaluation

Matthews correlation coefficient (**MCC**) evaluates binary calls; **average precision** evaluates probability ranking. Higher is better for both. The CYP3A4 comparison uses equal-weight means over five outer folds.

The **tree/graph control** contains only LightGBM, ExtraTrees and graph networks, renormalized to 18.75%, 56.25% and 25%. Each single-encoder arm uses 80% of this control and 20% of the corresponding encoder/readout. “Original CheMeleon” means the public encoder without additional ADME pretraining.

| CYP3A4 system | MCC ↑ | Average precision ↑ |
|---|---:|---:|
| Tree/graph control | 0.375454 | 0.509207 |
| Control + original CheMeleon | 0.380879 | 0.518196 |
| Control + Four-CYP | 0.377467 | 0.519425 |
| Control + ADME25 | 0.376064 | 0.519138 |
| **Full ADME-Fusion recipe** | **0.378488** | **0.518979** |

The full recipe gains 0.003034 MCC, improving three of five folds. Its conditional 95% interval is [−0.016860, +0.023858], from 1,000 paired chemical-group resamples. This does not meet the prespecified 0.005 minimum gain. The original-encoder control has the higher point estimate; added value from ADME pretraining remains unconfirmed.

These scores evaluate individual held-out folds, **not the final five-system majority vote or the two-endpoint model as a whole**. Adaptive development-set reuse and fixed-prediction intervals omit search and retraining uncertainty. No independent confirmation is claimed.

## Inference and availability

All 750 compounds receive binary calls. The CYP2D6 imputer handles 13 entirely missing feature rows. Output positive counts are 85 for CYP2D6 and 324 for CYP3A4; these are model call rates, not measured prevalence.

Code, models and provenance remain private. No official score is reported here.

---

**Accepted:** 3 November 2026, 23:58:38 UTC.

**SHA-256 (`predictions.csv`):** `b8d7f8c20106ae065dc120e026379185cb26453fb0fa2356d1b1f3886021fd9f`.
