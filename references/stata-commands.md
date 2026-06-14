# Stata Command Templates

Use these templates as starting points. Replace variable names and check `help` for installed package versions.

## Contents

- [Installation](#installation)
- [Callaway-SantAnna: `csdid`](#callaway-santanna-csdid)
- [Sun-Abraham: `eventstudyinteract`](#sun-abraham-eventstudyinteract)
- [Borusyak-Jaravel-Spiess: `did_imputation`](#borusyak-jaravel-spiess-did_imputation)
- [de Chaisemartin-DHaultfoeuille: `did_multiplegt_dyn`](#de-chaisemartin-dhaultfoeuille-did_multiplegt_dyn)
- [Older `did_multiplegt` Modes](#older-did_multiplegt-modes)
- [TWFE Diagnostics](#twfe-diagnostics)
- [HonestDiD / Rambachan-Roth: `honestdid`](#honestdid--rambachan-roth-honestdid)

## Installation

```stata
* Common dependencies
ssc install ftools, replace
ssc install reghdfe, replace
ssc install avar, replace

* Callaway-Sant'Anna and related plotting
ssc install drdid, replace
ssc install csdid, replace

* Borusyak-Jaravel-Spiess imputation and event-study plotting
ssc install did_imputation, replace
ssc install event_plot, replace

* de Chaisemartin-D'Haultfoeuille estimators and TWFE weights
ssc install did_multiplegt, replace
ssc install did_multiplegt_dyn, replace
ssc install twowayfeweights, replace

* HonestDiD from SSC, then plugin check
ssc install honestdid, replace
honestdid _plugin_check
```

For the latest Stata `honestdid` GitHub build:

```stata
local github https://raw.githubusercontent.com
net install honestdid, from(`github'/mcaceresb/stata-honestdid/main) replace
honestdid _plugin_check
```

For `eventstudyinteract`, the README uses Haghish's `github` installer:

```stata
net install github, from("https://haghish.github.io/github/")
github install lsun20/eventstudyinteract
reghdfe, compile
```

## Callaway-Sant'Anna: `csdid`

Use when `gvar` is the first treatment period and never-treated groups are coded `0`.

```stata
csdid y x1 x2, ivar(id) time(t) gvar(first_treat) method(dripw) notyet ///
    cluster(id) wboot rseed(12345) agg(event)

estat event, window(-4 6)
estat pretrend
csdid_plot, style(rcap) ytitle("ATT") xtitle("Event time")
```

Useful variations:

```stata
* Group-time ATTs, no aggregation
csdid y x1 x2, ivar(id) time(t) gvar(first_treat) method(dripw)

* Simple, group, calendar, and event aggregations after estimation
estat simple
estat group
estat calendar
estat event, window(-3 5)

* Save RIFs for later wild-bootstrap aggregation
csdid y x1 x2, ivar(id) time(t) gvar(first_treat) saverif(_rif_) replace
use _rif_, clear
csdid_stats event, wboot rseed(12345)
```

Notes:

- `notyet` requests never-treated plus not-yet-treated controls; default is never-treated only unless none exist.
- `method(dripw)` is the default doubly robust stabilized IPW method; `drimp`, `reg`, `stdipw`, and `ipw` are alternatives.
- Always-treated units are excluded.
- For unbalanced panels, inspect warnings and cohort support.

## Sun-Abraham: `eventstudyinteract`

Create relative-time indicators manually. Leave one event time out as the reference period, usually `-1`.

```stata
gen rel_time = t - first_treat
gen never_treated = missing(first_treat)

forvalues k = 5(-1)2 {
    gen lead`k' = rel_time == -`k'
    replace lead`k' = 0 if never_treated
}
forvalues k = 0/6 {
    gen lag`k' = rel_time == `k'
    replace lag`k' = 0 if never_treated
}

eventstudyinteract y lead5-lead2 lag0-lag6, ///
    cohort(first_treat) control_cohort(never_treated) ///
    absorb(i.id i.t) vce(cluster id)

matrix b = e(b_iw)
matrix V = e(V_iw)
ereturn post b V
test lead5 lead4 lead3 lead2
```

Notes:

- `cohort()` is the initial treatment timing and is missing for never-treated units.
- `control_cohort()` is a binary indicator for never-treated or last-treated controls.
- If using last-treated controls, exclude periods after the last-treated cohort becomes treated.
- The README notes Sun-Abraham validity is established for balanced panel data without covariates; be explicit if using covariates or unbalanced data.

## Borusyak-Jaravel-Spiess: `did_imputation`

Use when `Ei` is the first treatment date and is missing for never-treated units.

```stata
did_imputation y id t Ei, horizons(0/6) pretrends(5) autosample ///
    cluster(id) leaveout

event_plot, default_look graph_opt(xtitle("Event time") ytitle("Effect"))
```

Useful variations:

```stata
* All available nonnegative horizons
did_imputation y id t Ei, allhorizons autosample cluster(id)

* Balanced horizon composition
did_imputation y id t Ei, horizons(0/6) hbalance cluster(id)

* Flexible untreated-outcome model
did_imputation y id t Ei, fe(id state#t) controls(x1 x2) cluster(id)

* Heterogeneity by subgroup
did_imputation y id t Ei, horizons(0/6) hetby(group_var) cluster(id)

* Repeated cross-sections
did_imputation y person_id t Ei, fe(region t) cluster(region)

* Triple differences
did_imputation y unit_group t E_unit_group, fe(unit_group unit#t group#t) cluster(unit)
```

Notes:

- The command estimates untreated potential outcomes on untreated observations, imputes treatment effects for treated observations, then averages.
- `autosample` drops observations whose treatment effects cannot be imputed; inspect the generated `cannot_impute` logic and resulting estimand.
- `pretrends(k)` tests leads in a separate regression and does not change post-treatment estimates.
- `leaveout` is recommended when cohorts are small.

## de Chaisemartin-D'Haultfoeuille: `did_multiplegt_dyn`

Use when passing the current treatment variable `D`.

```stata
did_multiplegt_dyn y group t D, effects(6) placebo(3) cluster(group) graph_off
```

Useful variations:

```stata
* Normalized effects per unit of treatment and common switcher set
did_multiplegt_dyn y group t D, effects(6) placebo(3) normalized ///
    same_switchers effects_equal cluster(group)

* Treatment path description
did_multiplegt_dyn y group t D, effects(6) design(0.5, "console") graph_off

* Nonparametric trend groups or group-specific linear trends
did_multiplegt_dyn y group t D, effects(5) trends_nonparam(state) cluster(state)
did_multiplegt_dyn y group t D, effects(5) trends_lin cluster(group)

* Switchers-in and switchers-out separately
did_multiplegt_dyn y group t D, effects(5) switchers(in)
did_multiplegt_dyn y group t D, effects(5) switchers(out)
```

The wrapper command can route to the dynamic estimator:

```stata
did_multiplegt (dyn) y group t D, effects(6) placebo(3) cluster(group) graph_off
```

Notes:

- Group and time fixed effects are controlled internally; do not add them as controls.
- `placebo(k)` tests pre-switch comparisons and cannot exceed the requested number of effects.
- Use `same_switchers` or `same_switchers_pl` when horizon composition matters.
- Use `normalized` for per-unit treatment-dose interpretations.

## Older `did_multiplegt` Modes

```stata
* Older DID_M estimator
did_multiplegt (old) y group t D, breps(100) cluster(group)

* Static/stayer estimator
did_multiplegt (stat) y group t D, exact_match
```

Prefer `(dyn)` for new dynamic work unless reproducing a prior analysis.

## TWFE Diagnostics

Use `twowayfeweights` to inspect TWFE weights and robustness to heterogeneous effects:

```stata
twowayfeweights y group t D, type(feTR) summary_measures ///
    path("twfe_weights.dta")
```

Options to consider:

```stata
twowayfeweights y group t D, type(feTR) controls(x1 x2) summary_measures
twowayfeweights y group t D, type(feTR) other_treatments(D2) path("weights.dta")
```

The main `type()` choices are `feTR`, `feS`, `fdTR`, and `fdS`.

## HonestDiD / Rambachan-Roth: `honestdid`

After a conventional event-study regression:

```stata
reghdfe y b2013.Dyear, absorb(id t) cluster(id) noconstant

honestdid, pre(1/5) post(7/8) mvec(0.5(0.5)2)
honestdid, pre(1/5) post(6/7) mvec(0(0.01)0.05) delta(sd) omit coefplot
```

For an average of post-treatment effects:

```stata
matrix l_vec = 0.5 \ 0.5
honestdid, l_vec(l_vec) pre(1/5) post(6/7) mvec(0(0.5)2) omit coefplot
```

After `csdid`:

```stata
csdid y, time(t) ivar(id) gvar(first_treat) long2 notyet
estat event, window(-4 5) estore(csdid)
estimates restore csdid
honestdid, pre(3/6) post(7/12) mvec(0.5(0.5)2) coefplot
```

With custom coefficient and covariance matrices:

```stata
honestdid, pre(1/5) post(6/10) b(b_matrix) vcov(V_matrix) mvec(0.5(0.5)2)
```

Notes:

- `pre()` and `post()` refer to coefficient positions, not event-time labels.
- `mvec()` is `Mbar` for relative-magnitude restrictions by default.
- `delta(sd)` switches to smoothness restrictions using `M`.
- If plugin loading fails, compile the OSQP/ECOS plugin following the `stata-honestdid` README.
