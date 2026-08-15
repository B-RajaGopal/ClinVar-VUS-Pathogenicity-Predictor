# ClinVar VUS Pathogenicity Predictor 🧬

A machine learning pipeline designed to classify the clinical significance of DNA mutations and evaluate Variants of Uncertain Significance (VUS) using Explainable AI (XAI). 

## 🎯 Project Objective
Clinical genomics is bottlenecked by Variants of Uncertain Significance (VUS). This project engineers a predictive decision-support pipeline that ingests raw mutational data from NCBI ClinVar, resolves severe class imbalances, prevents identifier data leakage, and outputs calibrated pathogenic probabilities for unclassified variants using GPU-accelerated ensemble models.

## 📊 Dataset & Genomic Features
The data utilizes the **NCBI ClinVar Variation Summary (GRCh38 assembly)**.
*   **Data Access:** Raw data is accessible via the [NCBI ClinVar FTP server](https://ftp.ncbi.nlm.nih.gov/pub/clinvar/tab_delimited/).
*   **Feature Engineering:** 
    * Extracted mono-, di-, and tri-nucleotide k-mer frequencies from Reference and Alternate alleles.
    * Implemented polynomial interaction terms between genomic coordinates and sequence properties (strictly excluding database identifiers to prevent target leakage).
    * Applied high-cardinality categorical filtering to prevent sparse overfitting on rare gene symbols.

## 🏗️ Pipeline Architecture
1.  **Data Preprocessing:** Handled missing values, filtered ambiguous entries, and ensured unified Label Encoding across training and VUS datasets.
2.  **Imbalance Mitigation:** Engineered custom class wrappers utilizing `compute_sample_weight('balanced')` to stabilize training across underrepresented pathogenic classes within the XGBoost framework.
3.  **Hyperparameter Optimization:** Conducted Bayesian optimization via **Optuna** (MedianPruner) across Stratified 3-Fold CV, utilizing GPU acceleration (`tree_method='hist'`, `device='cuda'`).
4.  **Explainable AI (XAI):** Integrated **SHAP (SHapley Additive exPlanations)** with `tree_path_dependent` perturbation to extract biological drivers of pathogenicity.

## 🚀 Model Performance
The pipeline evaluates standard and optimized ensemble architectures. Performance metrics are based on the macro average of the 52,000+ variant held-out test set.

| Model Architecture | Optimization | AUC-ROC | F1-Score (Macro) |
| :--- | :--- | :--- | :--- |
| **XGBoost** | **Optuna Tuned** | **0.808** | **0.71** |

*(Insert `images/roc_curve.png` and `images/confusion_matrix.png` here)*

## 🔬 Clinical Output & VUS Evaluation
The tuned model was deployed against >1.7 million Variants of Uncertain Significance to generate pathogenic probability distributions. 

*(Insert `images/hgb_optuna_prob.png` here)*

**Biological Interpretability:**
SHAP analysis confirmed the model relies on genuine biological markers rather than memorizing database indices. The highest feature attributions map directly to `GeneSymbol` and specific chromosomal loci (`Chromosome Start`). 

Notably, in the unclassified VUS evaluation, the model flagged multiple high-confidence (99.5% probability) pathogenic candidates localized to the **CFTR** gene on Chromosome 7—a well-documented locus for Cystic Fibrosis.

*(Insert `images/shap_summary_plot.png` here)*

## ⚙️ Usage & Reproducibility
```bash
git clone [https://github.com/B-RajaGopal/ClinVar-VUS-Pathogenicity-Predictor.git](https://github.com/B-RajaGopal/ClinVar-VUS-Pathogenicity-Predictor.git)
cd ClinVar-VUS-Pathogenicity-Predictor
pip install -r requirements.txt

