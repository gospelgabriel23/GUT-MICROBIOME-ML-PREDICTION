# Predicting Colorectal Cancer Status from Gut Microbiome Composition

A machine learning pipeline that classifies gut microbiome samples as Healthy or Colorectal Cancer (CRC) using 16S rRNA amplicon sequencing data, with model interpretation via SHAP.

## Motivation

This project was built as a hands-on introduction to applying machine learning methods to microbiome data, with the goal of eventually extending the approach to hospital-associated microbiome surveillance (e.g., NICU pathogen colonization). It demonstrates the full pipeline: raw sequence-variant abundance data → taxonomic aggregation → classification → explainable AI → biological interpretation.

## Data

- **Source:** [CRC Gut Microbiome ML Data](https://www.kaggle.com/datasets/aramelheni/crc-gut-microbiome-ml-data) (Kaggle)
- **Samples:** 59 total (21 Colorectal Cancer, 19 Healthy, 19 Adenomatous Polyps); this analysis uses the 40-sample Healthy vs. Cancer subset
- **Features:** 6,693 amplicon sequence variants (ASVs) from 16S rRNA sequencing, each mapped to taxonomy (Kingdom → Species) via a companion taxonomy table

## Pipeline

1. **Data merging** — joined sample metadata (disease status) with the ASV abundance table on sample ID
2. **Feature cleanup** — removed all-zero columns
3. **Baseline model** — Random Forest classifier on raw ASV features (3-class: Healthy / CRC / Adenomatous Polyps), ~56% accuracy, close to chance for 3 classes
4. **Problem simplification** — restricted to the clearer Healthy vs. CRC binary comparison
5. **Taxonomic collapsing** — aggregated the 6,693 raw sequence columns into 279 features at the most specific available taxonomic level (Genus where possible, falling back to Family/Order/Class/Phylum), addressing the high-dimensionality/small-sample-size problem inherent to microbiome data
6. **Model evaluation** — 5-fold cross-validation, Random Forest classifier
7. **Explainability** — SHAP (SHapley Additive exPlanations) applied to identify which microbial genera drive predictions and in which direction

## Results

| Stage | Features | CV Accuracy |
|---|---|---|
| Raw ASVs, 3-class | 6,693 | ~56% (chance ≈ 33%) |
| Raw ASVs, binary (Healthy vs. CRC) | 6,693 | 72% |
| Taxonomically collapsed, binary | 279 | **85%** |

### Top predictive genera (by SHAP importance)

| Genus | Direction | Consistent with literature? |
|---|---|---|
| *Peptostreptococcus* | ↑ in Cancer | Yes — repeatedly linked to CRC tumor promotion in prior studies |
| *Fusicatenibacter* | ↑ in Healthy | Yes — butyrate producer, typically depleted in CRC |
| *Anaerostipes* | ↑ in Healthy | Yes — butyrate producer, typically depleted in CRC |
| *Lachnospira* | ↑ in Healthy | Yes — butyrate producer, typically depleted in CRC |
| *Bifidobacterium* | ↑ in Healthy | Yes — commonly reported as protective/beneficial |
| *Blautia* | ↑ in Cancer | Mixed in literature — some cohorts report the opposite direction |
| *Subdoligranulum* | ↑ in Cancer | Mixed in literature — some cohorts report the opposite direction |

## Limitations

- Sample size (n=40 for the binary comparison) is small; cross-validation fold accuracies ranged from 75% to 100%, indicating some sensitivity to which samples land in each fold
- Many ASVs could only be resolved to Family, Order, or Phylum level, limiting biological specificity for those features
- This is amplicon (16S) data, which identifies taxa but not strain-level detail or functional gene content — a limitation directly relevant to extending this approach to hospital/NICU pathogen surveillance, where strain-level and resistance-gene resolution matters more
- Findings on *Blautia* and *Subdoligranulum* diverge from some published cohorts, likely reflecting population differences or small-sample variance rather than a pipeline error

## Next steps

- Apply the same pipeline to a hospital-associated or NICU microbiome dataset
- Incorporate strain-level resolution (e.g., via shotgun metagenomics and tools like StrainPhlAn) rather than 16S taxonomy alone
- Explore antimicrobial resistance gene content as an additional feature set
- Model transmission patterns between patients/environment using a graph-based approach

## Tools used

Python, pandas, scikit-learn (Random Forest, cross-validation), SHAP, Google Colab
