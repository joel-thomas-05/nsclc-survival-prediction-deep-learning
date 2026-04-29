# NSCLC CT Survival Prediction

This project uses CT tumor slices and segmentation masks to predict survival outcome in non-small cell lung cancer (NSCLC). The workflow extracts deep image features with ResNet50, reduces the feature space with PR-Isomap, and trains a neural network classifier for binary survival prediction.

The model is developed using the Stanford Radiogenomics cohort and externally evaluated on the LUNG3 cohort.

## Project Overview

The goal of this project is to test whether CT-based tumor image features can provide predictive signal for NSCLC survival status.

Each patient can have multiple selected CT slices. The notebook processes each slice, extracts image features, predicts survival at the slice level, and then aggregates slice predictions to produce one patient-level prediction.

## Workflow

The notebook follows this pipeline:

1. Load Stanford and LUNG3 survival labels
2. Match CT slice files to patient IDs
3. Crop CT slices around the tumor segmentation mask
4. Normalize CT intensity values using the tumor region
5. Resize slices to 224 × 224 and prepare them for ResNet50
6. Extract 2048-dimensional ResNet50 features
7. Split Stanford data by patient to avoid slice leakage
8. Reduce feature dimensionality using PR-Isomap
9. Train a neural network survival classifier
10. Tune the prediction threshold using Stanford validation data
11. Evaluate performance on the external LUNG3 dataset
12. Aggregate slice-level predictions into patient-level predictions
13. Plot ROC curves, confusion matrices, and training curves

## Methods

### CT Preprocessing

The CT slices are processed using tumor-guided preprocessing. If a segmentation mask is available, the image is cropped around the tumor region with a small margin. Intensity normalization is performed using the tumor pixels. If no tumor mask is available, the full slice is normalized as a fallback.

Each slice is resized to 224 × 224 and converted into a three-channel image so it can be passed into ResNet50.

### Feature Extraction

A pretrained ResNet50 model is used as a feature extractor. The classification head is removed, and the model outputs a 2048-dimensional feature vector for each CT slice.

### Dimensionality Reduction

PR-Isomap is used to reduce the high-dimensional ResNet50 feature vectors into a lower-dimensional representation. The notebook tests multiple values for:

- `n_neighbors`
- `n_components`

The best setting is selected using validation ROC-AUC on the Stanford cohort.

### Survival Prediction

A neural network classifier is trained on the reduced PR-Isomap features. The model predicts binary survival status:

- `0` = dead
- `1` = alive

Class weights are used to account for label imbalance.

### Patient-Level Aggregation

Because each patient can have multiple CT slices, slice-level probabilities are averaged to produce one patient-level survival prediction. Patient-level evaluation is the main result because survival status belongs to the patient, not to an individual image slice.

## External Validation

The final model is evaluated on LUNG3 as an external validation cohort. The notebook reports:

- Slice-level accuracy
- Slice-level balanced accuracy
- Slice-level ROC-AUC
- Patient-level accuracy
- Patient-level balanced accuracy
- Patient-level ROC-AUC
- Confusion matrix
- Classification report

## Files

```text
NSCLC_CT_Survival_Prediction.ipynb
README.md
