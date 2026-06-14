# Modern DID Workflow

## Design Audit

Record these fields before writing code:

- `outcome`: outcome variable and scale.
- `id`: panel unit or repeated cross-section individual id.
- `group`: aggregate treatment group if different from `id`.
- `time`: calendar period, evenly spaced if required.
- `first_treat`: first treatment date; missing or zero for never-treated depending on command.
- `D`: current treatment level; binary absorbing, non-binary, or reversible.
- `controls`: whether controls are pre-treatment, time-varying, group-level, or interacted with time.
- `cluster`: clustering level, usually at or above treatment assignment level.
- `support`: available pre-periods and post-periods by cohort.

## Estimator Selection

Use `csdid` when:

- Treatment is binary and absorbing.
- The target is ATT(g,t), event-time ATT, group, calendar, or simple aggregations.
- Comparison should be never-treated and/or not-yet-treated units.
- You need Callaway-Sant'Anna style aggregation and pre-trend tests.

Use `did_imputation` when:

- Treatment is binary, staggered, and absorbing.
- You want the Borusyak-Jaravel-Spiess imputation estimator.
- You need flexible fixed effects in the untreated-outcome model, horizon-specific effects, pre-trend tests, `autosample`, `hbalance`, `hetby`, `project`, or `event_plot`.
- You are comfortable with an estimator where pre-trend testing is separate from post-treatment effect estimation.

Use `eventstudyinteract` when:

- The target is Sun-Abraham interaction-weighted event-study estimates.
- You can create relative-time indicators manually.
- Cohorts are well-defined and the control cohort is either never-treated or last-treated.
- You want IW estimates in `e(b_iw)` and pointwise variance in `e(V_iw)`.

Use `did_multiplegt_dyn` when:

- Treatment may be non-binary, continuous, non-absorbing, reversible, or switch multiple times.
- Lagged treatments may affect outcomes.
- You need placebos, normalized effects, same-switcher restrictions, treatment path descriptions, or heterogeneous effects by subgroup/path.
- You need de Chaisemartin-D'Haultfoeuille dynamic estimators for first-switch or complex treatment paths.

Use `did_multiplegt` wrapper when:

- You want `did_multiplegt (dyn)`, `(stat)`, `(had)`, or `(old)` through one Stata command.
- For new dynamic event-study work, route to `(dyn)` unless reproducing older `DID_M` results.

Use TWFE diagnostics when:

- A conventional TWFE estimate appears in the paper/code.
- Cohorts adopt at different times.
- Effects plausibly vary by group or event time.
- You need Goodman-Bacon decomposition or de Chaisemartin-D'Haultfoeuille weight diagnostics before interpreting TWFE.

Use HonestDiD when:

- You already have event-study coefficients and a covariance matrix.
- The question is how robust conclusions are to violations of parallel trends.
- You can define pre-treatment and post-treatment coefficient indices and choose relative-magnitude (`Mbar`) or smoothness (`M`) restrictions.

## Recommended Analysis Sequence

1. Create cohort and event-time variables.
2. Tabulate cohort-by-time support and identify always-treated or never-treated groups.
3. Run a conventional TWFE event study only as a benchmark.
4. Run Bacon decomposition and/or TWFE weight diagnostics if TWFE is discussed.
5. Run a primary modern estimator matching the design.
6. Run a second estimator when its assumptions match the same estimand closely enough.
7. Run pre-trend/placebo tests:
   - `csdid`: `estat pretrend` or `estat pretrend, window(a b)`.
   - `did_imputation`: `pretrends(k)`.
   - `did_multiplegt_dyn`: `placebo(k)`.
   - `eventstudyinteract`: post `e(b_iw)`/`e(V_iw)` then test selected lead coefficients.
8. Plot event studies with clear reference period and CI type.
9. Run HonestDiD sensitivity analysis when the event-study vector is central to the claim.
10. Report exact comparison group, clustering, estimand, software command, and package version/source.

## Interpretation Checklist

- State whether estimates are ATT(g,t), dynamic averages, normalized treatment-dose effects, or scalar sensitivity intervals.
- Explain whether controls are used for identification, precision, or untreated-outcome imputation.
- Explain event-time composition changes if the set of contributing cohorts changes across horizons.
- Use `hbalance`, `same_switchers`, or explicit support tables when horizon composition is a concern.
- Treat null pre-trend tests as low-power diagnostics, not proof of parallel trends.
- For HonestDiD, report the breakdown value or the largest `Mbar`/`M` under which the main qualitative conclusion survives.

## Common Data Conversions

- `csdid`: never-treated groups are coded `0` in `gvar`.
- `did_imputation`: never-treated units have missing `Ei`; the treatment is implied by `t >= Ei`.
- `eventstudyinteract`: never-treated cohorts are missing in `cohort()`, while `control_cohort()` is a binary indicator.
- `did_multiplegt_dyn`: pass the current treatment level `D`, not the first treatment date.
- `HonestDiD`/`honestdid`: pass the ordered event-study coefficients and covariance matrix; define which entries are pre and post.
