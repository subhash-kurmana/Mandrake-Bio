# AI Intern Exercise — Predicting Cas9 Gene-Editor Activity

## 1. Objective and dataset

The task evaluates a proposal to combine pretrained protein sequence representations with diffusion-based structure representations and replace the structure-prediction trunk with an activity-prediction component.

The supplied assay is `CAS9_STRP1_Spencer_2017_positive`, containing 8,117 single Cas9 mutants. The continuous target is `DMS_score`, an activity-related phenotype from a positive-selection deep mutational scanning assay.

The measurements support prediction of activity/fitness under this specific assay. They should not be interpreted as a universal measurement of gene-editing performance. Important limitations include one Cas9 protein, one experimental setting, single mutants only, and assay effects such as expression and correct cleavage contributing to the measured phenotype.

## 2. Evaluation protocol

The primary evaluation is position-held-out. All mutations at a residue position are kept in the same split, so no residue position is shared between train, validation and test. This directly tests generalization to mutations at previously unseen positions.

A random mutation split is used as a separate interpolation/challenge experiment, where different mutations at the same position may occur in different splits.

Metrics are RMSE, MAE, Pearson, Spearman and R². Hyperparameters are selected on validation data only. The primary test set is not used for model selection.

## 3. Baselines

Three baselines were implemented:

1. Global mean.
2. Amino-acid substitution identity.
3. Amino-acid substitution identity plus numerical position.

On the primary position-held-out test set, the substitution baseline achieved RMSE 0.5846 and MAE 0.3943. Adding numerical position did not improve the result.

## 4. Executed ESM-2 predictor

The executed pretrained sequence experiment uses `facebook/esm2_t30_150M_UR50D`.

The ESM-2 encoder is frozen and used only as a feature extractor. Cas9 has 1,368 residues, so two overlapping sequence windows of length 1,022 are used. The resulting residue representation covers the full protein as a 1,368 x 640 matrix.

For each mutation, the predictor receives:
- a 640-dimensional ESM-2 embedding at the WT mutation position;
- a 20-dimensional one-hot vector for the mutant amino acid.

Total input size: 660.

Two prediction heads were tested:
- Ridge regression;
- MLP: 660 -> 128 -> 64 -> 1.

Only the downstream head is trained.

## 5. Results

### Primary position-held-out test

| Model | RMSE | MAE | Pearson | Spearman | R² |
|---|---:|---:|---:|---:|---:|
| Global mean | 0.5855 | 0.3969 | — | — | ~0 |
| Substitution | **0.5846** | **0.3943** | 0.0935 | 0.1396 | 0.0031 |
| Substitution + Position | 0.5855 | 0.3949 | 0.0864 | 0.1243 | ~0 |
| ESM-2 + Ridge | 0.6042 | 0.4310 | 0.1893 | 0.1791 | -0.0648 |
| ESM-2 + MLP | 0.7218 | 0.5348 | 0.0704 | 0.0792 | -0.5196 |

The tested ESM representation therefore did not improve RMSE or MAE over the simple substitution baseline for unseen residue positions. The ESM-Ridge model had higher Pearson/Spearman correlation than the baseline, but its overall regression error was worse.

### Ablation

Removing the 20-dimensional mutant identity and using only the 640-dimensional ESM representation gave:

RMSE 0.6016, MAE 0.4285, Pearson 0.1917, Spearman 0.1902, R² -0.0559.

This is very close to the full ESM-Ridge result and therefore does not show a strong contribution from the explicit mutant one-hot feature in this setup.

### Separate challenge experiment

The selected ESM-Ridge model was evaluated on the random mutation split without retuning:

RMSE 0.6437, MAE 0.4325, Pearson 0.1774, Spearman 0.2025, R² 0.0067.

The random-split substitution + position baseline was RMSE 0.6384. Thus the ESM model did not improve this independent interpolation setting either.

## 6. Failure analysis

The executed ESM interface uses a single WT residue embedding plus mutant identity. It does not explicitly represent the mutated sequence or a local window around the mutation. The linear probe may therefore miss mutation-context interactions, while the tested nonlinear MLP did not improve generalization.

The ablation also suggests that adding explicit mutant identity to this particular ESM representation produced little change in the final test error.

No conclusion about diffusion-based structure features is made because they were not executed.

## 7. Recommendation and next experiment

Based on the executed evidence, the original proposal should be modified.

The evidence supports retaining pretrained sequence representations as an experimental component, but not claiming that the current frozen single-position ESM interface improves Cas9 activity prediction on unseen positions.

The next sequence-model experiment should use a local ESM context around each mutation rather than only the single residue embedding. A structural/diffusion branch should be evaluated separately after the sequence representation is improved.

A practical proposed architecture is:

WT Cas9 sequence
-> frozen ESM-2
-> local mutation-context representation
-> mutant identity
-> activity prediction head.

Only after an isolated structure-feature experiment shows useful signal should the diffusion/structure component be integrated.

## 8. Reproducibility and resources

Notebooks:
- `01_data_preprocessing_and_splitting.ipynb`
- `02_baselines_and_protein_model.ipynb`
- `03_esm2_activity_prediction.ipynb`

Saved results include:
- processed data;
- fixed position and random split assignments;
- ESM-2 residue embeddings;
- position-held-out ESM predictions;
- baseline and ESM result tables;
- ablation and challenge results.

Compute: CPU for ESM-2 inference and downstream training.

AI tools: ChatGPT for experiment planning, coding assistance, debugging and interpretation.

Unfinished: direct diffusion/structure-feature experiment and local-context ESM extension.
