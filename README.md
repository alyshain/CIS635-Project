# CIS635 Semester Project - Data Fusion Methods for Estimating Funding Impacts on Educational Outcomes

![Course](https://img.shields.io/badge/CIS_635-GVSU_Spring_2026-blue)
![Data](https://img.shields.io/badge/Data-SEDA_6.0_+_NCES_F--33-green)
![Districts](https://img.shields.io/badge/Sample-12%2C100_districts-orange)

Does how you aggregate school funding data change what machine learning can predict? We test three competing formulations — per-pupil, aggregate sum, and percentage allocation — using a multi-source district-level data fusion pipeline adapted from Chango et al. (2021).

---
## Team

Alysha Nadeem · Ishra Naznin · MD Imran Khaled · Kenaniah Williams

Grand Valley State University — CIS 635: Knowledge Discovery and Data Mining — April 2026

## Research Question

Which aggregation of school district funding data — per-pupil, aggregate total, or percentage allocation — best predicts student academic achievement, and to what extent does funding explain achievement when socioeconomic context is included?

---

## Video Presentation

Presentation YouTube Video Link [here.](https://youtu.be/Zzsz9gtMA4E)
Code Demo YouTube Video Link [here.](https://youtu.be/aXckeZrQh-I)


---

## Key Results

| Metric | Value |
|---|---|
| Best accuracy | 68.94% (Experiment 1, Sum formulation) |
| Best AUC | 0.89 (Logistic regression, numerical) |
| Sample size | 12,100 districts (77.8% match rate) |
| Strongest predictor | Free/reduced lunch eligibility (r = 0.84) |
| Per-pupil expenditure correlation | r = 0.17 (weak direct relationship) |

---

## Data Sources

| Source | Dataset | Description |
|---|---|---|
| A | SEDA 6.0 | District-level achievement scores, grades 3–8, pooled 2008–2019. Empirical Bayes adjusted composite score used as target variable. |
| B | NCES F-33 (FY 2018–19) | District revenue and expenditure data. Tested in three aggregation variants: per-pupil, aggregate sum, and percentage allocation. |
| C | SEDA Covariates | SES composite, poverty rate, free/reduced lunch eligibility, racial composition, urbanicity. 12 demographic variables. |

The final merged dataset joins SEDA achievement data with F-33 finance data on district LEAID, then left-joins demographic covariates on the same key. Rows with missing values, division-by-zero artifacts, non-positive expenditure, fewer than 50 enrolled students, or per-pupil expenditure above $50,000 were removed.

### Funding Aggregation Variants (Source B)

- **Per-pupil**: All revenue and expenditure figures divided by enrollment. Conventional policy normalization. Variables: `exppp`, `instpp`, `suppp`, `cfpp`, `fedrevpp`, `locrevpp`.
- **Aggregate sum**: Raw total dollar amounts, not divided by enrollment. Preserves absolute scale of district resources. Variables: `expsum`, `instsum`, `supsum`, `cfsum`, `fedrevsum`, `locrevsum`.
- **Percentage allocation**: Each category as a fraction of its parent total. Captures spending prioritization. Variables: `instpct`, `suppct`, `cfmargin`, `fedrevpct`, `locrevpct`.

---

## Methodology

Adapted from the four-experiment progressive fusion framework in Chango et al. (2021). Each experiment was applied across all three funding formulations and two data representations (numerical and discretized), evaluated with 10-fold stratified cross-validation.

### Experiments

| Experiment | Strategy | Avg Accuracy (Sum) | Top Classifier |
|---|---|---|---|
| 1 | Merge all attributes | 66.68% | Logistic regression |
| 2 | Global feature selection (SelectKBest) | 64.87% | Logistic regression |
| 3 | Decision-level ensemble (weighted soft vote) | 64.75% | Logistic regression |
| 4 | Ensemble + per-source feature selection | 63.97% | Logistic regression |

Naive merging (Experiment 1) consistently outperformed more complex fusion strategies across all three funding formulations, in contrast to Chango et al. (2021) where feature selection and ensembles improved results.

### Classifiers

Mapped from Weka (original paper) to scikit-learn equivalents:

| Original (Weka) | scikit-learn Equivalent |
|---|---|
| J48 | DecisionTreeClassifier (Gini, max_depth=10) |
| REPTree | DecisionTreeClassifier (min_impurity_decrease=0.01) |
| RandomTree | DecisionTreeClassifier (random splitter, sqrt features) |
| JRip | LogisticRegression (max_iter=2000) |
| NNge | KNeighborsClassifier (k=5) |
| PART | DecisionTreeClassifier (entropy, max_depth=8) |

### Preprocessing

- **Numerical**: Min-Max normalization to [0, 1].
- **Discretized**: Equal-width binning into three categories (Low=1, Medium=2, High=3) after normalization.
- **Target variable**: Tertile (quantile) binning into Low / Medium / High achievement districts, producing approximately balanced classes (~33.3% each). Equal-width binning was not used for the target as it produced extreme class imbalance (~0.3% Low) given the near-normal distribution of achievement scores.

### Ensemble Fusion (Experiments 3 & 4)

Weighted soft voting across three per-source classifiers with weights A=2, B=1, C=2. Source B (finance) received lower weight as the primary research variable under investigation, mirroring the weighting logic in Chango et al. Equal-weight ensembles (1:1:1) were tested for comparison and produced modestly lower performance.

---

## Key Findings

- Socioeconomic variables dominate feature selection in every formulation. No finance variable from Source B survives global selection when competing against demographic predictors.
- Aggregate sum funding preserves the strongest predictive signal, suggesting that per-pupil normalization removes meaningful variation in absolute district resources.
- Federal revenue per pupil is a negative predictor of achievement, acting as a poverty proxy that reflects Title I targeting of high-need districts.
- Local revenue per pupil is a positive predictor, consistent with property-tax-based wealth disparities across districts.
- Differences between funding formulations are under 0.6% average accuracy across all experiments. The choice of aggregation does not meaningfully alter model performance.
- Discretized data provides a marginal advantage in Experiment 3 but not consistently elsewhere, unlike the original paper where discretization uniformly helped.

---

## Summary Table: All Experiments

| Experiment | Per-Pupil Num Acc | Per-Pupil Disc Acc | Sum Num Acc | Sum Disc Acc | Pct Num Acc | Pct Disc Acc |
|---|---|---|---|---|---|---|
| 1: Merge all | 66.50% | 66.39% | 66.68% | 66.57% | 66.79% | 66.24% |
| 2: Feature selection | 64.87% | 65.40% | 64.87% | 64.97% | 64.87% | 64.97% |
| 3: Ensembles | 65.15% | 64.82% | 64.27% | 64.35% | 64.75% | 65.36% |
| 4: Ensembles + selection | 64.68% | 64.03% | 63.23% | 63.40% | 64.00% | 64.59% |

---

## Limitations

- Cross-sectional design: finance data from a single fiscal year (2018–19) paired with achievement estimates pooled across 2008–2019.
- Small districts (fewer than 50 students) and extreme per-pupil values (above $50,000) were excluded, which may under-represent rural and very small districts.
- Achievement coarsened into tertiles for class balance; this sacrifices granularity compared to regression approaches.
- Restricted to six white-box classifiers from the original paper for comparability. More powerful models may yield higher accuracy at the cost of interpretability.
- No external validation on a held-out cohort of districts or non-US contexts.

---

## Stack

- Python, pandas, scikit-learn, matplotlib, seaborn
- Google Colab
- Data: [SEDA 6.0](https://edopportunity.org/), [NCES F-33](https://nces.ed.gov/ccd/f33agency.asp)

---

## References

1. Chango, W., Cerezo, R., & Romero, C. (2021). Multi-source and multimodal data fusion for predicting academic performance in blended learning university courses. *Computers & Electrical Engineering*, 89, 106908.
2. Meng, T., Jing, X., Yan, Z., & Pedrycz, W. (2020). A survey on machine learning for data fusion. *Information Fusion*, 57, 115–129.
3. Roy, J. (2011). Impact of school finance reform on resource equalization and academic performance: Evidence from Michigan. *Education Finance and Policy*, 6(2), 137–167.

---
