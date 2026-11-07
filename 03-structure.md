# Structure prediction — Boltz-2 consensus

> [!NOTE]
> **TL;DR**
>
> **CYP3A4 sequence + ligand structure + heme → one protein–ligand complex**
>
> - **Coverage:** **20/20 compounds**, each with a full-length CYP3A4–ligand–heme complex.
> - **Method:** Boltz-2 generates **25 poses per compound**; identity/geometry filtering precedes largest-cluster medoid selection.
> - **Quality checks:** **489/500 native poses** are eligible; all 20 exported complexes pass the declared checks.
> - **Conditioning:** full-length CYP3A4, complete heme cofactor, an MSA and a protein-only 9BVB template.
> - **Selection:** heme-aligned ligand RMSD, average-linkage clustering at **2 Å**, then the medoid of the largest eligible cluster.

## Inputs and generation

Inputs are the organizer's 503-residue CYP3A4 sequence and the [official ligand structures](https://huggingface.co/datasets/openadmet/cyp-challenge-train-test/tree/d7eb3c973dca01aad8e33c91c29e5123fd925d8f), preserving the specified molecular identities and stereochemistry. Each complex includes the complete HEM cofactor, including iron. Sequence context comes from a multiple sequence alignment (MSA), with [PDB 9BVB chain A](https://www.rcsb.org/structure/9BVB) as a protein-only template. No experimental coordinates of the test ligands or activity labels are used.

| Setting | Value |
|---|---|
| Model | Boltz-2, version 2.2.1 |
| Sampling | Five seeds × five samples = 25 predictions per compound; 500 total |
| Seeds | 42, 21, 99, 300, 550 |
| Generation | Three recycles, 200 diffusion steps, physical potentials enabled |
| Pocket context | Soft 6 Å ligand-to-heme-iron restraint |
| MSA | 14,302 source rows; native deduplication/capping produced 8,192 rows |
| Template | Protein-only 9BVB chain A; not forced |

## Selecting one pose

1. **Check identity and geometry.** Reconstruct the ligand graph and specified stereochemistry from 3D at three fixed connectivity thresholds. Require all 22 PoseBusters 0.6.5 checks to pass, together with the predefined severe heme-clash screen and complete protein/43-heavy-atom HEM inventory.
2. **Group similar poses.** Align eligible complexes using named heme atoms. Compare symmetry-corrected ligand heavy-atom RMSD without fitting the ligands to one another, then cluster by average linkage at 2 Å.
3. **Choose a representative.** Select the medoid—the pose with the smallest average distance to the others—from the largest cluster. Ties favor smaller mean pairwise RMSD, then lexical pose identity. Neither model confidence nor reference accuracy determines the choice.

Of 500 generated poses, **489 passed**. Nineteen compounds had 25/25 eligible poses; `OCNT-2313038` had 14/25. Every compound retained eligible poses from all five seeds. None met the weak-support definition of fewer than five eligible poses or fewer than two represented seeds. All native samples and exclusion records are retained.

## Public development comparison

The development panel contains eight public ligand inputs, nine crystal references and seven chemical families. It evaluates the same 25-pose generation and selection procedure described above. The comparator is **Glide SP docking against a fixed 4NY4-derived receptor**, selecting the eligible pose with the lowest native SP score. The receptor contexts and sampling budgets differ, so this is a comparison of complete workflows.

| Outcome across all eight inputs | Glide SP / 4NY4 | Boltz-2 consensus |
|---|---:|---:|
| Inputs with an eligible selected complex | 8/8 | 6/8 |
| Inputs with mean BiSyRMSD strictly below 2 Å | 1/8 | 3/8 |

Ritonavir and the 4K9W input had no eligible pose among their 25 samples because of the fixed graph/stereochemistry checks. They remain failures in the eight-input success denominator.

Among the six chemical families with results from both workflows, ligand–protein interaction similarity (LDDT-PLI; higher is better) increased by **0.2128**, with a descriptive 95% family-bootstrap interval of [+0.0570, +0.3856]. BiSyRMSD (lower is better) decreased by **2.5063 Å**, with a candidate-minus-baseline interval of [−4.1137, −1.0921] Å. These conditional comparisons do not estimate mean improvement over all eight inputs.

The panel is adaptively reused development data, and pretrained exposure is unknown. Bootstrap intervals condition on the selected poses, so this is exploratory evidence rather than independent confirmation. The primary panel includes 4NY4, the source of the docking receptor; excluding that input gives eligible coverage of 7/7 for docking versus 5/7 for Boltz-2, with success below 2 Å of 0/7 versus 2/7.

## Limitations and availability

All 20 selected complexes pass the declared graph/stereochemistry and physical checks. Their coordinates are unchanged apart from PDB rounding; no coordinate repair or fallback substitution is applied.

Consensus measures agreement between correlated samples, not confidence that a binding pose is correct. PoseBusters primarily checks ligand geometry and its environment; it does not validate the whole protein. Keeping the full heme cofactor also does not establish correct oxidation, spin, protonation or metal coordination. Fe–Cys442 geometry was recorded separately.

No proprietary assay data were added, and Boltz-2 was not retrained. Foundation-model training overlap has not been exhaustively audited. Code and native archives remain private, with generation settings, all samples, exclusions and selected-pose/export provenance retained.

No official score is reported here.

**Acknowledgment.** The Boltz-2 sampling and consensus-selection workflow was inspired by discoverybytes’ method report. We used current organizer-specified stereochemistry, added pose-level identity and geometry filtering before clustering, and evaluated the workflow on a public crystal-structure development panel.

## References

- [Boltz source and model documentation](https://github.com/jwohlwend/boltz).
- [PoseBusters](https://github.com/maabuu/posebusters).
- [Official challenge](https://huggingface.co/spaces/openadmet/cyp-challenge) and [validation tutorial](https://github.com/OpenADMET/CYP-Challenge-Tutorial).

---

**Accepted:** 3 November 2026, approximately 23:30:57 UTC.

**SHA-256 (`predictions.zip`):** `3161334766d2a494f079da1125e079beb737477fc56c5ba4992801f13208bcc1`.
