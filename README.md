# QSAR Modeling for Anti-Trypanosoma cruzi Drug Discovery
## Translational Drug Discovery for Neglected Tropical Diseases — FAPESP Project No. 2025/10671-7

This repository contains a cheminformatics workflow designed to predict the bioactivity of chemical compounds against *Trypanosoma cruzi*. This pipeline was developed as a strategic component of a Direct PhD project funded by the São Paulo Research Foundation (FAPESP).

---

## Scientific Context

Chagas disease, caused by the protozoan *Trypanosoma cruzi*, is a Neglected Tropical Disease (NTD) endemic to 21 countries in the Americas. Current therapeutic options are limited to benznidazole and nifurtimox. Both drugs have limited efficacy and are associated with serious side effects.

Aiming to accelerate the discovery of safer and more effective treatments, this project combines experimental biological screening data with machine learning techniques to identify, characterize, and prioritize new drug candidates.

---

## Pipeline Overview

| Step | Description |
|------|-------------|
| 1 | Automated bioactivity data collection from ChEMBL via API |
| 2 | Data curation (exclusion of enzymatic/cytotoxicity assays, duplicate removal, quality filtering) |
| 3 | IC₅₀ → pIC₅₀ conversion and generation of ECFP4 molecular fingerprints (Morgan, radius=2, 2048 bits) |
| 4 | Training and benchmark of four ML models (Random Forest, SVR, Gradient Boosting, KNN) |
| 5 | External validation against experimental data (LCFD series) |
| 6 | Applicability domain assessment via Williams Plot |
| 7 | 3D chemical space visualization via UMAP |
| 8 | In silico prediction of planned structural derivatives |

---

## Repository Structure

```
.
├── Tcruzi-cheminformatics.ipynb    
├── data/
│   └── Tcruzi_bioactivity_dataset.csv   
├── outputs/
│   ├── knn_performance.png             
│   ├── williams_plot_knn.png           
│   └── umap_3d_chemical_space.png      
├── requirements.txt
└── README.md
```

---

## Methodology

### Data Collection and Curation (Section 1 — Optional)
Bioactivity data were retrieved from [ChEMBL](https://www.ebi.ac.uk/chembl/) using the Python API (`chembl_webresource_client`). The following filters were applied:
- Standard type: IC₅₀, standard relation: `=`
- Exclusion of enzymatic assays (`assay_type != 'E'`)
- Exclusion of cytotoxicity assays (keyword filter on assay description)
- Removal of entries flagged as suspicious or invalid
- Deduplication by canonical SMILES
- Conversion of IC₅₀ values (nM) to pIC₅₀ = −log₁₀(IC₅₀ in M)

> **Note:** Because ChEMBL is continuously updated, re-running Section 1 may return a slightly different number of compounds. The curated dataset used in this study (1,234 compounds) is provided directly in `data/tcruzi_bioactivity_dataset.csv` and Section 1 can be skipped.

### Molecular Descriptors
ECFP4 fingerprints were generated with [RDKit](https://www.rdkit.org/) using `rdFingerprintGenerator.GetMorganGenerator` (radius=2, fpSize=2048).

### Machine Learning Models
Four supervised regression algorithms were trained and evaluated via [scikit-learn](https://scikit-learn.org/):

| Model | R² (Test) | RMSE (Test) |
|---|---|---|
| Random Forest | 0.750 | 0.653 |
| Support Vector (SVR) | 0.732 | 0.676 |
| K-Nearest Neighbors (KNN) | 0.732 | 0.676 |
| Gradient Boosting | 0.681 | 0.738 |

Data split: 85% training / 15% test (`random_state=42`).

### Selected Model: KNN
**K-Nearest Neighbors** (`n_neighbors=5`, `weights='distance'`) was selected as the final model based on its superior predictive proximity to the experimental IC₅₀ values obtained for the internal 1,2,4-oxadiazole (LCFD) series. Performance was further assessed via 5-fold cross-validation.

### Applicability Domain
A **Williams Plot** was constructed using standardized residuals (±3σ threshold) and leverage values (mean Euclidean distance to the 5 nearest training neighbors), delimiting the chemical space within which predictions are statistically reliable.

### Chemical Space Visualization
Dimensionality reduction was performed via **UMAP 3D** (`n_neighbors=30`, `min_dist=0.5`, `random_state=42`), colored by pIC₅₀ to highlight regions of high predicted potency.

---

## Installation

### Requirements
- Python 3.8+
- Recommended: [Google Colab](https://colab.research.google.com) (all installation cells are already included in the notebook)

### Install locally

```bash
pip install -r requirements.txt
```

**requirements.txt:**
```
chembl_webresource_client
rdkit
scikit-learn
pandas
numpy
matplotlib
seaborn
umap-learn
```

---

## How to Use

### Google Colab (recommended)
1. Upload `Tcruzi-cheminformatics.ipynb` to Google Colab
2. Upload `Tcruzi_bioactivity_dataset.csv` to the Colab session or mount Google Drive
3. Run **Section 1** only if you wish to reproduce the data collection step (optional)
4. Run **Section 2** for all ML analyses — it loads the curated dataset directly

### Locally
```bash
git clone https://github.com/luisapmarques/Tcruzi-cheminformatics.git
cd Tcruzi-cheminformatics
pip install -r requirements.txt
jupyter notebook Tcruzi-cheminformatics.ipynb
```

---

## Data and Reproducibility

- All results are fully reproducible with `random_state=42` fixed across all models and UMAP
- The curated dataset (`tcruzi_bioactivity_dataset.csv`) is provided directly in the repository to ensure exact reproducibility regardless of ChEMBL updates
- Experimental IC₅₀ and ADME data for the internal LCFD compound series are **not included** in this repository, as these are unpublished results currently in preparation for publication

---

## References

LANDRUM, G. **RDKit: Open-Source Cheminformatics Software**. Available at: http://www.rdkit.org.

McINNES, L.; HEALY, J.; MELVILLE, J. UMAP: Uniform Manifold Approximation and Projection for Dimension Reduction. *arXiv*:1802.03426, 2018.

MENDEZ, D. et al. ChEMBL: towards direct deposition of bioassay data. *Nucleic Acids Research*, v. 47, n. D1, p. D930–D940, 2019. DOI: 10.1093/nar/gky1075.

PEDREGOSA, F. et al. Scikit-learn: Machine Learning in Python. *Journal of Machine Learning Research*, v. 12, p. 2825–2830, 2011.

ROGERS, D.; HAHN, M. Extended-Connectivity Fingerprints. *Journal of Chemical Information and Modeling*, v. 50, n. 5, p. 742–754, 2010. DOI: 10.1021/ci100050t.

---

## Citation

If this pipeline is useful for your research, please cite:

> MARQUES, L. *QSAR Modeling for Anti-Trypanosoma cruzi Drug Discovery* [Software]. GitHub, 2026.
> Available at: https://github.com/seu-usuario/Tcruzi-cheminformatics
> FAPESP Project No. 2025/10671-7

---

## License

[MIT License](LICENSE)
