# Explainable AI for Sustainable Plastics

**Mechanistically validated prediction of enzyme–polymer associations and chemical toxicity using SHAP-interpreted XGBoost**

This repository contains the research project for two complementary case studies in explainable machine learning for environmental sustainability. Both compare a conventional feature baseline with an extension that adds independently derived, mechanism-informed features, then examine whether model explanations are consistent with external biological, chemical, or structural evidence.

> **Scope note:** These are two separate prediction tasks, not one combined model. The enzyme task predicts recorded enzyme–polymer associations; the chemical task predicts activity at the Tox21 NR-AhR assay endpoint. NR-AhR activity is not a general measure of toxicity or environmental harm.

## At a glance

| Case study | Dataset | Task | Baseline | Mechanism-informed extension |
|---|---|---|---|---|
| 1. Enzyme–polymer association | PlasticEnz | Multi-label prediction across eight polymer classes | Sequence-derived properties and an independently recorded enzyme reaction-type annotation | Independently generated protein domain/family annotations from InterProScan |
| 2. Chemical activity screening | Tox21 | Binary prediction of NR-AhR assay activity | Ten RDKit molecular descriptors | Three literature-selected structural flags: indole, benzothiazole, and halogenated aromatic substructures |

The project emphasizes leakage-aware evaluation, directional SHAP interpretation, and validation against evidence independent of the model. The contribution is the experimental and interpretive framework—not a new machine-learning algorithm.

## Research questions

1. Do independently generated, mechanism-informed features change predictive performance compared with conventional features?
2. Which features influence predictions, and in which direction?
3. Are the most important model explanations consistent with independent sequence, structural, or published experimental evidence?
4. How does performance change when evaluation is made more demanding—for example, by preventing similar protein sequences or chemical scaffolds from crossing data partitions?

## Main findings

### Case study 1 — enzyme–polymer association

- After cleaning and deduplication, the modelling dataset contains **231 unique protein–polymer associations across 188 unique proteins** and eight polymer labels: PET, PHB, PCL, PBSA, PBAT, PLA, PHA, and Nylon.
- The train/test split contains **149 training proteins and 39 test proteins**. Protein pairs meeting the defined high-similarity criteria were grouped so near-duplicate proteins were not split across training and test.
- The 149 training proteins were divided into three similarity-group-aware folds using mixed-integer linear programming (MILP). The folds contain 49, 50, and 50 proteins, and each polymer has positive and negative examples in every fold.
- Model 1A uses nine encoded baseline columns. Model 1B adds ten independently generated domain/family features, for 19 columns in total. Target-explicit annotations were removed to avoid direct target leakage.
- Cross-validation performance was similar for the baseline and extended model: Model 1A averaged **ROC-AUC 0.738 (SD 0.150)** and **PR-AUC 0.417 (SD 0.247)**; Model 1B averaged approximately **ROC-AUC 0.732** and **PR-AUC 0.418**. The extended feature set did not materially improve aggregate predictive performance.
- The main interpretability result was polymer-specific: the PANTHER family `PANTHER_PTHR22946` was strongly associated with PET predictions in Model 1B. A separate GXSXG motif check and published evidence on dienelactone hydrolases provide convergent support for the biological plausibility of this signal. This is supportive evidence, not proof of a causal mechanism.
- **Important label caveat:** a zero indicates that an association was not recorded in the curated dataset; it does not establish that the enzyme was experimentally tested and found inactive.
- **Important rare-class caveat:** PHA, Nylon, and other low-prevalence polymer estimates are uncertain because there are few positive examples. Model 1 uses binary relevance: each polymer is modelled independently, so label dependencies are not explicitly learned.

### Case study 2 — Tox21 NR-AhR activity

- The cleaned dataset contains **6,534 unique, non-conflicting compounds**, including **760 active compounds (11.6%)**.
- Model 2A uses ten RDKit descriptors. Model 2B adds three SMARTS-derived binary flags: indole, benzothiazole, and halogenated-aromatic substructures.
- On the conventional stratified random split, Model 2A achieved **ROC-AUC 0.871** and **PR-AUC 0.497**. Model 2B achieved approximately **ROC-AUC 0.876** and **PR-AUC 0.531** in the paper draft. The gain is modest; overlapping bootstrap confidence intervals mean it should not be described as statistically significant superiority.
- Conditional SHAP analysis highlighted indole and benzothiazole flags as potentially influential positive contributors. These are rare structural patterns, so their effects should be interpreted with their small sample counts in mind.
- A scaffold-based evaluation exposed a generalization limitation. For Model 2B, random-split cross-validation yielded **ROC-AUC 0.8598 / PR-AUC 0.5459**, while scaffold-aware cross-validation yielded **0.8433 / 0.5085**. On the held-out unseen-scaffold test set, performance fell to **ROC-AUC 0.7635 / PR-AUC 0.3811**. This is evidence that random-split performance overstates performance on previously unseen chemical frameworks.
- NR-AhR is a specific receptor-activation assay endpoint. It should not be equated with whole-organism toxicity, environmental persistence, or overall environmental impact.

## Methods overview

### Shared design

For each case study:

1. Prepare and audit the data.
2. Build a conventional-feature baseline.
3. Add independently derived, mechanism-informed features.
4. Evaluate both models with the same task-appropriate protocol where possible.
5. Use SHAP to inspect feature importance and contribution direction.
6. Compare selected explanations with independent literature or structural evidence.
7. Report limitations and stricter robustness checks rather than relying only on the most favorable metric.

### Enzyme–polymer modelling

- **Input:** PlasticEnz-derived protein sequences, enzyme annotations, and recorded polymer associations.
- **Targets:** PET, PHB, PCL, PBSA, PBAT, PLA, PHA, Nylon.
- **Baseline features:** encoded `Type` annotation and seven sequence-derived properties: sequence length, GRAVY, aromatic fraction, proline fraction, cysteine fraction, theoretical pI, and instability index.
- **Extended features:** ten InterProScan-derived domain/family features after redundancy collapsing, target-explicit annotation removal, and frequency filtering.
- **Leakage control:** exhaustive pairwise sequence comparison across 188 proteins; high-similarity pairs were defined using at least 95% identity and at least 80% alignment coverage. Similarity groups were kept intact across train/test and cross-validation partitions.
- **Model:** one XGBoost binary classifier per polymer (binary relevance). Class weights were calculated from the relevant training partition only.
- **Evaluation:** three-fold similarity-group-aware cross-validation and a held-out 39-protein test set. Interpret rare-label test results cautiously.

### Tox21 modelling

- **Endpoint:** NR-AhR.
- **Baseline features:** molecular weight, calculated LogP, topological polar surface area, hydrogen-bond donor/acceptor counts, rotatable bonds, ring count, aromatic ring count, heavy atom count, and fraction of sp3-hybridised carbons.
- **Extended features:** indole, benzothiazole, and halogenated-aromatic SMARTS flags.
- **Data cleaning:** rows with missing endpoint labels, invalid SMILES, and duplicate canonical structures with conflicting labels were removed.
- **Primary evaluation:** stratified random 80/20 split (5,227 training compounds and 1,307 test compounds), with five-fold cross-validation for tuning. Classification metrics were reported at threshold 0.5.
- **Robustness evaluation:** Bemis–Murcko scaffold split with no scaffold overlap; scaffold-grouped cross-validation on the scaffold training partition.
- **Interpretability:** TreeExplainer SHAP, including conditional analyses for selected structural features.

## Reproducibility settings recorded in the project documents

| Setting | Recorded value |
|---|---|
| General random seed | 42 where specified |
| Enzyme proteins | 188 unique proteins |
| Enzyme independent test set | 39 proteins |
| Enzyme CV | 3 similarity-group-aware folds; 49/50/50 proteins |
| Protein similarity threshold | ≥95% identity and ≥80% alignment coverage |
| XGBoost, enzyme models | 200 estimators; learning rate 0.05; max depth 4; min child weight 1; subsample 0.8; column subsampling 1.0 |
| Tox21 cleaned dataset | 6,534 compounds; 760 positives |
| Tox21 random split | 5,227 train / 1,307 test |
| Tox21 hyperparameter search | Five-fold CV; 96 parameter combinations; average precision scoring |
| Tox21 selected XGBoost configuration | 200 estimators; learning rate 0.05; max depth 7; min child weight 1; subsample 0.8; column subsampling 1.0 |
| Tox21 structural flags | Indole, benzothiazole, halogenated aromatic |
| Tox21 bootstrap intervals | 2,000 test-prediction bootstrap resamples; seed 42 |
| Scaffold-based test | 4,641 training compounds / 1,893 test compounds; zero scaffold overlap |

These are settings recorded in the project documents, not a claim that every experiment has been rerun from a clean environment using this README. See **Reproducibility status** below before treating the repository as a turnkey pipeline.

## Repository contents

The exact paths and completeness of all generated artifacts should be checked against the current GitHub checkout. The project documents refer to the following artifact types:

- preprocessing and data-cleaning code;
- protein similarity groups and MILP-generated fold assignments;
- InterProScan annotations and feature matrices;
- Tox21 descriptor and structural-flag tables;
- trained model outputs and prediction metrics;
- SHAP values, feature-importance summaries, and figures;
- random-split and scaffold-based evaluation results;
- supplementary tables and audits.

Suggested layout (adapt this to the files actually present; do not create empty folders solely to match this example):

```text
.
├── README.md
├── notebooks/             # Analysis notebooks, if included
├── src/                   # Reusable preprocessing and modelling code, if included
├── data/                  # Document expected inputs; avoid committing restricted/raw data
├── figures/               # Figures used in the paper, if included
├── outputs/               # Metrics, tables, model outputs, and audit results
└── requirements.txt       # Add a verified environment file before claiming turnkey reproduction
```

## Reproducing the experiments

The two supplied Markdown documents describe the study and its settings, but they do not by themselves establish a verified end-to-end execution command or a complete dependency lockfile. Before presenting this repository as fully reproducible, verify that the checkout includes the required scripts/notebooks, input data access instructions, and environment specifications.

A reproducible run should document, at minimum:

1. Python and package versions (including XGBoost, scikit-learn, RDKit, SHAP, Biopython, SciPy, and pandas).
2. How to obtain PlasticEnz and Tox21 and which source versions/files were used.
3. How to run preprocessing and generate the cleaned datasets.
4. How to generate protein similarity groups and MILP folds.
5. How to run each baseline and extended model, including the fixed seeds and fold definitions.
6. How to generate SHAP analyses, figures, bootstrap intervals, and scaffold-based results.
7. Where expected outputs are written and how reported metrics can be checked against them.

Do not include private credentials or assume that raw datasets can be redistributed. Follow the source datasets' terms and cite their original publications. If large intermediate files are omitted from Git, explain how to regenerate them or where they can be obtained.

## Interpretation and limitations

- SHAP explains a model's learned associations; it does not establish causation.
- Mechanistic features are useful only if independently generated and carefully checked for target leakage.
- PlasticEnz labels are recorded associations, not complete experimental positive/negative labels.
- Protein-level data are small and polymer prevalence is uneven.
- Binary relevance does not model dependencies among polymer labels.
- Tox21 NR-AhR activity is an assay endpoint, not a comprehensive toxicity label.
- Random compound splits can place related chemistry in both partitions; scaffold-based results are lower and more relevant to unseen-scaffold generalization.
- The mechanism-informed extensions primarily strengthen interpretation; their predictive gains are modest or absent in aggregate metrics.

## Data and code availability

The project documents identify the intended repository as:

[`https://github.com/shazahmedsyed-lab/ICSEED-2026-1`](https://github.com/shazahmedsyed-lab/ICSEED-2026-1)

Before publishing this README, confirm that this is the correct public repository URL and that its committed contents match the analyses described here. Add links to the exact notebooks, scripts, and data/output directories once their paths have been verified.

## Citation

If you use or discuss this work, cite the project as a research manuscript and replace this provisional citation with the final conference or journal reference when available:

> Ahmed, S. S., Shoaib, S. M., Faraz, S. H., & Basha, S. F. A. *Explainable AI for Sustainable Plastics: Mechanistically-Validated Prediction of Enzyme-Polymer Associations and Chemical Toxicity Using SHAP-Interpreted XGBoost.* ICSEED 2026 project manuscript (draft).

## Acknowledgment

This project combines established tools—XGBoost, SHAP, RDKit, InterProScan, and sequence-similarity-aware evaluation—in a shared, evidence-checking workflow. Please cite the original datasets, tools, and relevant scientific publications when reusing components of the work.
