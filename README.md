# AIA4OneHealth — A Machine Learning–Driven One Health System for Managing Zoonotic Diseases

End-to-end pipeline that couples a **Bayesian-calibrated mechanistic Mpox model** with an **ML emulator ensemble** and **NSGA-II multi-objective optimisation** to recommend One Health policy packages across **three AI4PEP partner countries**: Nigeria, the Democratic Republic of the Congo (DRC), and Ethiopia.

The repository accompanies the manuscript *"A Machine Learning–Driven One Health System for Managing Zoonotic Diseases"* and the AI4PEP Mpox project.

---

## Notebooks

| File | Country | Partner tool | Data quality | What it does |
|---|---|---|---|---|
| [`Mpox_OneHealth_Main.ipynb`](Mpox_OneHealth_Main.ipynb) | Nigeria | (published NG study) | Rich OWID history | Original pipeline — calibrated to the Nigeria 2022 Clade IIb wave. |
| [`Congo_OneHealth_Main.ipynb`](Congo_OneHealth_Main.ipynb) | DRC | **AfiaGap** (actively deployed) | OWID + IDSR SEM14/2026 (26 provinces / 517 zones) | DRC counterpart with subnational view. **v2** adds robustness check + multi-budget scan + tier hierarchy diagram + Sobol'-vs-integration clarity. |
| [`Ethiopia_OneHealth_Main.ipynb`](Ethiopia_OneHealth_Main.ipynb) | Ethiopia | **MPSur** (pilot / clinical validation) | OWID only (thin: 48 cases, 22 weeks) + Open-Meteo climate covariates | Prospective framework demonstration with honest wide-CI framing. MPSur team can re-run against their governance-protected internal data. |
| [`Nigeria_Congo_Ethiopia_OneHealth_Combined.ipynb`](Nigeria_Congo_Ethiopia_OneHealth_Combined.ipynb) | All three | — | — | Joint cross-country pipeline — three-way Pareto overlays, three-way Sobol' comparison, three-country policy dashboard. |

All per-country notebooks follow the same ten-section architecture:

1. **Setup** — install dependencies, load OWID data.
2. **EDA** — weekly Mpox epidemic curves (+ IDSR overlay for DRC, + Open-Meteo climate for Ethiopia).
3. **White box** — coupled human–rodent SEIR-V ODE model.
4. **Bayesian calibration** — MCMC (`emcee`) with negative-binomial likelihood, posterior predictive checks.
5. **Black box** — five ML emulators (GP-Matérn, GP-RBF, Random Forest, XGBoost, Neural Net).
6. **Forecasting** — XGBoost lag-feature next-week case forecaster.
7. **Sobol' sensitivity** — total-effect indices over the five One Health levers.
8. **Grey box** — NSGA-II 3-objective Pareto (cases × cost × cross-domain equity).
9. **Uncertainty quantification + ablation** — GP confidence, parameter-posterior propagation, 5-seed × 120-gen robustness check, multi-budget scan, tier-hierarchy diagram.
10. **Dashboard** — one-page policy summary.

---

## Data — every source is real

| Path | Source | Notes |
|---|---|---|
| `owid_mpox.csv` *(auto-downloaded on first run)* | [Our World in Data — Mpox](https://github.com/owid/monkeypox) | Long weekly history for Nigeria, DRC, Ethiopia, Africa. |
| `data/congo/SEM14_2026.xlsx` | DRC Ministry of Health IDSR weekly bulletin | 65 k rows, 26 provinces, 517 health zones, 14 weeks of 2026. |
| `data/congo/BASE_ALERTES_MEDIA.xlsx` | DRC event-based surveillance feed | Media-derived alerts. |
| `data/congo/Rapport_Epidemiologique_S14_2026.pdf` | Narrative weekly report | Comparative S14/2026 summary across diseases. |
| [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) *(live at runtime)* | Public, no auth required | Real historical rainfall / temperature / humidity for Addis Ababa (8.98°N, 38.76°E) — used by the Ethiopia notebook. |

The DRC files were supplied by the in-country partner team (**AfiaGap**, actively deployed in the DRC). The Ethiopia response comes from **MPSur** (pilot / clinical validation). Both responses were collected via the AI4PEP stakeholder questionnaire.

**No synthetic case data anywhere.** If any external data source (OWID or Open-Meteo) is unreachable at runtime, the notebook fails loudly rather than silently substituting fabricated numbers.

### Ethiopia thin-data note

Ethiopia's OWID public feed reports **only 48 confirmed Mpox cases across 22 weeks** (May–October 2025). The Ethiopia notebook proceeds honestly with this signal:
- Reduced MCMC depth (16 walkers × 800 steps) matched to the data density.
- Posterior credible intervals reported at their full (wide) width.
- All Pareto recommendations carry those wide uncertainty bands.
- Framed as a **prospective framework** the MPSur team can re-run against their governance-protected internal community-signal data (reported sensitivity 0.97, AUC 0.97) when permissions allow.

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
| DRC (v2, incl. robustness + multi-budget scan) | ~25–35 min | ~7 min |
| Ethiopia (reduced MCMC, thin data) | ~12–18 min | ~4 min |
| 3-country combined | ~40–55 min | ~10 min |

---

## Running on Google Colab

Upload the `.ipynb` file plus the `data/congo/` folder (for the DRC and 3-country notebooks), then **Runtime → Run all**. The first cell (`%pip install …`) handles every dependency. Open-Meteo climate fetch for Ethiopia works out of the box on Colab.

If your GitHub repo is public, the `!git clone …` shortcut in Colab brings everything in one command.

---

## Architecture

```
                ┌─────────────────────────────┐    ┌─────────────────────────────┐    ┌─────────────────────────────┐
                │   NIGERIA (OWID)            │    │   DRC (OWID + IDSR)         │    │   ETHIOPIA (OWID + climate) │
                └──────────────┬──────────────┘    └──────────────┬──────────────┘    └──────────────┬──────────────┘
                               │                                   │                                   │
                               └──────────────┬────────────────────┴────────────────┬──────────────────┘
                                              ▼                                     ▼
                              ┌───────────────────────────────┐    ┌───────────────────────────────┐
                              │  WHITE BOX — shared           │    │  Bayesian calibration         │
                              │  Coupled human-rodent SEIR-V  │──▶ │  (emcee MCMC per country)     │
                              │  5 levers: ν, η_H, η_E, η_A, α│    │  Neg-Binomial likelihood      │
                              └───────────────────────────────┘    └──────────────┬────────────────┘
                                                                                   │
                                                        ┌──────────────────────────┴───────────────────────────┐
                                                        ▼                                                      ▼
                                    ┌─────────────────────────────┐                     ┌──────────────────────────────────┐
                                    │  BLACK BOX — ML emulator    │                     │  Forecasting head                │
                                    │  ensemble (GP/RF/XGB/NN)    │                     │  (XGBoost lag-features)          │
                                    └──────────────┬──────────────┘                     └──────────────────────────────────┘
                                                   ▼
                                    ┌─────────────────────────────┐
                                    │  Sobol' S_T  |  Integration │
                                    │  tier marginal-effect       │
                                    └──────────────┬──────────────┘
                                                   ▼
                                    ┌─────────────────────────────┐
                                    │  GREY BOX — NSGA-II         │
                                    │  3-objective Pareto:        │
                                    │  cases × cost × equity      │
                                    └──────────────┬──────────────┘
                                                   ▼
                                    ┌─────────────────────────────┐
                                    │  Ablation + robustness      │
                                    │  (5 seeds × 120 gens)       │
                                    │  + multi-budget scan        │
                                    │  + tier hierarchy diagram   │
                                    └──────────────┬──────────────┘
                                                   ▼
                                    ┌─────────────────────────────┐
                                    │  Robust recommendation      │
                                    │  + GP UQ + posterior prop.  │
                                    └─────────────────────────────┘
```

### Five One Health levers (decision variables)

| Symbol | Domain | Meaning | Range (NG / DRC) | Range (Ethiopia) |
|---|---|---|---|---|
| ν | Human | Daily vaccination rate | [0, 0.005] | [0, 0.003] *(narrower — MPSur funding constraint)* |
| η_H | Human | Isolation / case-finding | [0, 1] | [0, 1] |
| η_E | Environment | Bushmeat regulation + PPE (NG/DRC); climate-adaptive interventions (Ethiopia) | [0, 1] | [0, 1] |
| η_A | Animal | Reservoir management | [0, 0.5] | [0, 0.5] *(model-only — MPSur reports no animal-data collection)* |
| α | Human | Awareness / NPI / contact reduction | [0, 0.7] | [0, 0.7] |

### Three optimisation objectives

1. **Cumulative cases** over 1 year (epidemiological burden — minimise)
2. **Programme cost** under country-specific unit-cost weights (minimise)
3. **Domain imbalance** — coefficient-of-variation across human / animal / environment domains (minimise; penalises over-reliance on a single domain)

### Country-specific budget caps

- Nigeria & DRC ablation: **≤10M units** (larger national programme scale)
- Ethiopia ablation: **≤5M units** (matches MPSur's pilot-scale funding constraint per their questionnaire response)

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
- Open-Meteo Historical Weather API: <https://open-meteo.com/en/docs/historical-weather-api>

---

## License & acknowledgements

Research code released for academic use. Please cite the accompanying manuscript when re-using the pipeline.

- **AfiaGap** (DRC) — supplied the DRC IDSR SEM14/2026 bulletin, media alerts, and epidemiological report PDF, plus stakeholder questionnaire response.
- **MPSur** (Ethiopia) — supplied the AI4PEP stakeholder questionnaire response and pilot context. Ethiopia case data used here comes from the public OWID feed; MPSur's internal governance-protected community-signal data is not included in this repository.
- **Nigeria** data sourced from Our World in Data's open Mpox repository.
- **Climate data** sourced from Open-Meteo's public Historical Weather API.
- This is a joint AI4PEP research collaboration across the three partner teams.
