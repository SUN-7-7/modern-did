# R and Python Command Templates

Use these templates when the analysis is in R or Python, or when Stata results need companion diagnostics.

## Contents

- [R: Bacon Decomposition](#r-bacon-decomposition)
- [R: TwoWayFEWeights](#r-twowayfeweights)
- [R: did_multiplegt_dyn](#r-did_multiplegt_dyn)
- [R: HonestDiD](#r-honestdid)
- [R: Sun-Abraham Via `fixest::sunab`](#r-sun-abraham-via-fixestsunab)
- [Python: Callaway-SantAnna `csdid`](#python-callaway-santanna-csdid)
- [Python: Borusyak-Jaravel-Spiess Imputation Port](#python-borusyak-jaravel-spiess-imputation-port)

## R: Bacon Decomposition

```r
install.packages("bacondecomp")

library(bacondecomp)
df_bacon <- bacon(
  y ~ D,
  data = df,
  id_var = "id",
  time_var = "t"
)

bacon_summary(df_bacon)
```

Plot weights and estimates:

```r
library(ggplot2)

ggplot(df_bacon, aes(x = weight, y = estimate, shape = type)) +
  geom_point() +
  geom_hline(yintercept = 0) +
  theme_minimal()
```

Use this as a diagnostic for a TWFE regression, not as a modern DID estimator.

## R: TwoWayFEWeights

```r
install.packages("TwoWayFEWeights")
library(TwoWayFEWeights)

twowayfeweights(
  df = df,
  Y = "y",
  G = "group",
  T = "t",
  D = "D",
  type = "feTR",
  summary_measures = TRUE
)
```

Use `controls`, `weights`, `other_treatments`, and `path` as needed.

## R: did_multiplegt_dyn

```r
install.packages("DIDmultiplegtDYN")

library(DIDmultiplegtDYN)

out <- did_multiplegt_dyn(
  df = df,
  outcome = "y",
  group = "group",
  time = "t",
  treatment = "D",
  effects = 6,
  placebo = 3,
  cluster = "group",
  graph_off = TRUE
)

summary(out)
out$plot
```

Normalized effects:

```r
out_norm <- did_multiplegt_dyn(
  df = df,
  outcome = "y",
  group = "group",
  time = "t",
  treatment = "D",
  effects = 6,
  placebo = 3,
  normalized = TRUE,
  same_switchers = TRUE,
  effects_equal = TRUE
)
summary(out_norm)
```

On recent Macs, set `Sys.setenv(RGL_USE_NULL = TRUE)` before loading if `rgl` dependencies cause issues.

## R: HonestDiD

```r
install.packages("remotes")
Sys.setenv("R_REMOTES_NO_ERRORS_FROM_WARNINGS" = "true")
remotes::install_github("asheshrambachan/HonestDiD")

library(HonestDiD)
```

After an event-study model that yields `betahat` and `sigma`:

```r
delta_rm_results <- HonestDiD::createSensitivityResults_relativeMagnitudes(
  betahat = betahat,
  sigma = sigma,
  numPrePeriods = 5,
  numPostPeriods = 2,
  Mbarvec = seq(0.5, 2, by = 0.5)
)

original_results <- HonestDiD::constructOriginalCS(
  betahat = betahat,
  sigma = sigma,
  numPrePeriods = 5,
  numPostPeriods = 2
)

HonestDiD::createSensitivityPlot_relativeMagnitudes(
  delta_rm_results,
  original_results
)
```

Smoothness restrictions:

```r
delta_sd_results <- HonestDiD::createSensitivityResults(
  betahat = betahat,
  sigma = sigma,
  numPrePeriods = 5,
  numPostPeriods = 2,
  Mvec = seq(0, 0.05, by = 0.01),
  Delta = "DeltaSD"
)

HonestDiD::createSensitivityPlot(delta_sd_results, original_results)
```

Average over post-treatment periods:

```r
l_vec <- matrix(c(0.5, 0.5), ncol = 1)

delta_rm_avg <- HonestDiD::createSensitivityResults_relativeMagnitudes(
  betahat = betahat,
  sigma = sigma,
  numPrePeriods = 5,
  numPostPeriods = 2,
  l_vec = l_vec,
  Mbarvec = seq(0.5, 2, by = 0.5)
)
```

## R: Sun-Abraham Via `fixest::sunab`

The HonestDiD README shows extracting event-study coefficients from `fixest` models using `sunab()`. After extraction, pass `beta` and `sigma` to the HonestDiD functions above.

```r
library(fixest)

mod <- feols(
  y ~ sunab(first_treat, t) | id + t,
  data = df,
  cluster = "id"
)

iplot(mod)
```

Make sure extracted coefficients exclude the normalized reference period and are ordered with pre-periods first if using `numPrePeriods`.

## Python: Callaway-Sant'Anna `csdid`

```bash
pip install csdid
pip install git+https://github.com/d2cml-ai/DRDID
```

```python
import pandas as pd
from csdid.att_gt import ATTgt

out = ATTgt(
    yname="y",
    gname="first_treat",
    idname="id",
    tname="t",
    xformla="y ~ 1",
    data=df,
).fit(est_method="dr")

out.summ_attgt().summary2
out.aggte(typec="dynamic")
out.plot_aggte()
```

Use `typec="group"` for group aggregation and `typec="calendar"` for calendar-time aggregation.

## Python: Borusyak-Jaravel-Spiess Imputation Port

```bash
pip install didimpute
```

```python
from didimpute import DidImputation

est = DidImputation(
    y="Y",
    id="i",
    time="t",
    Ei="Ei",
    horizons=(-1, 1),
    minN=1
).fit(df)

print(est.summary())
```

This is a Python port. For Stata replication of the original package, prefer `ssc install did_imputation`.
