# Time-dependent inhibition — CYP ADME-Fusion

**Submission:** `rdk-CYP-ADME-Fusion` · **Track:** time-dependent inhibition

The portal acknowledged receipt for processing on 3 November 2026 at 23:58:38 UTC. No official score is reported here.

## Summary

CYP3A4 combines a classical/graph
ensemble with two frozen ADME-pretrained molecular encoders. CYP2D6 retains the
exact submitted TDI-002 decisions. This exploratory variant was submitted despite missing its prespecified minimum
development MCC gain; the failed criterion remains part of the assessment.

## Data and exclusions

The comparison uses the existing 4,834 safe development compounds, including 2,748
observed CYP3A4 TDI labels, and their fixed five outer/three inner chemical-group
folds. Missing responses stay missing. Confirmation and blinded assay labels were
not used. Prediction uses only the 750 official blinded IDs and source SMILES.

Both neural students start from the [public CheMeleon checkpoint](https://zenodo.org/records/15460715)
(SHA-256 `c376624d3407204e780a0ed13a9ac097cc9bb1c13ef89cdbc633c1715c183651`).
They were trained on the same 2,048 screened external molecules using published
Novartis model-generated ADME properties, not experimental activity. Four-CYP uses
one kobs and three inhibition targets; ADME25 uses all 25 published targets.
All official activity families were excluded from source admission. Original
CheMeleon and private teacher training membership remain unresolved.

## Method

Each outer-fold CYP3A4 reference combines 18.75% LightGBM, 56.25% four-seed
ExtraTrees and 25% four-seed molecular graph probabilities. ExtraTrees uses Morgan
counts, feature-Morgan counts, MACCS, MQN and RDKit descriptors. The graph network
has two message-passing steps, 48 channels, mean/normalized-sum pooling and a
descriptor branch. All four seeds 20261103–20261106 are retained.

Each student provides a frozen 2,048-dimensional mean-pooled, row-normalized native
graph embedding plus an availability indicator. Unavailable enhanced-stereo
embeddings retain zero vectors and explicit flags. A training-only StandardScaler
and logistic classifier with C=0.01, liblinear, max_iter=1000 and seed 20261104 are
fit for each inner/outer fold. Students were pretrained with 256 AdamW steps,
batch size 32, encoder learning rate 1e-5 and head learning rate 1e-3; neither encoder is
updated using challenge labels.

The candidate is fixed at 80% reference, 10% Four-CYP and 10% ADME25 probabilities.
No weight search was conducted.

## Validation and selection

All 60 comparison readouts and 25 training-only threshold selections are retained.
Cutoffs maximize inner-OOF MCC over 0.05–0.95 in 0.05 steps, with ties nearest 0.5,
then lower. The existing reference is reproduced exactly.

| System | Mean outer-fold CYP3A4 MCC | Mean average precision |
|---|---:|---:|
| Classical/graph reference | 0.375454 | 0.509207 |
| Reference + original CheMeleon control | 0.380879 | 0.518196 |
| Reference + Four-CYP | 0.377467 | 0.519425 |
| Reference + ADME25 | 0.376064 | 0.519138 |
| Reference + both students | 0.378488 | 0.518979 |

The both-student MCC gain is 0.003034, with conditional paired whole-group 95%
interval [−0.016860, +0.023858] from 1,000 shared resamples. Three of five folds improve.
The prespecified minimum 0.005 MCC gain was missed. Submission as an exploratory variant did not change this recorded result. Development data
have been adaptively reused; intervals omit retraining and historical search
uncertainty. No confirmation or promotion certification is claimed.

## Final prediction procedure

No production refit was performed. Inference reused the five saved outer-fold systems, each
containing the exact nine-model classical/graph reference plus two student readouts.
Within each fold, inference used the fixed 80/10/10 probabilities and that fold's training-only
threshold: folds 0–4 use 0.30, 0.25, 0.40, 0.35, 0.30. Final CYP3A4 calls are majority
vote across these five binary decisions. All 750 IDs and original source SMILES
were preserved; CYP2D6 decisions from TDI-002 were joined by compound ID without
changes. TDI-002 uses a fingerprint/descriptor random forest for CYP2D6; no ADME
student or alert-panel change was applied to that endpoint.

The single-heldout-member scores above do not directly evaluate the final
fivefold deployment majority vote. The submitted endpoint columns contain binary decisions, not probabilities.

## Reproducibility

Classical/graph inference used RDKit 2025.09.6, scikit-learn 1.9.1, LightGBM 4.6.0
and CPU Torch 2.11.0. Native CheMeleon inference used a separate runtime with
Chemprop 2.2.1, RDKit 2026.9.1 and Torch 2.15.0. Frozen model states, source and
input hashes, fold memberships, student checkpoints and external-source admission
records are retained privately. Saved-state comparison replay covers 60 models,
24,170 predictions and 25 cutoffs; this verifies inference from retained states,
not identical stochastic retraining. Source code and fitted models are not
publicly released.

## Limitations and disclosures

This is an explicitly exploratory package following a failed preparation screen.
The original encoder control has a higher MCC point estimate; added value from
ADME pretraining is unconfirmed. Missing enhanced-stereo embeddings and unknown
original/private-teacher pretraining overlap remain explicit. External sources are
model-generated supervision rather than experimental labels. No proprietary
dataset was directly used by this project. Encoder source terms are retained.

## Submitted artifact identity

- Alias: `rdk-CYP-ADME-Fusion`.
- Package: `000024-tdi-4a3eda70ab`.
- Prediction CSV SHA-256: `b8d7f8c20106ae065dc120e026379185cb26453fb0fa2356d1b1f3886021fd9f`.
