# `davao-sovi`: Spatiotemporal Social Vulnerability Index (SoVI) Framework for the Davao Region (2020–2024)

[![Python 3.11+](https://shields.io)](https://python.org)
[![Open In Colab](https://google.com)](https://google.com)
[![Code License: Apache 2.0](https://shields.io)](LICENSE)
[![Data License: CC BY 4.0](https://shields.io)](DATA_LICENSE)
[![CRS: PRS92 / Zone 5](https://shields.io)](https://epsg.io)

A reproducible spatial data science framework constructing a longitudinally comparable, scale-normalized **Social Vulnerability Index (SoVI)** for **Region XI (Davao Region), Philippines** from **2020 to 2024** (`N = 30` province-year observations).

---

## Overview

This repository provides an automated data science pipeline that resolves temporal scale incompatibility and baseline drift in Philippine SoVI applications through **pooled multi-year Z-score standardization** and **Varimax-rotated PCA** across **15 multi-source Philippine Statistics Authority (PSA) indicators** spanning Davao de Oro, Davao del Norte, Davao del Sur, Davao Occidental, Davao Oriental, and Davao City. Data were extracted and tabulated via **CSPro v7.7 (2022)** and validated against official PSA statistics under **Republic Act No. 10121**, benchmarking against **Cutter et al. (2003)** and **Lloyd et al. (2022)** models.

---

## Analytical Pipeline Architecture

The workflow proceeds from microdata extraction through spatial interpolation, pooled Z-scoring, Varimax-rotated PCA, dual-notebook validation, and archive packaging:
* **Stages 1–4 (`davao_sovi_pipeline.ipynb`):** PSA/CSPro microdata ingestion, cross-validation against official reports, linear temporal interpolation with Leave-One-Year-Out (LOYO) cross-validation, and pooled Z-score standardization.
* **Stages 5–6A (`davao_sovi_pipeline.ipynb`):** Inductive PCA-Varimax dimensionality reduction, Min-Max `[0, 1]` classification into five equal-interval vulnerability tiers, PRS92 Zone 5 (`EPSG:3125`) GIS integration, Global Moran's I spatial autocorrelation, and core sensitivity checks (Steps 1–4).
* **Stage 6B (`davao_sovi_svd_montecarlo_validation.ipynb`):** Standalone execution of **Step 5** (Eckart–Young–Mirsky SVD subspace optimality bounds) and **Step 6** (`B = 10,000` Monte Carlo algorithmic stability simulations) by computing pooled Z-scores directly from `data/processed/Region_XI_SoVI_Panel_Results_2020_2024.csv` alongside reference loadings from `outputs/diagnostics/`.
* **Stage 7 (`davao_sovi_pipeline.ipynb`):** Automated CSV and ZIP packaging (`sovi_tabular_outputs.zip` and `sovi_visual_plots.zip`).

---

## Selected Indicators & Primary Data Sources (2020–2024)

The framework incorporates **15 variables** categorized across five domains (Socioeconomic, Demographic, Housing, Basic Services, and Urbanization) with standardized directional alignment (`+` for heightened vulnerability, `-` for mitigation). Inter-survey gaps across sources—including the Census of Population and Housing (CPH), Census of Population (POPCEN), Labor Force Survey (LFS), Annual Poverty Indicators Survey (APIS), Functional Literacy Survey (FLEMMS), and Family Income and Expenditure Survey (FIES)—are resolved via linear temporal interpolation bounded by official anchors and validated using LOYO cross-validation.

---

## Validation & Robustness Protocol

The framework undergoes a six-domain diagnostic suite split across two notebooks:

### Executed in Main Pipeline (`notebooks/davao_sovi_pipeline.ipynb`)
1. **Step 1 — Indicator Sensitivity:** Iterative `N - 1` variable removal and domain-wide jackknifing (`r_s >= 0.80`, class shift `ΔC <= 1`).
2. **Step 2 — Factor Weighting Sensitivity:** Unweighted additive sum vs. variance-explained proportional weighting (`r_s >= 0.85`).
3. **Step 3 — Spatial Jackknifing:** Leave-one-out provincial sub-sampling (`N = 5` units; `N_T = 25` observations) to verify rank invariance.
4. **Step 4 — Baseline Benchmarking:** Wilcoxon signed-rank (`W`) and Friedman (`Q`) tests comparing the pooled SVD/PCA pipeline against **Lloyd et al. (2022)** and **Cutter et al. (2003)**.

### Executed in Standalone Diagnostics Notebook (`notebooks/davao_sovi_svd_montecarlo_validation.ipynb`)
5. **Step 5 — Subspace Optimality Bounds:** Truncated SVD rank-`m` (`m = 4`) reconstruction error verification and paired Wilcoxon signed-rank residual tests under the **Eckart–Young–Mirsky Matrix Approximation Theorem** (`table_4_15_eckart_young_mirsky_svd_bounds.csv`, `table_4_15b_svd_rank_wilcoxon_tests.csv`).
6. **Step 6 — Algorithmic Stability (`B = 10,000` Monte Carlo):** Controlled Gaussian noise perturbations (`σ_noise = 0.05 SD`) tracking correlation matrix condition number (`κ(R) < 30.0`), determinant stability (`|R|`), Spearman rank correlation (`r_s >= 0.80`), Class Mobility Index (`M_class <= 0.15`), and observation-level 95% confidence intervals (`table_4_16_monte_carlo_stability_summary.csv`, `table_4_16b_monte_carlo_observation_ci.csv`).

---

## Automated CSV & Archive Outputs

Outputs are organized into structured directories under `data/processed/` (including the full 30-row panel `Region_XI_SoVI_Panel_Results_2020_2024.csv`, provincial summaries, pooled Z-scores, and baseline comparisons) and `outputs/diagnostics/` (containing descriptive stats, LOYO metrics, correlation matrices, KMO/Bartlett tests, rotated loadings, Moran's I statistics, sensitivity tables, SVD bounds, and Monte Carlo confidence intervals).

---

## Repository Structure & Software Environment

```text
davao-sovi/
├── data/
│   └── processed/                                          # Generated panel results, Z-scores, & baseline comparisons
├── notebooks/
│   ├── davao_sovi_pipeline.ipynb                           # Main end-to-end Google Colab / Jupyter pipeline (Steps 1–4)
│   └── davao_sovi_svd_montecarlo_validation.ipynb          # Standalone SVD bounds & Monte Carlo notebook (Steps 5–6)
├── outputs/
│   ├── figures/                                            # Choropleths, ΔSoVI maps, & statistical plots
│   └── diagnostics/                                        # Statistical, LOYO, Moran's I, sensitivity, SVD & MC CSVs
├── requirements.txt                                        # Pinned Python dependencies (Pandas, GeoPandas, SciPy, etc.)
├── CITATION.cff                                            # Machine-readable citation metadata
├── LICENSE                                                 # Apache License 2.0 (Source Code)
├── DATA_LICENSE                                            # Creative Commons Attribution 4.0 (Data & Figures)
└── README.md
```

**Replication:** Run `notebooks/davao_sovi_pipeline.ipynb` first via Google Colab (`Runtime -> Run all`) or a local Python 3.11/3.12 virtual environment (`pip install -r requirements.txt`) to generate the base CSV outputs, then run `notebooks/davao_sovi_svd_montecarlo_validation.ipynb` to execute Steps 5 and 6 (SVD optimality bounds and Monte Carlo diagnostics).

---

## Ethical Use & Scope Disclaimer

Results represent relative macro-level vulnerability tiers (`[0, 1]`) across Region XI and should guide regional disaster risk reduction (DRRM) rather than stigmatize local communities. All data processing complies with the **Data Privacy Act of 2012 (RA 10173)**.

---

## Citation & License

If you use this framework or dataset, please cite:
```bibtex
@misc{dayot2026davao,
  author       = {Dayot, Cleriza O. and Barsomo, James Marc M. and Mariano, Ronalyn L.},
  title        = {Spatiotemporal Data Analytics and Dimensionality Reduction for Social Vulnerability Assessment in {Region XI} (2020--2024)},
  year         = {2026},
  month        = nov,
  howpublished = {School of Arts and Sciences, Ateneo de Davao University / GitHub repository: \(\url{https://github.com}\)}
}
```
* **Source Code:** [Apache License, Version 2.0](LICENSE)
* **Processed Datasets & Visualizations:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](DATA_LICENSE)
