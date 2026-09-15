# ECG Classification on PTB-XL

This repo trains a 1D ResNet to classify 12-lead ECGs from PTB-XL into five diagnostic superclasses, built to develop biosignal processing skills for a specific job application.

## What

The task is multi-label classification of 12-lead ECG signals from PTB-XL v1.0.3 (21,799 records, 100 Hz) into five diagnostic superclasses: NORM, MI, STTC, CD, and HYP. The official stratified fold split is used: folds 1-8 for training (17,084 records), fold 9 for validation (2,146 records), and fold 10 as the held-out test set (2,158 records). Records with no superclass label after aggregation are dropped. The model is a 1D ResNet in the resnet1d_wang style, trained from scratch on this dataset.

## Results

This repo reaches 0.9081 macro AUC on the held-out test set (fold 10), compared to approximately 0.93 for the published resnet1d_wang result from Strodthoff et al. 2021 on the same task and split.

| Model | Macro AUC | Task |
|---|---|---|
| This repo (from-scratch ResNet-1D) | 0.9081 | superdiagnostic, fold 10 |
| resnet1d_wang, Strodthoff et al. 2021 | ~0.93 | superdiagnostic, fold 10 |

Per-class AUC on fold 10, sorted by AUC descending:

| Class | AUC | Support (positives in test set) |
|---|---|---|
| NORM | 0.9388 | 963 |
| STTC | 0.9338 | 521 |
| CD | 0.9252 | 496 |
| MI | 0.9155 | 550 |
| HYP | 0.8272 | 262 |

## How to run

**Environment.** Python 3.11. Install dependencies:

```bash
pip install torch wfdb scikit-learn matplotlib numpy pandas
```

**Data.** Download PTB-XL v1.0.3 from PhysioNet and place the extracted folder at `ptb-xl-a-large-publicly-available-electrocardiography-dataset-1.0.3/` in the repo root.

**Notebooks, in order:**

1. `01_load_explore.ipynb` — loads the dataset, checks label distribution, plots sample records.
2. `02_train.ipynb` — trains the model on folds 1-8, validates on fold 9, saves `checkpoints/best_model.pt`, `checkpoints/test_preds.npy`, and `checkpoints/test_targets.npy`.
3. `03_evaluate.ipynb` — loads the saved prediction arrays (no model reload), computes AUC, confusion counts at 0.5 threshold, and the five hardest test records.

## Limitations

1. Model trained from scratch rather than initialised from published pretrained weights, unlike some benchmark results.
2. Single train run with one random seed, no ensembling or cross-validation across seeds.
3. Signals normalised per-record (z-score per lead) rather than using train-set-fitted global statistics, which may shift calibration slightly versus the benchmark protocol.
4. Class imbalance across the five superclasses not corrected in the loss function (no class weighting or resampling), which likely explains lower AUC on minority classes such as HYP.

## Relevance to biosignal and healthcare AI work

Working with PTB-XL at full scale means dealing with real clinical data: noisy labels, multi-label annotations, class imbalance, and an evaluation protocol designed to reflect clinical practice. The use of the official stratified fold split and a published benchmark as a comparison point follows how biosignal models are evaluated in the research literature. The gap between 0.9081 and the benchmark ~0.93 is a real result, not a rounding artefact, and the limitations section explains where it likely comes from. This project does not produce a clinical tool, but it covers the full pipeline from raw ECG files to a reproducible benchmark evaluation using standard methods in the field.
