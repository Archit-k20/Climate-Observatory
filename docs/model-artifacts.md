# Model compatibility and honest failure states

The inference runtime requires **both genuine binaries and verified training metadata**. No metadata has been invented for the archived checkpoints. A fresh checkout may contain Git LFS pointer files; these are not usable models.

## Retrieve the original binaries

The new repository retains the active artifacts from [Climate-Risk-System](https://github.com/Archit-k20/Climate-Risk-System), including their original Git LFS object hashes. From the repository root:

```sh
git lfs install --local
git lfs pull --include="ml/models/*"
git lfs fsck
```

| File | Bytes | Original SHA-256 / LFS object ID |
| --- | ---: | --- |
| `label_encoder.pkl` | 599 | `c8fd111cbc2c167fc3e8698b47b4d3c738ca0f20476a61b652347718b3e4dd7e` |
| `resnet_eurosat.pth` | 44,806,667 | `ffb3d54b28dab17e433c9851d666de8e942f61c91e4e9cfdbbeb5e0c88e02296` |
| `vit_eurosat.pth` | 343,306,663 | `01575a97497d5efffef5da4727f1990d7cef116883968ed338be190ef7a8d5d3` |
| `xgboost_eurosat.pkl` | 81,304,173 | `d144eea0023ed7cd40337db00df36db9c5e42966bd08bc2ee546f822ea1633e7` |

On macOS, compare downloaded files with `shasum -a 256 ml/models/*`; on Linux use `sha256sum`. Matching these hashes confirms transfer integrity only. Training metadata and real-checkpoint inference remain unverified.

## Establish compatibility before enabling inference

1. Install Git LFS from its official distribution, then run `git lfs install` and `git lfs pull --include="ml/models/*"` in this repository.
2. Recover the original training configuration. Verify the label order, seven XGBoost features, input dimensions, normalization, ResNet18 architecture and ViT base configuration against each checkpoint. This repository's dataset loader is evidence of the intended transform, not proof of how the archived weights were trained.
3. Create an ignored `ml/models/manifest.json` **only after compatibility is established**:

```json
{
  "version": "your-verified-checkpoint-version",
  "training_metadata_verified": true,
  "preprocessing": "eurosat-rgb224-imagenet-xgb64-canny-glcm-v1",
  "labels": ["AnnualCrop", "Forest", "HerbaceousVegetation", "Highway", "Industrial", "Pasture", "PermanentCrop", "Residential", "River", "SeaLake"],
  "sha256": {
    "label_encoder.pkl": "verified-sha256",
    "resnet_eurosat.pth": "verified-sha256",
    "vit_eurosat.pth": "verified-sha256",
    "xgboost_eurosat.pkl": "verified-sha256"
  }
}
```

The label list must equal the recovered encoder's exact class order. Do not simply copy this example or set the verification flag to bypass missing evidence. Compare SHA-256 digests with the repository's trusted LFS OIDs. Load only trusted pickle/joblib artifacts.

Inference reuses `ml/features/extract_features.py` (64px RGB means, grayscale Canny density and GLCM contrast/homogeneity/energy) and the shared dataset transform (224px RGB, ImageNet normalization). The ViT configuration is created locally; startup does not download an unrelated pretrained model. State dictionaries load strictly with `weights_only=True`.

A missing manifest, pointer file, checksum mismatch, incompatible encoder, or failed model load makes the model unavailable. A three-way ensemble disagreement is inconclusive and produces no risk score. Otherwise the original rule mapping returns a **heuristic** Low/Medium/High index of 22/55/82; agreement is reported separately as a count out of three.

This validates file and runtime compatibility, **not scientific accuracy or calibration**. No retraining, new dataset, or accuracy benchmark is included in this UI/workflow repair.
