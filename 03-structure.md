# Structure prediction — full-MSA Boltz-2 consensus

**Submission:** `rdk-CYP-Structure-002` · **Track:** CYP3A4 structure prediction

The portal acknowledged receipt on 3 November 2026 at approximately 23:30:57 UTC. This report describes the exact submitted archive, not the earlier Structure-001 workflow. No official Structure-002 score is claimed here.

## Summary

This exploratory submission supplies one newly generated full-length CYP3A4–ligand complex for each of the 20 current structure-track compounds. Its fixed generation budget is 500 native predictions: 20 compounds, five seeds and five samples per seed. Blinded-target accuracy remains unknown; the small public development comparison below supplies exploratory evidence only.

## Data and exclusions

Inputs use the current official blinded structure CSV at revision d7eb3c973dca01aad8e33c91c29e5123fd925d8f and the 503-residue organizer sequence. Historical competitor stereo expansions are excluded. The registered regenerated full alignment and 9BVB protein-only template provide sequence/template context; no experimental coordinates of the blinded target ligands or activity labels are used. The separate full CCD HEM remains present in every native output. All native samples and exclusion records are retained.

## Method

Boltz-2 version 2.2.1 uses seeds 42, 21, 99, 300, 550, five samples per seed, three recycles, 200 diffusion steps, physical potentials and a soft 6 Å ligand-to-HEM-iron pocket. The full 14,302-row source alignment undergoes native deduplication/capping to 8,192 actual rows with no additional subsampling. The 9BVB chain A protein template is not forced. Both GPU implementations were qualified on this exact recipe before expansion. This is a fresh implementation inspired by discoverybytes; no competitor modeling or evaluation code was imported.

## Validation and selection

Every native sample is checked against the exact current molecular graph and specified stereochemistry reconstructed from 3D at three fixed connectivity thresholds. All 22 checks of pinned PoseBusters 0.6.5 must complete and pass, together with the predeclared severe HEM-clash screen and full 503-residue/named 43-heavy-atom HEM context. The pool has 489 eligible poses out of 500. Here eligible means passing these declared ligand checks; it does not certify a chemically valid holoenzyme or heme electronic state.

Within each compound, eligible poses are aligned by named HEM atoms, compared with symmetry-corrected ligand heavy-atom RMSD without ligand fitting, and clustered by average linkage at 2 Å. The largest cluster supplies its medoid; ties use smaller mean pairwise RMSD and finally lexical pose identity. Model confidence and reference accuracy do not choose the pose. Weak support means fewer than five eligible poses or fewer than two represented seeds; no compounds meet that warning criterion.

Nineteen compounds have 25/25 eligible poses. The remaining compound,
`OCNT-2313038`, has 14/25. All 20 compounds retain eligible poses from all five
seeds, and none triggers the weak-support flag.

## Final prediction procedure

The 20 selected native complexes are exported without coordinate changes beyond PDB decimal serialization. Every PDB is independently rechecked for current graph/stereo and the same physical admission criteria. The exact 20-member ZIP must pass the pinned organizer validator with required LIG names, residue numbering and exact current input SMILES. Zero-eligible compounds or any export validation failure block package preparation; no pose repair or older-pose fallback is automatic.

## Public development comparison

A separate comparison applied this exact 25-pose-per-input workflow to eight prespecified public ligand inputs (nine crystal references, seven chemical families) and compared it with the actual submitted Structure-001 Glide SP workflow. This compared whole workflows with different sampling budgets and receptor contexts.

| Outcome on all eight public inputs | Structure-001 | Boltz-2 consensus |
|---|---:|---:|
| Inputs with an eligible selected complex | 8/8 | 6/8 |
| Demonstrated mean BiSyRMSD strictly below 2 Å | 1/8 | 3/8 |

Ritonavir and 4K9W had no eligible candidate among their 25 samples under the frozen ligand graph/stereochemistry checks. They remain coverage failures; they were not silently omitted from the eight-input success denominator or repaired after scoring. Among the six complete chemical families, candidate-minus-baseline LDDT-PLI was +0.2128 and BiSyRMSD was −2.5063 Å. The corresponding descriptive 95% family-bootstrap intervals were [+0.0570, +0.3856] and [−4.1137, −1.0921] Å. These conditional contrasts are not full-eight-input mean improvements.

The public inputs had been inspected previously, pretrained exposure is unknown, and uncertainty conditions on the saved selected poses. These results are not independent prospective confirmation. The separate challenge delivery has 20/20 selected complexes passing the declared checks; that does not imply 20/20 correct binding poses. The exploratory decision to submit did not alter the previously prepared prediction bytes.

## Reproducibility

The frozen generation and selection records retain all 500 native prediction
identities, every exclusion, cluster membership, selected-pose hashes and input
pins. Export records bind native poses to delivered PDBs and record coordinate
rounding, repeated physical checks and organizer validation. The evaluator pins
PoseBusters version/configuration and HEM atom identities. Source code and native
archives remain private; this description alone is not an executable reproduction.

## Limitations and disclosures

This remains an exploratory candidate with unknown blinded-target accuracy. Correlated samples and consensus are not independent accuracy evidence. The local tie rule and physical admission are explicit adaptations; upstream helper details are incomplete. Fe–Cys442 geometry is recorded separately, and full HEM inventory does not validate oxidation, spin, protonation, metal coordination or force-field chemistry. PoseBusters focuses on ligand geometry and environment rather than complete protein quality. No blinded experimental target reference was used. The public development comparison informed exploratory adoption, but it does not certify prospective challenge improvement. No proprietary assay data were added and the pretrained model was not retrained for this submission. Foundation-model training exposure has not been exhaustively audited. This repository publishes the method description; implementation code and native archives remain private, and no open-code claim is made.

## Submitted artifact identity

- Alias: `rdk-CYP-Structure-002`.
- Local package identity: `000019-structure-e7d09a002a`.
- Prediction archive: `predictions.zip`, 20 complexes, 1,469,339 bytes.
- SHA-256: `3161334766d2a494f079da1125e079beb737477fc56c5ba4992801f13208bcc1`.

The scientific description and submission-status update do not change the
submitted archive or its frozen preparation evidence.

## Method and data references

- [Official challenge](https://huggingface.co/spaces/openadmet/cyp-challenge) and [official data at the input revision](https://huggingface.co/datasets/openadmet/cyp-challenge-train-test/tree/d7eb3c973dca01aad8e33c91c29e5123fd925d8f).
- [Boltz source and model documentation](https://github.com/jwohlwend/boltz), used here at version 2.2.1.
- [PDB 9BVB](https://www.rcsb.org/structure/9BVB), used for protein-only template context.
- [PoseBusters](https://github.com/maabuu/posebusters), with pinned version 0.6.5 and retained local configuration.
- [Official challenge tutorial](https://github.com/OpenADMET/CYP-Challenge-Tutorial), for organizer validation conventions.
