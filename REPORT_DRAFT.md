# AI Intern Exercise — Predicting Cas9 Gene-Editor Activity

## 1. Objective and dataset

This project evaluates the proposal to combine pretrained protein sequence and diffusion-based structure representations for prediction of gene-editor activity.

The supplied dataset is `CAS9_STRP1_Spencer_2017_positive`, containing 8,117 single Cas9 mutants from a positive-selection deep mutational scanning assay. The target is the continuous `DMS_score`, representing an activity-related phenotype measured in this specific assay.

The dataset supports prediction of assay-specific Cas9 activity/fitness, not a universal measure of gene-editing performance. Important limitations include the single protein/organism, one assay condition, single-mutant coverage only, and the fact that experimental activity is also affected by expression and other assay-specific factors.

## 2. Evaluation protocol

The primary split is a position-held-out split. All mutations at a residue position belong to the same split, so no position is shared between train, validation and test. This tests generalization to mutations at unseen Cas9 positions.

A random mutation split is retained as a secondary interpolation/challenge setting, where different mutations at the same position can appear in different splits.

The primary metrics are RMSE, MAE, Pearson correlation, Spearman correlation and R2. Ridge regularization is selected using validation data only. The primary test set is kept untouched until final evaluation.

## 3. Baselines

Three simple baselines were implemented:

- Global mean
- Amino-acid substitution identity
- Amino-acid substitution + numerical sequence position

On the position-held-out test set, the substitution baseline obtained RMSE 0.5846, MAE 0.3943, Pearson 0.0935, Spearman 0.1396 and R2 0.0031.

These results establish that simple mutation identity contains a small amount of predictive signal, while numerical position adds little.

## 4. Executed pretrained-model experiment

The executed sequence-model experiment uses `facebook/esm2_t30_150M_UR50D`.

ESM-2 is used as a frozen feature extractor. The 1,368-residue Cas9 sequence is represented using two overlapping windows of length 1,022, producing a Cas9-wide matrix of shape 1,368 x 640.

For each mutation, the model input is:

- 640-dimensional ESM-2 embedding at the WT mutation position
- 20-dimensional one-hot encoding of the mutant amino acid

Total representation: 660 dimensions.

Two prediction heads were tested:

1. Ridge regression
2. MLP with architecture 660 -> 128 -> 64 -> 1

The ESM encoder is frozen; only the prediction head is trained.

## 5. Results

On the position-held-out validation set:

| Model | RMSE | MAE | R2 |
|---|---:|---:|---:|
| Substitution baseline | 0.5871 | 0.4055 | 0.0173 |
| ESM-2 + Ridge | 0.6418 | 0.4542 | -0.1743 |
| ESM-2 + MLP | 0.7403 | 0.5577 | -0.5627 |

After selecting the hyperparameters, the models were retrained on train + validation and evaluated on the untouched primary test set:

| Model | RMSE | MAE | Pearson | Spearman | R2 |
|---|---:|---:|---:|---:|---:|
| Global mean | 0.5855 | 0.3969 | — | — | ~0 |
| Substitution | 0.5846 | 0.3943 | 0.0935 | 0.1396 | 0.0031 |
| Substitution + Position | 0.5855 | 0.3949 | 0.0864 | 0.1243 | ~0 |
| ESM-2 + Ridge | 0.6042 | 0.4310 | 0.1893 | 0.1791 | -0.0648 |
| ESM-2 + MLP | 0.7218 | 0.5348 | 0.0704 | 0.0792 | -0.5196 |

The current results do not show an improvement from the tested single-position ESM representation under the primary unseen-position evaluation.

## 6. Failure analysis and interpretation

The frozen ESM representation may contain useful protein information, but the executed interface only exposes the representation of the WT residue at the mutation site plus mutant identity. A linear head may be too restrictive, while the small MLP did not improve generalization.

The results therefore do not justify claiming that pretrained sequence representations improve Cas9 activity prediction for unseen positions in this setup.

The experiment also does not test whether diffusion-based structural features improve the task. That part of the original proposal remains untested.

## 7. Proposed architectural modification

Based on the executed evidence, the proposal should be modified rather than accepted unchanged.

A practical architecture for a next experiment is:

WT Cas9 sequence
-> frozen ESM-2
-> local residue/context representation
-> mutant identity
-> activity prediction head

A structural/diffusion component should be added only after an isolated experiment demonstrates that structure-derived features provide useful signal on this assay.

## 8. Ablation and challenge experiment

To be completed in the final experiment:

- Ablate the 20-dimensional mutant identity feature while keeping the ESM residue representation.
- Test a local pooled ESM representation around each mutation.
- Evaluate the selected ESM model on the random mutation split as a separate interpolation/challenge experiment.

## 9. Reproduction and resources

Notebooks:
- `01_data_preprocessing_and_splitting.ipynb`
- `02_baselines_and_protein_model.ipynb`
- `03_esm2_activity_prediction.ipynb`

Generated results include split assignments, processed data, ESM embeddings, held-out ESM predictions and baseline/ESM result tables.

Compute used: CPU for ESM-2 inference and downstream model training.

AI tools used: ChatGPT for experiment planning, coding assistance, debugging and interpretation.

Unfinished work at this stage: local-context extension, mutant-feature ablation, random-split challenge experiment, and any direct diffusion/structure-feature experiment.
