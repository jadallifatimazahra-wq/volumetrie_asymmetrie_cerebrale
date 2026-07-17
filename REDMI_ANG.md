# Volumetric Asymmetry Project

# 1. Analysis of Brain Volumetric Asymmetry in Mice

This project aims to study the volumetric asymmetry of the ACB brain region in mice according to sex and alcohol consumption (Alcohol vs Water).

It combines image processing, volume computation, and statistical analyses to compare the left and right hemispheres and investigate the impact of biological and environmental factors.

The scripts allow:

* checking and correcting the symmetry of Regions of Interest (ROIs),
* computing Jacobian-corrected volumes,
* statistically analyzing asymmetry between experimental groups,
* visualizing the results through graphical representations.

The data consist of CSV files containing volumetric measurements and subject metadata.

This work is based on [a scientific publication](https://www.sciencedirect.com/science/article/pii/S1053811925000175) investigating asymmetry in the ACB across different experimental groups.

---

## Data

All Jacobian images and Regions of Interest are provided in `.nii` (NIfTI) format.

---

## symetry_atlas

<img src="atlas.png" width="100">

* Detects the Left/Right axis using Python's `aff2axcodes` function.

* Measures voxel-wise asymmetry:

  * Compares each left voxel with its mirrored right voxel.
  * Counts the number of different voxels above a numerical tolerance (`tol = 10e-6`).
  * Computes the percentage of asymmetric voxels:

`asym_vox_vox = 100 * (number of different voxels / total number of voxels)`

* Reconstructs a symmetric atlas:

  * Copies the right hemisphere and mirrors it onto the left hemisphere.
  * Preserves the middle slice when the x-dimension is odd.
  * Saves the newly generated symmetric atlas.

<img src="atlas1.png" width="100">
---

## symetry_Rois

* Detects the Left/Right axis as in the previous step.

* Comparison between the two ROIs (Left/Right):

The right ROI is mirrored (flipped) along the Left/Right axis to match the left ROI, allowing voxel-by-voxel comparison.

* Computes the number of active voxels in each ROI.

The OR operator (`|`) represents the union of voxels.

* Measures asymmetry using the difference (XOR) between the left ROI and the mirrored right ROI.

The XOR operator (`^`) identifies asymmetric voxels.

The asymmetry percentage is computed as:

`asym_percent = 100 * (number of asymmetric voxels / number of voxels in the union)`

### ROI Symmetrization

* Keeps the right hemisphere as the reference.
* Generates a symmetric left ROI by mirroring the right ROI.
* Saves the corrected left and right ROIs separately.

<img src="rois.png" width="250">
---

## 03_asy_many_roi.py

### Physical ROI Volume Computation

The physical ROI volume is computed as:

`Physical ROI Volume = Number of active voxels × Voxel volume`

Volumes are expressed in both mm³ and µm³.

### Mean Jacobian Computation

* Selects voxels belonging to the ROI.
* Computes the mean Jacobian value while ignoring non-positive values.
* Computes Jacobian-corrected volumes:

`vol_L = mean_L × physical left ROI volume`

`vol_R = mean_R × physical right ROI volume`

### Asymmetry Computation

`Asymmetry = 100 × (R − L) / L`

### Export of Results

Results from all subjects are compiled into:

`res_asy_multiROI_2BC.csv`

with the following columns:

| Num_Mouse | ROI | R_vol_mm3 | L_vol_mm3 | mean_jac_L | mean_jac_R | asymmetry_% |
| --------- | --- | --------- | --------- | ---------- | ---------- | ----------- |

---

# Statistical Analyses

The statistical analyses are located in the `stats` directory, which contains three scripts dedicated to evaluating volumetric asymmetry across multiple brain regions according to sex and drinking condition (Alcohol vs Water).

The analyses rely on:

* `res_asy_multiROI.csv`, containing volumetric asymmetry measurements.
* `ConnectDrink.csv`, containing subject metadata.

### `res_asy_multiROI.csv`

| Num_Mouse | ROI | R_vol_mm3 | L_vol_mm3 | mean_jac_L | mean_jac_R | asymmetry_% |
| --------- | --- | --------- | --------- | ---------- | ---------- | ----------- |

### `ConnectDrink.csv`

| Included | Num_Mouse | Sex | Drink |
| -------- | --------- | --- | ----- |

---

# Overall Objectives of the Statistical Analyses

The analyses aim to answer four main questions:

**Is the mean asymmetry of a ROI significantly different from zero?**

→ One-sample t-test (overall and by experimental group)

**Do experimental groups significantly differ from one another?**

→ Pairwise independent t-tests

**What are the effects of Sex, Drinking Condition, and their interaction?**

→ Two-Way ANOVA (Sex × Drink)

**Which ROIs exhibit the strongest or most significant asymmetries?**

→ Volcano plots and ranking analyses

Each ROI is analyzed independently.

---

# Statistical Analysis of Volumetric Asymmetry

All analyses are performed independently for each ROI using the asymmetry values previously computed.

---

## One-Sample t-test (vs 0)

### Objective

Determine whether the mean asymmetry significantly differs from zero, corresponding to the absence of perfect symmetry between the left and right hemispheres.

The analysis is performed:

* on the entire cohort,
* separately for each experimental group:

  * M.ALC
  * M.WAT
  * F.ALC
  * F.WAT

---

### Method

The one-sample t-test evaluates whether the sample mean differs from the reference value (0):

`t = (x̄ − 0) / (s / √n)`

where:

* x̄ = mean asymmetry
* s = standard deviation
* n = number of subjects

---

### Interpretation

The test returns:

* t statistic,

* associated p-value.

* p < 0.05 → significant asymmetry

* p ≥ 0.05 → no significant asymmetry detected

Interpretation of the sign:

* positive value → Right volume > Left volume (R > L)
* negative value → Left volume > Right volume (L > R)

---

## Pairwise Group Comparisons

### Objective

Compare independent experimental groups to determine whether volumetric asymmetry differs between them.

The following comparisons are performed:

* M.ALC vs F.ALC
* M.WAT vs M.ALC
* F.ALC vs F.WAT
* M.WAT vs F.WAT

<img src="Image1.png" width="300">
---

### Method

Welch's independent t-test is used without assuming equal variances:

`t = (x̄₁ − x̄₂) / √(s₁² / n₁ + s₂² / n₂)`

where:

* x̄₁, x̄₂ = group means
* s₁, s₂ = standard deviations
* n₁, n₂ = sample sizes

---

### Interpretation

For each ROI, the test reports:

* difference of means (group1 − group2),
* p-value,
* logp = −log10(p-value),
* effect direction:

  * Right if diff > 0
  * Left if diff < 0

A p-value below 0.05 indicates a statistically significant difference between groups.

Results are visualized using volcano plots.
<img src="Image2.png" width="300">
---

## Two-Way ANOVA (Sex × Drink)

### Objective

Evaluate the influence of two experimental factors on volumetric asymmetry:

* Sex (Male vs Female),
* Drinking condition (Alcohol vs Water),
* Sex × Drinking interaction.

This analysis determines whether the effect of alcohol depends on sex.

---

### Method

The following linear model is fitted:

`asymmetry ~ Sex + Drink + Sex:Drink`

A Type II Two-Way ANOVA is then performed to evaluate:

* the main effect of Sex,
* the main effect of Drinking condition,
* the Sex × Drink interaction.

---

### Interpretation

For each ROI, the ANOVA provides:

* p-value for Sex,

* p-value for Drinking condition,

* p-value for the interaction.

* A significant main effect indicates that the factor influences asymmetry.

* A significant interaction indicates that the effect of alcohol differs between males and females.

The results are visualized using separate volcano plots for each experimental factor.
<img src="Image3.png" width="300">
