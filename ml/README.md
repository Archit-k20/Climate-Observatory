# ClimateRisk ML source and artifacts

This directory preserves the original project's land-cover models, feature extraction, risk mapping, datasets, and experiment code. The connected application uses the validated ensemble in `ensemble/combine_models.py`.

**Checkpoint binaries are stored with Git LFS. Verified training metadata is still required before inference can be enabled.** Downloading a binary and matching its hash establishes file integrity, not scientific accuracy or preprocessing compatibility.

## Initialize the runtime

The recommended environment is the root Docker image: Python 3.11, CPU PyTorch, and the pinned dependencies in `requirements-runtime.txt`. No separate ML service or GPU is required.

From the repository root:

```sh
git lfs install --local
git lfs pull --include="ml/models/*"
git lfs fsck
docker compose up --build -d
```

Then follow [the artifact verification guide](../docs/model-artifacts.md). Do not claim the checkpoints are compatible merely because the containers start. Without verified metadata, `/api/v1/health` reports the model as unavailable and queued analyses fail honestly.

## Active inference path

| Module | Responsibility |
| --- | --- |
| `ensemble/combine_models.py` | Validate manifest and hashes, load artifacts strictly, run the three classifiers, require a majority |
| `datasets/eurosat_dataset.py` | Shared 224 × 224 RGB / ImageNet-normalized transform |
| `features/extract_features.py` | Seven XGBoost inputs: RGB means, Canny edge density, GLCM contrast/homogeneity/energy at 64 × 64 |
| `risk/risk_mapper.py` | Original land-cover-to-risk categories |
| `risk/risk_engine.py` | Retrieve the rule-based assessment for a class |
| `models/` | ResNet18, ViT, XGBoost, and label encoder artifacts |

The encoder order must match the genuine training metadata. The ten expected land-cover categories are AnnualCrop, Forest, HerbaceousVegetation, Highway, Industrial, Pasture, PermanentCrop, Residential, River, and SeaLake; the guide's example order must not replace evidence from the original encoder.

Two or three matching votes produce a classification. Three different votes return `inconclusive`, with no score. The backend maps Low / Medium / High to deterministic indices 22 / 55 / 82, and stores the votes separately. These indices are **heuristic categories, not probabilities or calibrated confidence**.

## Historical material

`archive_demo/` contains earlier Bhuvan experiments, scripts, evaluation utilities, and their associated data. `explainability/` contains experiment utilities. They are retained for reference and are not invoked by the current API analysis workflow. The historical `requirements.txt` is not the pinned serving environment.

This repository does not provide a newly verified, end-to-end reproduction of training for the archived EuroSAT checkpoints. No models were retrained, no new dataset was added, and no new accuracy benchmark was established during the UI/workflow rework. Preserve the original datasets' provenance and licensing obligations.

## Verification

The backend regression suite includes deterministic voting, preprocessing consistency, and explicit unavailable-artifact checks:

```sh
docker compose exec -T postgres createdb -U climate_user climate_test
docker compose --profile test run --rm test
```

Skip database creation if it already exists. Successful workflow tests use a stub predictor. A real-model smoke test must be recorded separately after compatibility has been verified; passing the regression suite is not a claim of model accuracy.
