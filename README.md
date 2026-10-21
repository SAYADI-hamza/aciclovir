# Aciclovir and valaciclovir: population pharmacokinetic models

Population pharmacokinetic (popPK) models of aciclovir, administered intravenously or orally, or as its oral
prodrug valaciclovir, coded with [mrgsolve](https://mrgsolve.org) and compatible with
[mapbayr](https://github.com/FelicienLL/mapbayr) for MAP Bayesian estimation and model-informed
precision dosing.

## Structure

```
<population>/
├── aciclovir_IV/models/ACV_<FirstAuthor>[_<year>[a|b]].Rmd      intravenous aciclovir
├── aciclovir_VO/models/ACV_<FirstAuthor>[_<year>[a|b]].Rmd      oral aciclovir
└── valaciclovir_VO/models/VACV_<FirstAuthor>[_<year>[a|b]].Rmd  oral valaciclovir
```

| Folder | Population | Route | Prefix | Models |
|---|---|---|---|---|
| `adulte/aciclovir_IV/` | Adults | Intravenous aciclovir | `ACV_` | 1 |
| `adulte/valaciclovir_VO/` | Adults | Oral valaciclovir | `VACV_` | 1 |
| `pediatrie/aciclovir_IV/` | Children and neonates | Intravenous aciclovir | `ACV_` | 5 |
| `pediatrie/aciclovir_VO/` | Children and neonates | Oral aciclovir | `ACV_` | 2 |
| `pediatrie/valaciclovir_VO/` | Children | Oral valaciclovir | `VACV_` | 2 |

When an article describes several routes (or both adults and children), each folder contains the version of
the model for its route and population; the files are named after the same article. The year is added when an author has several articles in the
repository, with a letter (`a`, `b`, in order of publication) for the same year.

## Content of a model file

Each R Markdown file contains:

- the reference of the source article (DOI or journal citation);
- the **model type**: parametric population pharmacokinetic model (nonlinear mixed-effects), nonparametric population pharmacokinetic model (NPAG, with a caution on the parametric approximation) or non-compartmental analysis, with the estimation software and a caution when relevant;
- a **Notes** section describing the modelling choices; each ambiguity of the article is reported as `**Ambiguity:** ... **Choice:** ...` and the corresponding parameter is annotated `(ambiguous, see Notes)`;
- the mrgsolve model (`[PROB]` description, parameters, covariates, compartments, random effects, residual error);
- a simulation example reproducing typical dosing regimens of the article.

## Conventions

- `[SET] end = 24, delta = 0.1`; doses in mg, volumes in L, concentrations in mg/L (total plasma or serum aciclovir).
- Covariates are declared with `@covariates` and are directly measurable patient data, with the same names, units and codes in all models:
  `AGE` (years), `WT` (kg, total body weight), `HT` (cm), `SCR` (serum creatinine, umol/L), `SEX` (1 = male, 0 = female),
  `OCC` (occasion 1 or 2), binary covariates (1 = present, 0 = absent); in children also `PNA` (postnatal age, days) and
  `GA` (gestational age, weeks).
- Derived quantities (creatinine clearance or eGFR with the exact equation of the article: Schwartz with its k constants,
  Counahan-Barratt...; body surface area; postmenstrual or postconceptional age) are computed in the model and captured in the output.
- Default covariate values are the medians of the study population; reference values of the covariate equations are constants in `[PARAM]`.
- Valaciclovir doses (`amt`) are always entered in mg of valaciclovir (tablet strength); when the article coded doses as
  aciclovir equivalents, the molar conversion (x 225.2/324.3 = 0.694, parameter `MWR`) is applied inside the model.
- Random effects follow the mapbayr format: each individual parameter uses `ETAn + ETA(n)`.
- Variability reported as %CV is converted with omega^2 = ln(1 + CV^2); omegas or sigmas reported as SD are squared; reported variances are used directly.
- Between-occasion variability is implemented with two paired occasions (covariate `OCC`).
- Residual error: one proportional/additive sigma pair per observed compartment (`DV = IPRED*(1 + EPS(1)) + EPS(2)`); other error models (power, exponential) are approximated by this pair and the approximation is described in the Notes; when the residual error is not published (e.g. Pmetrics), a 20% proportional error is assumed.
- Intravenous doses are infusions into the central compartment (`rate` in the data, actual infusion duration, 1 h when not reported); oral doses go to the depot compartment.

## Requirements

R with `mrgsolve`, `mapbayr` (0.10.2), `dplyr`, `tidyr` and `ggplot2`.
