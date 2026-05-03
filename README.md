# VDS — Variational Decision Shield

> *A domain-agnostic decision framework grounded in the minimisation of a composite variational functional.*

[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Seeds: 5](https://img.shields.io/badge/seeds-5-orange.svg)]()
[![Domains: 2](https://img.shields.io/badge/domains-2-purple.svg)]()

---

## Overview

VDS selects a decision policy **π\*** by solving:

$$\pi^* = \arg\min_{\pi \in \Pi} \; J(\pi), \qquad J(\pi) = L(\pi) + \alpha\,U(\pi) + \beta\,R(\pi) + \gamma\,C(\pi) - \delta\,I(\pi)$$

| Term | Role |
|------|------|
| **L** | Task-specific loss (prediction error / operational cost) |
| **U** | Predictive uncertainty (entropy / state-stress score) |
| **R** | Risk (asymmetric classification risk / grid overload) |
| **C** | Complexity regularisation (held in reserve; set γ = 0 for both domains here) |
| **I** | Bounded information utility (decision margin / operational headroom) |

The same functional structure, the same minimisation principle, and the same coefficient vocabulary are applied across two structurally different domains. This cross-domain consistency is the central empirical claim of the paper.

---

## Repository Structure

```
VDS_Variational_Decision_Shield/
│
├── VDS_Variational_Decision_Shield.ipynb   # Main notebook (Sections 1–12 + ablation)
│
├── figures/                                # All publication figures (PNG)
│   ├── classification_risk_coverage.png
│   ├── classification_attack_success_curve.png
│   ├── classification_selective_under_pgd.png
│   ├── energy_robustness_curves.png
│   ├── energy_stability_curves.png
│   ├── ablation_classification_1d.png
│   └── ablation_energy_1d.png
│
├── tables/                                 # Aggregated result tables (CSV)
│   ├── vds_classification_clean_agg.csv
│   ├── vds_classification_attack_agg.csv
│   ├── vds_energy_clean_agg.csv
│   ├── vds_energy_attack_agg.csv
│   ├── vds_cls_ablation_{alpha,beta,delta,c_abstain,fn_weight}.csv
│   ├── vds_energy_ablation_{alpha,beta,gamma,delta}.csv
│   ├── vds_cls_operating_points.csv
│   ├── vds_energy_operating_points.csv
│   └── vds_cross_domain_summary.csv
│
├── paper/
│   ├── VDS_Methods_Section.docx            # Methods section draft
│   └── VDS_Results_Section.tex             # Results & Discussion (compile-ready LaTeX)
│
└── README.md
```

---

## Domains

### Domain I — Selective Classification

The action space is **Π = {predict 0, predict 1, abstain}**. VDS solves the per-instance argmin of J, treating abstention as a third action with a fixed cost c_abstain rather than as a post-hoc threshold rule.

**Dataset:** Wisconsin Breast Cancer (569 instances, 30 features, 25 % test split, stratified).  
**Classifier:** Logistic regression — chosen to enable analytic PGD attacks.  
**Attacks:** Random Gaussian perturbation + PGD (L₂, 25 steps).

| Metric | Argmax baseline | VDS (balanced default) |
|--------|----------------|------------------------|
| Accuracy / Selective accuracy | 0.972 ± 0.013 | **0.985 ± 0.005** |
| Coverage | 1.000 | 0.909 ± 0.009 |
| Selective risk (clean) | 0.028 | **0.015** |
| Selective accuracy (PGD ε = 0.2) | 0.769 | **0.851 ± 0.015** @ cov 0.709 |

*All values: mean ± std across 5 seeds.*

**Known limitation.** At high PGD budgets (ε ≥ 0.6) the attack saturates predicted probabilities, manufacturing false confidence and causing VDS to commit incorrectly. This failure mode is documented explicitly; no coefficient choice resolves it. Defending against it requires an uncertainty signal independent of the base classifier's posterior.

---

### Domain II — Cyber-Physical Energy Management

A 14-day, 15-minute-resolution battery + grid + renewable + load system. VDS-EMS minimises J over a discretised 7 × 7 charge/discharge action grid at each timestep.

**Attacks:** Random FDI (multiplicative Gaussian) + worst-case FDI (bilevel grid search, 27 candidates/step).

| Metric | Baseline | VDS-EMS (balanced default) |
|--------|----------|---------------------------|
| Total cost (clean) | 1049.4 ± 5.4 | 1052.7 ± 5.4 (+0.31 %) |
| SOC standard deviation (clean) | 1.285 ± 0.049 | **1.096 ± 0.067** (−14.7 %) |
| Mean J̄ (worst-case FDI, ε = 0.4) | 1.705 | **0.821** (−51.8 %) |
| Mean J̄ (worst-case FDI, ε = 1.0) | 3.265 | **1.127** (−65.5 %) |

*All values: mean ± std across 5 seeds.*

---

## Ablation Summary

Each coefficient was swept across 5 values with all others held at their defaults, replicated over 5 seeds.

### Classification

| Coefficient | Primary effect | Range of coverage | SelAcc (clean) |
|-------------|---------------|-------------------|----------------|
| c_abstain | **Dominant coverage dial** | 0.884 → 0.951 | 0.989 → 0.982 |
| δ (info-utility) | Complementary coverage dial | 0.880 → 0.926 | stable ~0.985 |
| α (uncertainty) | Moderate coverage reduction | 0.922 → 0.891 | stable ~0.985 |
| β (risk) | Mild coverage reduction | 0.925 → 0.895 | stable ~0.985 |
| w_FN (asymmetry) | Shifts FN/FP boundary | 0.913 → 0.904 | stable ~0.985 |

Selective accuracy under PGD at ε = 0.4 is flat across **all** coefficient settings (0.415–0.444), confirming the structural nature of the high-budget failure mode.

### Energy

| Coefficient | J-reduction at ε = 0.4 | Cost overhead | Interpretation |
|-------------|------------------------|---------------|----------------|
| δ = 0.10 → 1.00 | **35.6 % → 72.1 %** | constant 0.31 % | Dominant robustness knob |
| α = 0.50 → 2.00 | 77.3 % → 0.9 % | constant 0.31 % | Robustness–cost-focus trade-off |
| β = 0.10 → 0.80 | 60.8 % → 35.6 % | constant 0.31 % | Secondary uncertainty regulariser |
| γ = 0.20 → 1.20 | 55.1 % → 46.9 % | constant 0.31 % | Secondary risk regulariser |

---

## Operating Points

### Classification

| Operating point | c_abstain | α | δ | Coverage | SelAcc (clean) | SelAcc (PGD 0.4) |
|-----------------|-----------|---|---|----------|----------------|------------------|
| High-precision  | 0.05 | 0.50 | 0.60 | 0.895 | 0.989 | 0.416 |
| **Balanced (default)** | **0.15** | **0.30** | **0.40** | **0.909** | **0.985** | **0.422** |
| High-coverage   | 0.40 | 0.05 | 0.25 | 0.957 | 0.981 | 0.441 |

### Energy

| Operating point | α | β | γ | δ | Cost overhead % | J-reduction % (ε = 0.4) |
|-----------------|---|---|---|---|-----------------|--------------------------|
| Cost-priority   | 2.00 | 0.10 | 0.20 | 0.25 | 0.31 | 3.0 |
| **Balanced (default)** | **1.00** | **0.35** | **0.60** | **0.50** | **0.31** | **51.8** |
| Stability-priority | 0.75 | 0.55 | 0.90 | 0.75 | 0.31 | 65.1 |

---

## Reproducing the Results

### On Google Colab (recommended)

1. Upload `VDS_Variational_Decision_Shield.ipynb` to Colab.
2. Run **Cell 1 (Setup)** — it will mount your Google Drive automatically and write all outputs to `MyDrive/Outputs/VDS/`.
3. Run all cells in order. Total runtime: ~15–18 minutes (main experiments ~7 min, ablation ~8 min on first run; ablation results are cached to Drive on subsequent runs).

### Locally

```bash
pip install scikit-learn numpy pandas matplotlib
# Set output directory (optional)
export VDS_OUTPUT_ROOT=./outputs/VDS
jupyter notebook VDS_Variational_Decision_Shield.ipynb
```

**Requirements:** Python 3.9+, scikit-learn ≥ 1.0, numpy, pandas, matplotlib. No GPU required.

---

## Experimental Protocol

- **Seeds:** 5 independent seeds (11, 23, 47, 89, 173) for all experiments.
- **Reporting:** Mean ± standard deviation throughout; 95 % CI via normal approximation (z = 1.96).
- **Classification attacks:** Random Gaussian (ε ∈ {0.0, 0.1, …, 1.0}) and PGD L₂ (25 steps, step fraction 0.2, radius ε√d in standardised feature space).
- **Energy attacks:** Random FDI (multiplicative Gaussian) and worst-case bilevel FDI (3-point per-axis grid, 27 candidates/timestep, maximises realised step cost on true state).
- **Ablation:** 1D sweeps, 5 values per coefficient, all others held at defaults, 5 seeds each.

---

## Citation

If you use this code or results in your research, please cite:

```bibtex
@article{vds2026,
  title   = {VDS: Variational Decision Shield --- A Cross-Domain
             Decision-Making Framework via Composite Functional Minimisation},
  author  = {[Authors]},
  journal = {[Journal]},
  year    = {2026},
  note    = {Under review}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
