# README-Derived Package Notes

These notes summarize the README/help files read while building the skill. Use the upstream repositories as the source of truth for updates.

## Sources Read

- `friosavila/csdid_drdid` README: https://github.com/friosavila/csdid_drdid
- `friosavila/stpackages/csdid` Stata help files: https://github.com/friosavila/stpackages/tree/main/csdid
- `d2cml-ai/csdid` README: https://github.com/d2cml-ai/csdid
- `lsun20/EventStudyInteract` README and Stata help: https://github.com/lsun20/EventStudyInteract
- `borusyak/did_imputation` README and Stata help: https://github.com/borusyak/did_imputation
- `Credible-Answers/did_multiplegt` README: https://github.com/Credible-Answers/did_multiplegt
- `Credible-Answers/did_multiplegt_dyn` README: https://github.com/Credible-Answers/did_multiplegt_dyn
- `evanjflack/bacondecomp` README: https://github.com/evanjflack/bacondecomp
- `Credible-Answers/twowayfeweights` README: https://github.com/Credible-Answers/twowayfeweights
- `asheshrambachan/HonestDiD` README: https://github.com/asheshrambachan/HonestDiD
- `mcaceresb/stata-honestdid` README: https://github.com/mcaceresb/stata-honestdid

## Key README/Help Takeaways

### `csdid`

- Implements Callaway-Sant'Anna multiple-period DID through ATT(g,t) building blocks and aggregations.
- Current Stata files live under `friosavila/stpackages`; older README points there.
- Syntax uses `csdid depvar controls, ivar(id) time(t) gvar(first_treat)`.
- Never-treated groups are coded `0` in `gvar`; always-treated units are excluded.
- Aggregations include `simple`, `group`, `calendar`, `event`, and `attgt`.
- Postestimation includes `estat pretrend`, `estat event`, `csdid_stats`, `csdid_plot`, and RIF workflows.

### Python `csdid`

- Implements Callaway-Sant'Anna in Python, based on the original R `did` package.
- Main class is `ATTgt`; aggregations use `aggte(typec="dynamic"|"group"|"calendar")`.
- Supports plotting via `plot_attgt()` and `plot_aggte()`.

### `eventstudyinteract`

- Implements Sun-Abraham interaction-weighted event-study estimates.
- Requires manually created relative-time indicators.
- Requires `cohort()`, `control_cohort()`, and `absorb()`.
- Stores IW estimates in `e(b_iw)` and pointwise covariance in `e(V_iw)`.
- README/help emphasize never-treated or last-treated control cohorts and note the original validity result is for balanced panels without covariates.

### `did_imputation`

- Stata package for Borusyak-Jaravel-Spiess imputation DID.
- Syntax: `did_imputation Y i t Ei`, where `Ei` is the treatment date and missing means never-treated.
- Treatment is implied as `D = 1[t >= Ei]`.
- Supports `horizons()`, `allhorizons`, `hbalance`, `pretrends()`, `autosample`, `leaveout`, `fe()`, `controls()`, `hetby()`, `project()`, repeated cross-sections, and triple differences.
- Companion `event_plot` plots estimates from `did_imputation`, `did_multiplegt`, `csdid`, `eventstudyinteract`, and conventional OLS.
- Local Stata confirmed `ssc describe did_imputation` and `ssc describe event_plot` succeed.

### `did_multiplegt` and `did_multiplegt_dyn`

- `did_multiplegt` wraps multiple de Chaisemartin-D'Haultfoeuille estimators through modes `(dyn)`, `(stat)`, `(had)`, and `(old)`.
- README recommends `(dyn)` for new dynamic event-study work.
- `did_multiplegt_dyn` handles binary staggered treatment but also non-binary, continuous, non-absorbing, reversible, and multiple-switch treatment paths.
- Core syntax: `did_multiplegt_dyn Y G T D, effects(#) placebo(#)`.
- Options include `normalized`, `normalized_weights`, `effects_equal`, `design()`, `trends_lin`, `trends_nonparam()`, `same_switchers`, `same_switchers_pl`, `switchers()`, `by()`, `predict_het()`, `save_results()`, and `graph_off`.
- Group and time fixed effects are internally handled; they should not be passed as ordinary controls.

### `bacondecomp`

- R package for Goodman-Bacon decomposition with treatment timing variation.
- Main function is `bacon()`, returning all 2x2 DID estimates and weights.
- `bacon_summary()` summarizes comparison types and average estimates.

### `twowayfeweights`

- R and Stata package for weights attached to TWFE regressions and robustness summaries from de Chaisemartin-D'Haultfoeuille.
- Stata syntax: `twowayfeweights Y G T D [D0], type(string)`.
- R syntax: `twowayfeweights(df, Y, G, T, D, type = ...)`.
- Main types are `feTR`, `feS`, `fdTR`, and `fdS`.
- `summary_measures` reports diagnostics about robustness to heterogeneous treatment effects.

### `HonestDiD` / `honestdid`

- Implements Rambachan-Roth robust inference and sensitivity analysis for DID/event-study designs.
- Relative-magnitude restrictions use `Mbar`; smoothness restrictions use `M`.
- R functions include `createSensitivityResults_relativeMagnitudes()`, `createSensitivityResults()`, `constructOriginalCS()`, and plotting helpers.
- Stata `honestdid` can use the active estimation results or explicit `b()` and `vcov()` matrices.
- Stata options include `pre()`, `post()`, `numpre()`, `mvec()`, `delta(sd)`, `l_vec()`, `coefplot`, `cached`, and `parallel()`.
- Stata README demonstrates compatibility with `csdid`, `did_multiplegt`, and `jwdid` if a coefficient vector and covariance matrix are available.

## Caveats for Future Agents

- Package READMEs may lag SSC/CRAN/GitHub releases; verify syntax with `help`, `?function`, or package docs when running live code.
- `did_multiplegt_dyn` README and repository ownership have moved under `Credible-Answers`, while older docs may use `chaisemartinPackages` links.
- Use exact event-time labels and coefficient ordering when bridging estimators to `honestdid`.
- Keep user-facing research claims separate from software diagnostics: pre-trend tests and weight decompositions diagnose assumptions; they do not prove identification.
