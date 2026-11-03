# Direct inhibition: interval-L1 ensemble with count/graph, pooled Snakes and ShEPhERD2

**Submission:** `rdk-CYP-Inhibition-002` · **Track:** direct inhibition

The portal acknowledged receipt for processing on 3 November 2026 at 23:56:16 UTC. No official score is reported here.

## Summary

This candidate predicts CYP1A2, CYP2C9, CYP2D6 and CYP3A4 direct-inhibition pIC50. It combines 75% of the frozen interval-L1 ensemble, 18.75% of a count/graph/pooled-Snakes ridge readout, and 6.25% of a ShEPhERD2 ridge readout. Its reused five-fold development mean ST-RAE (lower is better) is 0.700456 versus 0.708890 for the parent ensemble. This is exploratory development evidence, not a blinded score or a passed promotion gate.

## Data and exclusions

Final supervised fitting uses exactly 4,834 admitted official compounds, with 5,230 observed compound-endpoint values: 1,129 CYP1A2, 1,056 CYP2C9, 1,209 CYP2D6 and 1,836 CYP3A4. Reserved compounds and 107 additional related compounds remain excluded. No protected or blinded labels are used. Only raw structures and IDs are read for the 750 blinded compounds; no encoder adaptation or preprocessing fit uses blind structures. Missing labels remain masked. The inherited parent uses private ADME-teacher ancestry; proprietary-data use is declared. Pretraining overlap for inherited encoders is unresolved.

## Method

The frozen parent combines fingerprint/descriptor random forests, absolute-error histogram boosting, three descriptor-adapted CheMeleon networks and an interval-L1 CheMeleon network. For CYP1A2, CYP2C9 and CYP3A4, its internal weights are 28.125% RF, 28.125% mean descriptor-adapted neural prediction, 9.375% Mordred HGB, 9.375% Morgan/RDKit HGB and 25% interval-L1 neural prediction. CYP2D6 uses 75% RF and 25% interval-L1. Parent predictions are reused byte-for-byte from the previously fitted ensemble; those fits used the same admitted population. Descriptor adaptation predicts training-only molecular descriptors before supervised readout fitting and fine-tuning. The separate interval-L1 model minimizes absolute distance outside the supplied response confidence interval, with zero interval loss inside it.

The new graph branch contains 2,048 radius-2 chiral Morgan count features, 100 graph descriptors and 233 pooled Snakes descriptors. Graph descriptors cover rotors, rotor networks, ring constraints, topological flexibility and symmetry/state proxies. They are computed proxies, not measured conformational or binding entropy. Snakes encode atom-pair and shortest-path attributes as molecular graph tokens. They use long-diverse token selection, at most 32 tokens, seed 20261103, `central_atoms=1`, `short_path_fraction=0.25` and no graph-proxy token addition. Mean, population standard deviation, maximum and sum of 58 token columns plus token count yield 233 features. These are fixed pooled descriptors, not a learned token model; nominal-code pooling has physical interpretation limits.

The ShEPhERD2 branch uses the frozen 2,304-dimensional append readout with xTB charges and sigma 0.1. One conformer is attempted per structure using deterministic ETKDGv3 generation and up to 500 UFF optimization iterations, with base seed 17 and surface seed 20261102. Enhanced stereo, multicomponent conformer rejection and native failures retain explicit missingness. No alternative charge model or substitute embedding is used.

Each new branch fits four endpoint-specific ridge readouts: alpha 10, LSQR tolerance 0.0001, maximum 2,000 iterations. Counts use log1p and training-only median imputation with missingness indicators. Continuous features use training-only median imputation, missingness indicators and standardization. Empty features are retained. Weights and hyperparameters were fixed for the development comparison below.

## Validation and selection

Five fixed inferred-core folds contain 4,834 compounds. Each endpoint uses observed training labels only; held-out compounds remain excluded from that fold's preprocessing and fit. The score is the equal mean across five folds and four ST-RAE endpoints. Results are repeatedly reused development evidence.

| Endpoint | Parent | New candidate |
|---|---:|---:|
| CYP1A2 | 0.801068 | 0.809805 |
| CYP2C9 | 0.634501 | 0.618193 |
| CYP2D6 | 0.883095 | 0.879384 |
| CYP3A4 | 0.516897 | 0.494444 |
| Macro | 0.708890 | 0.700456 |

The macro gain is 0.008434, positive in four of five folds; the conditional 95% interval is [0.001241, 0.014972], and the 15-contrast multiplicity-adjusted interval is [−0.002740, +0.018115]. The model without Snakes scores 0.703301; incremental Snakes gain of 0.002844 has adjusted interval [−0.002172, +0.007788]. CYP1A2 worsens by 0.008737. Intervals resample whole allocation groups within folds, conditional on fixed predictions, and omit retraining and historical search uncertainty. No independent confirmation or formal promotion is claimed. This variant was submitted as an exploratory contest choice despite the inconclusive adjusted comparison.

## Final prediction procedure

Eight auxiliary models are fitted using all admitted endpoint-observed training rows. All 750 blind IDs are retained. Genuine extraction failures use each fitted model's training-only imputation policy. The prediction is `0.75 × parent + 0.1875 × count_graph_snakes + 0.0625 × ShEPhERD2`, with no clipping, offset, noise or blind-feedback fitting. CSV preserves original publisher SMILES and Molecule_Name and contains only these identifiers plus four pIC50 columns. Model-native separate-process replay, keyed mixture reconstruction, CSV/Parquet agreement, finite values, completeness and local official-reference validation passed before packaging.

## Reproducibility

Frozen inputs, fold memberships, source and checkpoint hashes, held-out predictions, model states and native feature outputs are retained privately. Final inference was replayed in a separate process, with maximum absolute difference 8.88 × 10⁻¹⁶. Count/graph generation used RDKit 2025.09.6; pooled Snakes and conformers used the qualified Python 3.14 runtime. This verifies saved-state inference, not identical stochastic retraining.

## Limitations and disclosures

Code remains private. Proprietary data usage is declared due to the inherited ADME teacher. Upstream encoder overlap remains unresolved. Small adaptive-development improvement, uncertainty crossing zero after multiplicity adjustment, CYP1A2 degradation and incomplete physical meaning of descriptor proxies limit claims. Blinded missing-feature coverage is recorded explicitly. The accepted submission is identified below; acceptance for processing does not establish predictive accuracy.

Final blind coverage: 735/750 complete native ShEPhERD2 vectors. Count/graph/Snakes wholly finite rows: 737/750; per-cell missingness is retained.

## Submitted artifact identity

- Alias: `rdk-CYP-Inhibition-002`.
- Package: `000023-inhibition-582dfc3eed`.
- Prediction CSV SHA-256: `a7928c76a9898aa294591788fc68ec5be2d97a67729f1916ac19740aab3786e8`.
