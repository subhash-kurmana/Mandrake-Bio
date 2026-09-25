# Reproduction Instructions

## Environment

Python 3.13 was used for the notebook runs.

Install the main dependencies:

```bash
pip install -r requirements.txt
```

The ESM notebook also installs `torch` and `transformers` when needed.

## Run order

Open the notebooks in VS Code Jupyter and run them in this order:

1. `notebooks/01_data_preprocessing_and_splitting.ipynb`
2. `notebooks/02_baselines_and_protein_model.ipynb`
3. `notebooks/03_esm2_activity_prediction.ipynb`

Notebook 1 validates the supplied assay, parses mutations, checks the reference sequence, creates the fixed position-held-out and random splits, and saves processed data.

Notebook 2 trains and evaluates the simple baselines.

Notebook 3 loads the fixed splits, extracts frozen ESM-2 representations, trains Ridge and MLP prediction heads, evaluates the held-out test set, runs the ESM-only ablation, and runs the random-split challenge experiment.

## Main outputs

The notebooks write results to `results/`, including:

- `processed_cas9_dms.csv`
- `position_split_assignments.csv`
- `random_split_assignments.csv`
- `baseline_results.csv`
- `baseline_position_test_predictions.csv`
- `esm2_cas9_residue_embeddings.npy`
- `esm2_position_test_predictions.csv`
- `esm2_results.csv`
- `esm2_ablation_and_challenge_results.csv`

## Primary evaluation

The primary metric is RMSE on the position-held-out test set. Hyperparameters are selected using validation data only. The primary test set is not used for model selection.
