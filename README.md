# Selection on Observables, Audited Against an Experiment

**Estimating the effect of the National Supported Work programme on 1978 earnings from non-experimental data, with the experimental answer as the benchmark**

The National Supported Work (NSW) demonstration randomised disadvantaged workers into a subsidised-employment programme in the mid-1970s. Because it was randomised, the causal effect of the programme on subsequent earnings is *known*, which makes it one of the few settings in which observational estimators can be graded rather than merely compared with one another. Following LaLonde (1986) and Dehejia and Wahba (1999), this notebook takes the experimental treatment group, replaces the experimental controls with a comparison sample drawn from the Current Population Survey (CPS-3), and asks how much of the resulting selection bias can be removed by conditioning on observed covariates.

Everything is implemented from first principles with `numpy`, `pandas`, `scipy`, `statsmodels`, `scikit-learn` and `matplotlib`; no causal-inference package is used. Each estimator is derived before it is coded, and each is paired with an inference procedure that is valid for it.

## Contents of the notebook

| Section | What it does |
|---|---|
| 1 Setup | Seeds, plotting defaults, helper functions (standardised mean differences, balance tables, Love plots, bootstrap) |
| 2 Data | Loads the Dehejia–Wahba experimental sample (185 treated, 260 controls) and the CPS-3 comparison sample (429 men); documents provenance and eligibility-relevant features of the covariates |
| 3 Framework | Potential outcomes, SUTVA, the ATT as target parameter, identification under conditional ignorability and overlap, with proofs of the g-formula and the weighting representation |
| 4 Experimental benchmark | Difference in means with Neyman variance; regression adjustment; Lin (2013) interacted estimator |
| 5 Observational problem | Measures the selection bias exactly (the experimental controls make this possible) and characterises the covariate imbalance that produces it |
| 6 Outcome regression | OLS with a treatment dummy versus g-computation for the ATT; why the two target different quantities (Angrist 1998; Słoczyński 2022) |
| 7 Propensity scores | Four specifications compared on balance, effective sample size and overlap; the role of the zero-earnings indicators; Hájek IPW-ATT with a bootstrap that re-estimates the score; the ATE-weights trap; overlap trimming |
| 8 Matching | 1-NN matching on the logit score with a caliper; Abadie–Imbens (2006) variance versus the (inconsistent) naive bootstrap; comparison-unit reuse; M-sweep with Abadie–Imbens (2011) bias correction |
| 9 Doubly robust estimation | AIPW for the ATT with its influence function; cross-fitted AIPW with random-forest nuisance functions and DML-style aggregation over repeated splits |
| 10 Sensitivity analysis | Cinelli–Hazlett (2020) robustness values and bias contours; VanderWeele–Ding E-values for point estimates and confidence limits |
| 11 Synthesis | Summary table, forest plot, conclusions, limitations, references |

## Headline results (primary specification, ATT on 1978 earnings in USD)

| Estimator | Estimate | SE | SE method | 95% CI covers benchmark |
|---|---:|---:|---|:---:|
| Experimental benchmark (difference in means) | 1,794 | 671 | Neyman | — |
| Naive comparison with CPS-3 | −635 | 677 | Neyman | no |
| OLS with treatment dummy | 866 | 739 | HC1 | yes |
| g-computation (ATT) | 1,170 | 813 | bootstrap | yes |
| IPW (ATT weights) | 1,003 | 846 | bootstrap | yes |
| 1-NN matching, caliper | −1,682 | 1,565 | Abadie–Imbens | no |
| 10-NN matching | 1,062 | 845 | Abadie–Imbens | yes |
| AIPW (doubly robust) | 852 | 837 | bootstrap | yes |
| Cross-fitted AIPW, random forests | 996 | 741 | IF + split dispersion | yes |

The selection bias in the raw comparison is −\$2,429. Conditioning on the ten observed covariates removes most of it: every well-behaved estimator lands between roughly \$850 and \$1,200 with standard errors of \$740–\$880, and all of their intervals cover the experimental benchmark. Two findings are worth emphasising:

* **The estimand matters.** The same propensity model that gives a sensible ATT produces an ATE of about \$225, because ATE weights target the CPS population rather than the programme population. The "IPTW performs badly" conclusion that circulates in some textbook treatments of these data is a consequence of the estimand, not the estimator.
* **One-to-one matching is fragile here.** With 429 comparison men and 185 treated, 1-NN matching uses only 73 distinct controls and reuses one of them nineteen times. The Abadie–Imbens standard error is roughly twice the naive bootstrap standard error, and the point estimate is unstable across specifications. Matching with M = 10 neighbours restores both balance and stability.

The sensitivity analysis shows that the observational estimates are individually fragile to unmeasured confounding (robustness values of a few per cent of the residual variance; E-values around 1.5), which is the honest reading of an effect that is large in dollars but small relative to the dispersion of earnings.

## Repository structure

```
.
├── nsw_selection_on_observables.ipynb   # the notebook (executed; all outputs included)
├── data/
│   ├── nsw_dw_experimental.csv          # Dehejia–Wahba experimental sample, 185 treated + 260 controls
│   └── cps3_controls.csv                # CPS-3 comparison sample, 429 men
├── requirements.txt
└── README.md
```

Both CSV files use the NBER column layout: `treat, age, education, black, hispanic, married, nodegree, re74, re75, re78`. If the `data/` directory is absent, the notebook falls back to downloading the original text files from Rajeev Dehejia's NBER page.

## Running it

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook nsw_selection_on_observables.ipynb
```

Full execution takes about two minutes on a laptop (the cross-fitted random-forest section and the bootstraps dominate). All random draws are seeded (`SEED = 2026`), so the numbers reproduce exactly on a given platform; small differences in the last digit of bootstrap standard errors across platforms are normal.

## Data

* LaLonde, R. J. (1986). Evaluating the econometric evaluations of training programs with experimental data. *American Economic Review* 76(4), 604–620.
* Dehejia, R. H. and Wahba, S. (1999). Causal effects in nonexperimental studies: Reevaluating the evaluation of training programs. *Journal of the American Statistical Association* 94(448), 1053–1062.

The files distributed here are byte-for-byte consistent with the Dehejia–Wahba samples (`nswre74_treated.txt`, `nswre74_control.txt`, `cps3_controls.txt`) hosted at https://users.nber.org/~rdehejia/nswdata2.html. A full reference list for the methods appears at the end of the notebook.
