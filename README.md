# Investigating Inflammatory Subtypes of Depression in NHANES

This project examines whether inflammatory blood markers predict depression in a nationally representative US sample, and — as the core novel contribution — which specific depressive symptoms show the strongest inflammatory associations. Seven markers are tested against all nine PHQ-9 symptom items using weighted regression, unsupervised clustering, and machine learning.

---

## Background

Depression is highly heterogeneous: two people meeting diagnostic criteria can present with entirely different symptom profiles, suggesting that the category likely encompasses multiple biological subtypes. One candidate mechanism is inflammation. Meta-analyses have consistently shown elevated levels of CRP, IL-6, and TNF-α in depressed populations (Dowlati et al., 2010; Haapakoski et al., 2015), and elevated inflammatory markers predict antidepressant treatment resistance, raising the possibility that a subgroup of patients with high inflammatory load may require inflammation-targeting approaches (Raison et al., 2013).

However, the majority of studies treat depression as a unitary outcome — either a binary diagnosis or a total symptom score — which may obscure more specific associations. The "sickness behaviour" model (Dantzer et al., 2008) predicts that inflammation should selectively drive neurovegetative symptoms (fatigue, sleep disturbance, appetite change, psychomotor slowing) that overlap with the physiological response to infection, rather than cognitive or affective symptoms such as guilt or suicidality.

This project tests that prediction directly by running separate regression models for each of the nine PHQ-9 symptom items, asking which markers predict which symptoms after adjustment for demographic and lifestyle confounders. A clustering analysis further examines whether inflammatory profiles identify distinct biological subtypes with differing symptom signatures.

---

## Data

Data were drawn from the National Health and Nutrition Examination Survey (NHANES), a continuous cross-sectional survey conducted by the Centers for Disease Control and Prevention (CDC) that monitors the health and nutritional status of the non-institutionalised civilian US population. Two survey cycles were pooled:

- **2015–2016** (cycle I): n = 9,971 examined participants
- **2017–March 2020** (cycle J): n = 9,254 examined participants (collection halted before the COVID-19 pandemic to avoid inflammatory confounding)

**Pooled raw N: 19,225. Primary analysis N: 1,689** (complete cases on all 7 biomarkers, confounders, and PHQ-9).

The reduction from 19,225 to 1,689 reflects two filters: (1) the blood tests require attendance at a Mobile Examination Centre (MEC), which not all survey participants complete; (2) complete-case analysis was applied, dropping participants with any missing value across the required variables.

### Depression outcome

Depression was measured using the **PHQ-9** (Patient Health Questionnaire), a validated 9-item self-report scale. Each item asks how often the participant has been bothered by a symptom over the past two weeks, scored 0 (not at all) to 3 (nearly every day). Items cover: anhedonia, depressed mood, sleep disturbance, fatigue, appetite change, feelings of worthlessness, concentration difficulties, psychomotor changes, and suicidal ideation.

- **Binary depression**: PHQ-9 total ≥ 10 (standard validated threshold; sensitivity ~88%, specificity ~88% for MDD). Prevalence in this sample: **13.3%**
- **Continuous outcome**: PHQ-9 total score (0–27), mean = 3.9, used in the weighted linear regression analysis

### Survey weights

NHANES uses stratified, multistage probability sampling and deliberately oversamples certain subgroups (adults aged 60 and over, Hispanic Americans, and non-Hispanic Black Americans) to improve population estimates for those groups. Raw counts therefore do not represent the US population. Survey weights (`WTMEC2YR`) correct for this by making each participant's contribution proportional to the number of people they represent nationally.

When pooling two 2-year cycles, CDC guidelines specify dividing `WTMEC2YR` by 2 to produce a 4-year pooled weight (`WTPOOL`). This was applied throughout all regression analyses.

### Biomarkers

Seven inflammatory markers were selected based on their availability across both NHANES cycles and their established relevance to immune function:

| Marker | NHANES variable | Direction with inflammation |
|--------|----------------|----------------------------|
| High-sensitivity CRP (hsCRP) | `LBXHSCRP` | ↑ |
| White blood cell count (WBC) | `LBXWBCSI` | ↑ |
| Neutrophil % | `LBXNEPCT` | ↑ |
| Lymphocyte % | `LBXLYPCT` | ↓ |
| Neutrophil-to-lymphocyte ratio (NLR) | derived | ↑ |
| Albumin | `LBXSAL` | ↓ |
| Ferritin | `LBXFER` | ↑ |

**High-sensitivity CRP (hsCRP)** is an acute-phase protein produced by the liver in response to pro-inflammatory cytokines, particularly IL-6. It is the most extensively studied inflammatory marker in depression research and rises with infection, tissue injury, and chronic low-grade inflammation.

**White blood cell count (WBC)** reflects the total circulating immune cell burden. An elevated count indicates a more active immune response, though it is a broad measure that does not distinguish between immune cell subtypes.

**Neutrophil %** and **Lymphocyte %** are expressed as proportions of total WBC. Neutrophils are innate immune cells and first responders to inflammation; their proportion rises under stress and inflammatory conditions. Lymphocytes (T and B cells) drive adaptive immunity and typically move inversely to neutrophils during an inflammatory response.

**Neutrophil-to-lymphocyte ratio (NLR)** is derived by dividing neutrophil % by lymphocyte %. It captures the balance between innate and adaptive immunity and has emerged as a sensitive marker of systemic inflammation and psychological stress, particularly in psychiatry research.

**Albumin** is a negative acute-phase protein — during an inflammatory response, the liver downregulates albumin synthesis in favour of CRP and other acute-phase reactants. Low albumin therefore signals chronic or sustained inflammation rather than acute activation.

**Ferritin** is the body's primary iron-storage protein but also an acute-phase reactant that rises during inflammation independently of iron status. It provides a complementary index of chronic low-grade inflammation, though elevations can also reflect non-inflammatory conditions such as liver disease or haemochromatosis.

#### Why log transformation?

CRP, WBC, NLR, and ferritin are right-skewed in population data: most people have low values but a small number have extremely high values due to active infection or illness. Log transformation compresses this tail, producing approximately normal distributions and preventing extreme values from unduly influencing regression estimates. Coefficients from log-transformed predictors are interpreted as the effect of a proportional change in the marker. CRP is transformed as log(hsCRP + 1) because valid zero values exist in the data; log(0) is undefined, so adding 1 before logging avoids this.

### Confounders

Six demographic and lifestyle variables were included as confounders in all adjusted models, selected on the basis of their established associations with both inflammatory markers and depression:

- **Age** (`RIDAGEYR`): inflammation increases with age; older adults also have higher depression prevalence
- **Sex** (`RIAGENDR`): women have higher depression prevalence and differ from men on several inflammatory markers, particularly CRP
- **BMI** (`BMXBMI`): adipose tissue is metabolically active and a major source of pro-inflammatory cytokines; BMI is strongly correlated with CRP
- **Smoking** (`SMQ040_bin`): smoking elevates inflammatory markers independently of depression; coded as current smoker (every day or some days) vs non-smoker
- **Alcohol** (`ALQ130`): average drinks per day in the past 12 months; heavy alcohol use affects both inflammation and mood
- **Race/ethnicity** (`RIDRETH3`): included because NHANES intentionally oversamples minority groups and because both inflammatory marker levels and depression prevalence differ across racial/ethnic groups. Coded as dummy variables with Non-Hispanic White as the reference category

---

## Methods

### 1. Descriptive Analysis

Weighted descriptive statistics were computed for the full sample and separately by depression status (PHQ-9 < 10 vs ≥ 10). Continuous variables are reported as weighted mean ± SD; categorical variables as weighted proportions. Biomarker distributions were examined visually via histograms to confirm that log transformations produced approximately normal distributions.

Raincloud plots were used to compare biomarker levels between depressed and non-depressed participants. Each panel combines a half-violin (kernel density estimate), a box plot, and jittered individual data points, providing a complete picture of the distribution without obscuring the raw data. Mann-Whitney U tests were used for group comparisons given the non-normal raw distributions.

A Pearson correlation heatmap examined intercorrelations among the 7 markers and PHQ-9 total score. A preliminary symptom-level visualisation split participants into CRP tertiles and plotted mean PHQ-9 item scores per tertile to motivate the formal symptom-level analysis.

### 2. Logistic Regression: Binary Depression

For each of the 7 markers, a weighted logistic regression was fitted predicting binary depression (PHQ-9 ≥ 10) using `statsmodels` GLM with a Binomial family and survey weights applied as frequency weights. Two models were run per marker:

- **Unadjusted**: marker only
- **Adjusted**: marker + age, sex, BMI, smoking, alcohol, race/ethnicity dummies

Results are reported as odds ratios (OR) with 95% confidence intervals. OR > 1 indicates higher marker levels are associated with greater odds of depression; OR < 1 indicates a protective association. False discovery rate (FDR) correction was applied across the 7 adjusted p-values using the Benjamini-Hochberg procedure. Results are visualised as a forest plot.

> **Note on standard errors**: survey weights were applied as frequency weights (`freq_weights`), which treats each participant's weight as a replication count. This inflates the effective sample size and produces artificially narrow confidence intervals and near-zero p-values. The odds ratio point estimates are directionally valid, but the precision estimates should be interpreted with caution. A full design-based variance estimator using the PSU and strata variables would be the methodologically correct approach.

### 3. Symptom-Level Analysis

This is the core novel contribution of the project. Each PHQ-9 item was binarised at a score of ≥ 2 ("more than half the days"), representing clinically meaningful symptom presence. Adjusted weighted logistic regressions were then run for every combination of marker and symptom — 7 markers × 9 symptoms = **63 models** — with the same covariate adjustment as the primary analysis.

FDR correction (Benjamini-Hochberg) was applied across all 63 p-values jointly. Results are visualised as a heatmap with markers on one axis and PHQ-9 symptoms on the other. Cell colour represents the log odds ratio (red = positive association, blue = negative/protective), cells are annotated with the raw OR, and FDR-significant associations are marked with an asterisk.

The theoretical expectation, derived from the sickness behaviour literature, is that neurovegetative symptoms (fatigue, sleep, appetite, psychomotor) will show stronger inflammatory associations than cognitive or affective symptoms (concentration, self-worth, suicidality).

### 4. Additional Analyses

**Sex × CRP interaction**: A logistic regression model including a `log_CRP × sex` interaction term was fitted to test whether the association between CRP and depression differs between men and women. Stratified ORs were reported separately for males and females.

**Continuous PHQ-9**: Weighted linear regression (WLS) was run using PHQ-9 total score as a continuous outcome. Coefficients (β) represent the change in PHQ-9 total score per 1-unit increase in the log-transformed marker. FDR correction was applied across the 7 adjusted models.

### 5. Inflammatory Subtype Clustering

K-means clustering was applied to the 7 biomarkers (standardised to zero mean and unit variance) to identify inflammatory subtypes without reference to depression status. The number of clusters (k) was selected by testing k = 2 to 6 and choosing the value maximising the silhouette score — a measure of how similar each point is to its own cluster relative to the nearest other cluster, ranging from -1 (misclassified) to +1 (well-separated).

The optimal solution (k = 2, silhouette = 0.25) was characterised by:
- Mean biomarker profiles per cluster (raw values and z-scores relative to the overall mean)
- Depression prevalence and mean PHQ-9 total score per cluster
- Mean PHQ-9 item scores per cluster (heatmap)
- Demographic composition per cluster

Clusters were visualised in two dimensions using Principal Component Analysis (PCA), which compresses the 7-marker space into the two directions of greatest variance for plotting purposes. The clustering itself was performed in the full 7-dimensional space.

### 6. Predictive Modelling

Three classifiers were evaluated on their ability to predict binary depression from inflammatory markers:

- **Logistic Regression (L2)**: standard regularised logistic regression
- **Logistic Regression (L1)**: sparse regularisation, forcing weaker predictors toward zero
- **Random Forest**: ensemble of 500 decision trees

Each model was implemented as a scikit-learn pipeline (StandardScaler → classifier) to prevent data leakage during cross-validation. Class imbalance (13.3% depressed) was addressed by setting `class_weight='balanced'`, which upweights the minority class proportionally.

Performance was evaluated using **10-fold stratified cross-validation** — the data is split into 10 folds, the model trains on 9 and tests on the remaining 1, repeated across all folds. Stratification preserves the class ratio in each fold. The primary metric is **ROC-AUC** (area under the receiver operating characteristic curve), which measures discrimination across all classification thresholds and is robust to class imbalance. Average precision and balanced accuracy are reported as secondary metrics.

Models were compared across two feature sets:
- **Biomarkers only**: the 7 inflammatory markers
- **Full**: biomarkers + 6 confounders

**Permutation feature importance** was computed for the Random Forest by repeatedly shuffling each feature and measuring the resulting drop in ROC-AUC — features that matter more cause larger drops when shuffled. This is preferred over impurity-based importance for correlated features.

**Per-symptom prediction**: each of the 9 binarised PHQ-9 items was used as a separate outcome in 10-fold CV using logistic regression on biomarkers only, producing a per-symptom AUC showing which depressive symptoms are most biologically predictable from inflammatory markers alone.

---

## Results

### Sample characteristics

The analytical sample comprised 1,689 participants (mean age 47.9 years, 48.5% female) with complete data on all markers, confounders, and PHQ-9. Depression prevalence (PHQ-9 ≥ 10) was 13.3%.

### 1. Logistic regression: binary depression

All seven markers showed associations with binary depression in the expected directions after adjustment. Markers reflecting immune activation (WBC, NLR, hsCRP, neutrophil %) were positively associated with depression; markers that fall with inflammation (albumin, ferritin) were inversely associated.

| Marker | Adjusted OR | 95% CI |
|--------|-------------|--------|
| hsCRP | 1.02 | 1.017–1.020 |
| WBC | 1.76 | 1.751–1.762 |
| Neutrophil % | 1.01 | 1.007–1.007 |
| Lymphocyte % | 0.99 | 0.989–0.990 |
| NLR | 1.22 | 1.215–1.221 |
| Albumin | 0.67 | 0.664–0.667 |
| Ferritin | 0.89 | 0.887–0.889 |

> **Important caveat**: because survey weights were applied as frequency weights, the effective sample size is inflated to several million observations. All p-values are effectively zero and confidence intervals are artificially narrow. The odds ratio point estimates are directionally interpretable but precision estimates should not be taken at face value. See Limitations.

### 2. Symptom-level analysis

62 of 63 marker–symptom combinations were nominally significant after FDR correction — again reflecting the inflated effective N. The pattern of effect sizes is the more informative result.

The strongest positive associations were:

- **WBC** with concentration (OR = 1.86), appetite (OR = 1.86), and anhedonia (OR = 1.74)
- **NLR** with anhedonia (OR = 1.77) and concentration (OR = 1.34)
- **hsCRP** with anhedonia (OR = 1.46) and appetite (OR = 1.20)

**Albumin** showed the broadest protective pattern — lower albumin (reflecting higher inflammation) was associated with higher odds of fatigue (OR = 0.60), depressed mood (OR = 0.60), and suicidality (OR = 0.59). **NLR** also showed a strong inverse association with suicidality (OR = 0.37).

Broadly, neurovegetative symptoms (fatigue, appetite, sleep) and anhedonia showed the largest positive associations with immune activation markers, while suicidality showed mixed or inverse associations — consistent with the sickness behaviour hypothesis.

### 3. Inflammatory subtype clustering

K-means clustering identified **k = 2** as the optimal solution (silhouette score = 0.25). The two clusters were modestly differentiated:

| Cluster | N | PHQ-9 mean | Depression % |
|---------|---|------------|--------------|
| 0 | 869 | 4.4 | 14.0% |
| 1 | 820 | 3.9 | 12.4% |

The low silhouette score (0.25) indicates that inflammatory markers do not cleanly separate participants into discrete biological subtypes in this population sample — a finding consistent with the dimensional rather than categorical nature of both inflammation and depression.

### 4. Predictive modelling

All models performed modestly above chance, consistent with the literature showing that inflammatory markers explain a real but small proportion of depression variance.

| Model | Biomarkers only | Full (+ confounders) |
|-------|----------------|----------------------|
| LR (L2) | 0.623 ± 0.040 | 0.651 ± 0.057 |
| LR (L1) | 0.625 ± 0.040 | 0.650 ± 0.058 |
| Random Forest | 0.573 ± 0.056 | 0.592 ± 0.056 |

Logistic regression models outperformed the Random Forest, suggesting the marker–depression relationship is approximately linear on the log-odds scale. Adding demographic and lifestyle confounders improved AUC by approximately 0.02–0.03, indicating they carry predictive information beyond the markers alone.

### 5. Per-symptom predictability

Predicting individual PHQ-9 symptoms from biomarkers alone revealed meaningful variation in predictability:

| Symptom | AUC |
|---------|-----|
| Fatigue | 0.610 |
| Concentration | 0.609 |
| Sleep | 0.604 |
| Appetite | 0.603 |
| Self-worth | 0.594 |
| Depressed mood | 0.584 |
| Anhedonia | 0.575 |
| Psychomotor | 0.562 |
| **Suicidality** | **0.511** |

Neurovegetative symptoms (fatigue, sleep, appetite) and concentration were the most biologically predictable. Suicidality was essentially at chance (AUC = 0.51), suggesting it is driven by psychological and social factors rather than inflammation. This pattern directly supports the sickness behaviour model and represents the core empirical finding of the project.

### Figures

| Figure | Description |
|--------|-------------|
| `figures/forest_plot_depression.png` | Adjusted ORs for each marker predicting binary depression |
| `figures/symptom_marker_heatmap.png` | Marker × symptom OR heatmap (core novel result) |
| `figures/biomarker_by_depression.png` | Raincloud plots by depression status |
| `figures/pca_clusters.png` | Inflammatory subtypes visualised in PCA space |
| `figures/cluster_biomarker_profiles.png` | Z-scored marker profiles per cluster |
| `figures/phq9_by_cluster.png` | PHQ-9 symptom profiles per cluster |
| `figures/roc_curves.png` | In-sample ROC curves for all models |
| `figures/feature_importance.png` | Permutation importance (Random Forest) |
| `figures/per_symptom_auc.png` | Per-symptom predictability from biomarkers only |
| `figures/correlation_heatmap.png` | Pearson correlations among markers and PHQ-9 |

---

## Limitations

**Cross-sectional design.** NHANES is a single-timepoint survey, so causality cannot be established. Depression could elevate inflammation (via stress-induced immune activation, unhealthy behaviours, or medication use) as plausibly as inflammation could drive depression. Longitudinal cohort data with repeated biomarker measurements would be needed to test temporal precedence.

**Complete-case analysis.** The sample drops from 19,225 to 1,689 after requiring complete data across all seven markers, confounders, and PHQ-9. This reduction is largely driven by MEC attendance (blood tests require a clinic visit), which is itself associated with health-seeking behaviour, age, and socioeconomic factors. If participants with missing data differ systematically from those included — for example, if the least healthy individuals are least likely to attend — the complete-case sample may not be representative even within NHANES. Multiple imputation would be a more principled approach but was not implemented here.

**Frequency-weight approximation.** Survey weights were applied as `freq_weights` in `statsmodels`, which treats each weight as a replication count and inflates the effective sample size to several million. This produces p-values that are functionally zero and confidence intervals that are several times narrower than they should be. The correct approach is design-based variance estimation using the PSU (cluster) and strata variables provided by NHANES, which accounts for the complex survey design. All precision estimates (CIs, p-values) in this project should be treated as illustrative rather than inferentially valid. Odds ratio point estimates remain directionally interpretable.

**Self-reported depression.** The PHQ-9 is a validated screening tool but is not a clinical diagnosis. It cannot distinguish major depressive disorder from other conditions that elevate the score (bereavement, general distress, somatic illness). A proportion of participants classified as depressed may not meet diagnostic criteria for MDD, and the PHQ-9 items may be influenced by the physical symptoms of the inflammatory conditions themselves — for example, fatigue and appetite change caused by illness could inflate symptom scores independently of mood.

**No independent replication.** All analyses were conducted in a single dataset. Without replication in an independent sample, it is not possible to distinguish true associations from those that reflect sampling variability or overfitting. The symptom-level analysis in particular — 63 models with FDR correction — benefits from hypothesis confirmation in a separate cohort.

**Excluded markers.** Serum ferritin is included but is not a pure inflammatory marker — elevations also occur in liver disease, haemochromatosis, and metabolic syndrome. Fibrinogen, IL-6, TNF-α, and IL-1β are the most theoretically motivated markers from the meta-analytic literature (Haapakoski et al., 2015) but were either unavailable in both NHANES cycles or had excessive missingness. The results are therefore limited to the markers that happened to be available, which may not be the most sensitive or specific indicators of the inflammation-depression pathway.

---

## Reproducing the Analysis

### Requirements

```
pip install -r requirements.txt
```

### Order of execution

```
notebooks/01_data_loading.ipynb       # downloads NHANES data, saves processed parquet files
notebooks/02_eda.ipynb                # exploratory analysis and figures
notebooks/03_symptom_analysis.ipynb   # regression analyses and heatmap
notebooks/04_subtype_clustering.ipynb # k-means clustering
notebooks/05_predictive_modelling.ipynb # ML classification
```

> Note: raw data and processed files are not committed to this repository (see `.gitignore`). Running `01_data_loading.ipynb` will download the required NHANES files directly from the CDC website.

---

## References

Dantzer R, O'Connor JC, Freund GG, Johnson RW, Kelley KW. From inflammation to sickness and depression: when the immune system subjugates the brain. *Nature Reviews Neuroscience*. 2008;9(1):46–56.

Dowlati Y, Herrmann N, Swardfager W, et al. A meta-analysis of cytokines in major depression. *Biological Psychiatry*. 2010;67(5):446–457.

Haapakoski R, Mathieu J, Ebmeier KP, Alenius H, Kivimäki M. Cumulative meta-analysis of interleukins 6 and 1β, tumour necrosis factor α and C-reactive protein in patients with major depressive disorder. *Brain, Behavior, and Immunity*. 2015;49:206–215.

Kroenke K, Spitzer RL, Williams JB. The PHQ-9: validity of a brief depression severity measure. *Journal of General Internal Medicine*. 2001;16(9):606–613.

Raison CL, Miller AH. Inflammation and treatment resistance in major depression: the perfect storm. *Psychiatric Times*. 2013;30(9).

---

## Repository Structure

```
.
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_symptom_analysis.ipynb
│   ├── 04_subtype_clustering.ipynb
│   └── 05_predictive_modelling.ipynb
├── data/
│   ├── raw/          # downloaded NHANES XPT files (gitignored)
│   └── processed/    # analysis-ready parquet files (gitignored)
├── figures/          # all output plots
├── requirements.txt
└── README.md
```
