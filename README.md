# cmse410-final
# A Fully Reproducible Pipeline for Single-Cell RNA-seq Clustering

**Jared Frazier | College of Natural Science | Data Science | CMSE 410 | Spring 2026 | Michigan State University**

---

## Overview

This repository contains a fully reproducible single-cell RNA sequencing (scRNA-seq) clustering pipeline built using Python and Scanpy. The pipeline takes raw gene expression count data, performs quality control, normalization, dimensionality reduction, and graph-based clustering, then validates the results both quantitatively and biologically against a published reference annotation.

The central focus of this project is reproducibility. Every random seed is fixed, every parameter choice is documented with justification, and every figure includes an expected output description so that anyone running the notebook can verify they are seeing correct results.

**Best ARI achieved: 0.888 at Leiden resolution 0.5** against the published PBMC 3k reference annotation.

---

## Table of Contents

- [Background](#background)
- [Dataset](#dataset)
- [Pipeline Overview](#pipeline-overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [How to Run](#how-to-run)
- [Repository Structure](#repository-structure)
- [Results](#results)
- [Reproducibility](#reproducibility)
- [Known Issues and Fixes](#known-issues-and-fixes)
- [Future Work](#future-work)
- [References](#references)

---

## Background

Single-cell RNA sequencing profiles gene expression in individual cells rather than averaging across a tissue sample. The output is a matrix where each row is a cell and each column is a gene. The most important computational step is clustering, where cells are grouped by expression similarity so that each cluster represents a distinct cell type.

Despite the availability of high-quality tools, most published scRNA-seq analyses cannot be reproduced because software versions, random seeds, and parameter choices are rarely documented. This project directly addresses that gap.

---

## Dataset

**PBMC 3k — 10x Genomics**

- ~2,700 peripheral blood mononuclear cells from a healthy human donor
- 32,738 genes per cell (filtered to top 2,000 most variable genes)
- Loaded automatically via `sc.datasets.pbmc3k()` — no manual download required
- Cell types present: T cells (CD4+, CD8+), B cells, NK cells, Monocytes, Dendritic cells

**Reference annotation** for ARI validation is loaded via `sc.datasets.pbmc3k_processed()`, also automatically downloaded by Scanpy.

No external data files need to be downloaded manually.

---

## Pipeline Overview

The pipeline runs in five sequential steps:

| Step | Description | Key output |
|------|-------------|------------|
| 1 | Quality control | Removes empty droplets, doublets, and damaged cells |
| 2 | Normalization & HVG selection | Scales to 10k counts, log-transforms, selects top 2,000 HVGs |
| 3 | Dimensionality reduction | PCA (50 → 15 components), k-NN graph, UMAP embedding |
| 4 | Leiden clustering | Clusters at resolutions 0.3, 0.5, 0.8, 1.0 |
| 5 | Validation | ARI vs. reference + Wilcoxon marker gene annotation |

All figures are saved to the `figures/` directory automatically when the notebook is run.

---

## Requirements

- Python 3.10 or higher
- The following packages (see `requirements.txt` for pinned versions):
scanpy>=1.9
leidenalg
scikit-learn
matplotlib
seaborn
numpy
pandas
---

## Installation

**1. Clone the repository**
```bash
git clone https://github.com/jcf454/cmse410-final.git
cd cmse410-final
```

**2. Create a virtual environment (recommended)**
```bash
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

**4. Launch Jupyter**
```bash
jupyter notebook
```

Then open `CMSE410_Frazier_scRNAseq_FINAL.ipynb`.

---

## How to Run

1. Open the notebook in Jupyter
2. Run all cells from top to bottom using **Kernel > Restart & Run All**
3. All figures will be saved automatically to the `figures/` directory
4. The final summary cell prints the ARI scores for all four resolutions and confirms the best result

The dataset downloads automatically on first run — no manual setup required beyond installing the dependencies.

**Expected runtime:** approximately 3 to 5 minutes on a standard laptop.

---

## Results

| Resolution | Clusters | ARI Score | Assessment |
|------------|----------|-----------|------------|
| 0.3 | 6 | 0.829 | Good broad separation |
| **0.5** | **8** | **0.888** | **Best agreement** |
| 0.8 | 9 | 0.640 | Over-splitting begins |
| 1.0 | 11 | 0.496 | Significant over-splitting |

**Marker gene validation at resolution 0.5:**

| Cluster | Cell type | Key markers |
|---------|-----------|-------------|
| 0, 2 | CD4+ T cells | CD3D, CD3E, IL7R |
| 3 | CD8+ T cells | CD3D, CD8A, CD8B |
| 4 | NK cells | NKG7, GNLY, PRF1 |
| 5 | B cells | MS4A1, CD79A, CD19 |
| 1, 6 | Monocytes | CD14, LYZ, S100A8 |
| 7 | Dendritic cells | FCER1A, CST3, HLA-DRA |

---

## Reproducibility

This pipeline was built with reproducibility as a primary goal:

- **Random seed:** All stochastic steps (PCA, UMAP, Leiden) use `SEED = 42`
- **Parameter documentation:** Every parameter choice is explained in the notebook with justification
- **Raw counts preserved:** `adata.raw` is saved before normalization for use in marker gene analysis
- **Expected outputs:** Every figure cell includes a printed description of what a correct result looks like
- **Version pinning:** Package versions are printed at the top of the notebook and listed in `requirements.txt`

Running **Kernel > Restart & Run All** from a clean environment will produce identical results to those reported here.

---

## Known Issues and Fixes

- **`sc.pl.highly_variable_genes()` ax= argument removed:** Newer Scanpy versions dropped this keyword. Fixed by rebuilding the plot manually in matplotlib
- **`sc = ax.scatter()` variable collision:** The scatter plot return variable overwrote the `sc` (scanpy) module name silently. Fixed by renaming scatter variables to `sp` throughout the notebook
- **Missing closing parenthesis:** A print statement in the ARI summary section was missing a closing `)`. Fixed

---

## Future Work

- Extend the pipeline to a more complex Human Cell Atlas dataset to test generalizability
- Add automated doublet detection using Scrublet or DoubletFinder
- Implement reference-based annotation using SingleR
- Add a sensitivity analysis varying QC thresholds and HVG counts
- Refactor into reusable Python modules with unit tests

---

## References

Hubert, L., & Arabie, P. (1985). Comparing partitions. *Journal of Classification*, 2(1), 193-218.

Luecken, M. D., & Theis, F. J. (2019). Current best practices in single-cell RNA-seq analysis: a tutorial. *Molecular Systems Biology*, 15(6), e8746.

Traag, V. A., Waltman, L., & van Eck, N. J. (2019). From Louvain to Leiden: guaranteeing well-connected communities. *Scientific Reports*, 9(1), 5233.

Wolf, F. A., Angerer, P., & Theis, F. J. (2018). SCANPY: large-scale single-cell gene expression data analysis. *Genome Biology*, 19(1), 15.

10x Genomics. PBMC 3k dataset. https://www.10xgenomics.com
