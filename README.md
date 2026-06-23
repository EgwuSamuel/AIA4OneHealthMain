# AIA4OneHealth — A Machine Learning–Driven One Health System for Managing Zoonotic Diseases

End-to-end pipeline that couples a **Bayesian-calibrated mechanistic Mpox model** with an **ML emulator ensemble** and **NSGA-II multi-objective optimisation** to recommend One Health policy packages for Nigeria and the Democratic Republic of the Congo (DRC).

The repository accompanies the manuscript *"A Machine Learning–Driven One Health System for Managing Zoonotic Diseases"* and the AI4PEP Mpox project.

---

## Notebooks

| File | Country | What it does |
|---|---|---|
| [`Mpox_OneHealth_Main.ipynb`](Mpox_OneHealth_Main.ipynb) | Nigeria | Original pipeline — calibrated to the Nigeria 2022 Clade IIb wave (OWID). |
| [`Congo_OneHealth_Main.ipynb`](Congo_OneHealth_Main.ipynb) | DRC | DRC counterpart — calibrated to the 2024–2026 wave (OWID + IDSR SEM14/2026). Adds a subnational province-level view. |
| [`Nigeria_Congo_OneHealth_Combined.ipynb`](Nigeria_Congo_OneHealth_Combined.ipynb) | Both | Joint cross-country pipeline that runs both countries side-by-side and produces comparative Pareto fronts, Sobol' indices, and policy dashboards. |

All three notebooks follow the same ten-section architecture:

1. **Setup** — install dependencies, load OWID data.
2. **EDA** — weekly Mpox epidemic curves (+ IDSR overlay for DRC).
3. **White box** — coupled human–rodent SEIR-V ODE model.
4. **Bayesian calibration** — MCMC (`emcee`) with negative-binomial likelihood, posterior predictive checks.
5. **Black box** — five ML emulators (GP-Matérn, GP-RBF, Random Forest, XGBoost, Neural Net).
6. **Forecasting** — XGBoost lag-feature next-week case forecaster.
7. **Sobol' sensitivity** — total-effect indices over the five One Health levers.
8. **Grey box** — NSGA-II 3-objective Pareto (cases × cost × cross-domain equity).
9. **Uncertainty quantification** — GP confidence + parameter-posterior propagation.
10. **Ablation + dashboard** — value of integrated One Health, one-page policy summary.

---

## Data

| Path | Source | Notes |
|---|---|---|
| `owid_mpox.csv` *(auto-downloaded on first run)* | [Our World in Data — Mpox](https://github.com/owid/monkeypox) | Long weekly history for Nigeria, DRC, and Africa-aggregate. |
| `data/congo/SEM14_2026.xlsx` | DRC Ministry of Health IDSR weekly bulletin | 65 k rows, 26 provinces, 517 health zones, 14 weeks of 2026. |
| `data/congo/BASE_ALERTES_MEDIA.xlsx` | Event-based surveillance feed | Media-derived alerts (DRC). |
| `data/congo/Rapport_Epidemiologique_S14_2026.pdf` | Narrative weekly report | Comparative S14/2026 summary across diseases. |

The DRC files were supplied by the in-country partner team (**AfiaGap**, actively deployed in the DRC) — see `Response 2.docx` in the source folder for the stakeholder questionnaire response.

---

## Running the notebooks locally

### 1. Install Python (3.11 recommended)

If you're on Python 3.14 some libraries (`pymoo`, `xgboost`, `emcee`) may lack prebuilt wheels. Use Python 3.11 to avoid build errors:

```cmd
py -3.11 -m venv .venv
.venv\Scripts\activate
```

### 2. Install dependencies

```cmd
python -m pip install jupyterlab pymoo SALib xgboost emcee corner openpyxl seaborn scikit-learn scipy pandas matplotlib
```

### 3. Launch JupyterLab

```cmd
python -m jupyterlab
```

A browser tab opens at `http://localhost:8888/lab`. Double-click any notebook, then **Run → Run All Cells**.

### 4. Quick first pass (skip MCMC, ~3 min)

To validate the environment quickly, in the MCMC cell (Section 3) set:

```python
RUN_MCMC = False
```

The pipeline falls back to the L-BFGS-B point estimate. Flip it back to `True` once everything runs.

### Runtime expectations (laptop, no GPU)

| Notebook | With MCMC | Without MCMC |
|---|---|---|
| Nigeria | ~10–15 min | ~3 min |
| DRC | ~15–25 min | ~4 min |
| Combined | ~25–35 min | ~6 min |

---

## Running on Google Colab

Upload the `.ipynb` file plus the `data/congo/` folder, then **Runtime → Run all**. The first cell (`%pip install …`) handles every dependency.

---

## Architecture

```
                ┌──────────────────────────┐
                │  OWID + IDSR weekly data │
                └────────────┬─────────────┘
                             │
              ┌──────────────▼──────────────┐
              │ Coupled Human–Rodent ODE    │  ← WHITE BOX
              │   (SEIR-V + reservoir)      │
              └──────────────┬──────────────┘
                             │
              ┌──────────────▼──────────────┐
              │ Bayesian calibration (emcee)│
              │   8-D posterior, NB likelih.│
              └──────────────┬──────────────┘
                             │
        ┌────────────────────┴────────────────────┐
        │                                         │
  ┌─────▼──────────┐                    ┌─────────▼──────────┐
  │ ML emulator    │  ← BLACK BOX       │  Forecasting head  │
  │  (5 families)  │                    │   (XGBoost lags)   │
  └─────┬──────────┘                    └────────────────────┘
        │
  ┌─────▼─────────────────────────────┐
  │ NSGA-II 3-objective Pareto        │  ← GREY BOX
  │   cases × cost × equity           │
  └─────┬─────────────────────────────┘
        │
  ┌─────▼─────────────────────────────┐
  │ Robust recommendation             │
  │  + GP UQ + posterior propagation  │
  └───────────────────────────────────┘
```

### Five One Health levers (decision variables)

| Symbol | Domain | Meaning | Range |
|---|---|---|---|
| ν | Human | Daily vaccination rate | [0, 0.005] |
| η_H | Human | Isolation / case-finding | [0, 1] |
| η_E | Environment | Bushmeat regulation + PPE | [0, 1] |
| η_A | Animal | Reservoir management | [0, 0.5] |
| α | Human | Awareness / NPI / contact reduction | [0, 0.7] |

### Three optimisation objectives

1. **Cumulative cases** over 1 year (epidemiological burden — minimise)
2. **Programme cost** under country-specific unit-cost weights (minimise)
3. **Domain imbalance** — coefficient-of-variation across human / animal / environment domains (minimise; penalises over-reliance on a single domain)

---

## Citations

- Yinka-Ogunleye A, et al. (2019). *Outbreak of human monkeypox in Nigeria 2017-18*. **Lancet Infectious Diseases**.
- Kibungu EM, et al. (2024). *Clade I-Associated Mpox Cases Associated with Sexual Contact, DRC*. **Emerging Infectious Diseases**.
- WHO AFRO. *Mpox in the African Region — Situation reports 2024–2026*.
- Zhao Z, Wang L, Bergquist R, et al. (2025). *Crafting an innovative one health-aligned machine learning framework for neglected tropical diseases elimination*. **Science in One Health**.
- Doty JB, et al. (2023). *Orthopoxvirus Infections in Rodents, Nigeria, 2018–2019*. **Emerging Infectious Diseases**.
- Madubueze CE, et al. (2022). *The transmission dynamics of the monkeypox virus in the presence of environmental transmission*. **Frontiers in Applied Mathematics and Statistics**.
- Deb K, Pratap A, Agarwal S, Meyarivan T (2002). *NSGA-II*. **IEEE Trans. Evol. Comput.**
- Foreman-Mackey D, Hogg DW, Lang D, Goodman J (2013). *emcee: The MCMC Hammer*. **PASP** 125, 306–312.
- Rasmussen CE, Williams CKI (2006). *Gaussian Processes for Machine Learning*. MIT Press.
- Our World in Data — Mpox: <https://ourworldindata.org/mpox>

---

## License & acknowledgements

Research code released for academic use. Please cite the accompanying manuscript when re-using the pipeline.

DRC data and field context provided by the **AfiaGap** team (DRC). Nigeria data sourced from Our World in Data's open Mpox repository.
