# Unsupervised Clustering of Colon Histopathology Images

MSc coursework (Machine Learning case study), University of Glasgow.

Can colon tissue images be grouped by tissue type **without labels**? I compare clustering
methods over deep feature representations of the same images, then use the held-out tissue
labels only to evaluate the clusters.

## Setup

- **Representations:** image embeddings from ResNet50, VGG16, InceptionV3 and a PathologyGAN
  encoder (PGE)
- **Dimensionality reduction:** PCA vs **UMAP**
- **Clustering:** K-Means vs **HDBSCAN**, including soft-clustering to recover points HDBSCAN
  initially labels as noise
- **Evaluation:** external metrics (V-measure, ARI) against the true tissue types, and internal
  metrics (silhouette, **DBCV** for density-based clusters)

## Results

**K-Means** (k = 9, the number of tissue classes; every point assigned):

| Features | ARI | V-measure | Silhouette |
|---|---|---|---|
| PGE + PCA | 0.253 | 0.378 | 0.125 |
| PGE + UMAP | 0.403 | 0.552 | 0.531 |
| ResNet50 + PCA | 0.384 | 0.508 | 0.159 |
| **ResNet50 + UMAP** | **0.462** | **0.663** | **0.529** |

Louvain community detection on a k-NN graph (ResNet50 + UMAP): ARI 0.276, V-measure 0.619.

**HDBSCAN** (density-based; points it can't place are labelled noise). The V-measure is
computed **on clustered points only**, so the noise column matters as much as the score:

| Features | V-measure | DBCV | Noise (of ~5,000) |
|---|---|---|---|
| PGE + PCA | 0.160 | - | - |
| PGE + UMAP | 0.630 | 0.254 | - |
| ResNet50 + PCA | 0.881 | 0.054 | 3,896 (78%) |
| **ResNet50 + UMAP** | **0.723** | **0.287** | **594 (12%)** |
| + soft-clustering noise recovery | 0.721 | 0.287 | 588 |

## Takeaways

- **High V-measure can hide a broken model.** ResNet50 + PCA scores 0.88, but only because
  HDBSCAN discards 78% of the images as noise, and a DBCV near zero confirms the clusters have
  no real structure. Coverage has to be reported alongside the score.
- **UMAP is what makes density-based clustering work here.** HDBSCAN's density estimates fall
  apart in PCA space.
- **ResNet50 + UMAP was the strongest representation** with both methods. It beat the
  domain-specific PathologyGAN features (0.72 vs 0.63 V-measure with HDBSCAN, and 0.66 vs 0.55
  with K-Means).
- **Soft-clustering noise recovery made no real difference.** It rescued only 6 of 594 noise points.

## A correction

In the original version of this notebook, the three ResNet50 HDBSCAN cells printed a stale
variable (`v_score_umap`) instead of the score they had just computed, so all three displayed
0.630. They have been corrected, re-run against the same feature files, and the two markdown
cells that quoted the old numbers have been updated. The noise counts and DBCV scores from the
re-run match the original outputs exactly.

## Data

The pre-extracted feature files (`*_dim_reduced_feature.h5`) were provided by the course and are
not included. Place them next to the notebook to run it.
