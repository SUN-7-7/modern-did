---
name: modern-did
description: Modern difference-in-differences workflow guidance for staggered adoption, event studies, heterogeneous treatment effects, TWFE diagnostics, and sensitivity analysis. Use when working with csdid, eventstudyinteract, did_imputation, did_multiplegt, did_multiplegt_dyn, bacondecomp, twowayfeweights, HonestDiD, honestdid, Goodman-Bacon decomposition, Callaway-Sant'Anna, Sun-Abraham, Borusyak-Jaravel-Spiess, de Chaisemartin-D'Haultfoeuille, Rambachan-Roth, or modern DID code in Stata, R, or Python.
---

# Modern DID

Use this skill to plan, code, diagnose, and explain modern DID analyses for staggered treatment timing and heterogeneous treatment effects.

## Core Workflow

1. Identify the design before choosing software: binary absorbing treatment, multi-valued treatment, reversible treatment, repeated cross-sections, triple differences, or event-study-only inference.
2. Define the estimand explicitly: group-time ATT, dynamic/event-time effects, calendar effects, overall ATT, path-specific effects, or sensitivity bounds.
3. Run TWFE only as a benchmark/diagnostic. For staggered adoption with heterogeneous effects, do not treat TWFE as the main estimator without decomposition or weight diagnostics.
4. Choose a primary modern estimator and at least one triangulation check when feasible.
5. Report event-time support, cohort support, comparison groups, pre-trend/placebo results, clustering, and whether confidence intervals are pointwise, simultaneous, wild-bootstrap, or HonestDiD robust intervals.

## Read References

- Read `references/workflow.md` before designing an analysis plan, choosing an estimator, or interpreting results.
- Read `references/stata-commands.md` when writing or reviewing Stata code.
- Read `references/r-python-commands.md` when writing or reviewing R or Python code.
- Read `references/readme-sources.md` when provenance, package scope, or README-derived caveats matter.

## Routing Rules

- Prefer `csdid` for Callaway-Sant'Anna group-time ATTs in binary absorbing staggered adoption designs.
- Prefer `did_imputation` for Borusyak-Jaravel-Spiess imputation event studies, especially when flexible fixed effects, controls, pre-trend tests, or `event_plot` comparison plots are needed.
- Prefer `eventstudyinteract` for Sun-Abraham interaction-weighted event-study estimates with explicit cohort and control-cohort definitions.
- Prefer `did_multiplegt_dyn` for de Chaisemartin-D'Haultfoeuille designs with non-binary, non-absorbing, reversible, multiple-switch, or lagged treatment effects.
- Use `bacondecomp` and `twowayfeweights` as diagnostics for TWFE weights, negative weights, and heterogeneity risk.
- Use `HonestDiD` or Stata `honestdid` after obtaining an event-study coefficient vector and covariance matrix to assess sensitivity to violations of parallel trends.

## Guardrails

- Check whether the treatment date variable means first treatment date, current treatment, or relative event time; many commands use different inputs.
- Check whether never-treated units are coded as missing, zero, or a later artificial date before passing data to a command.
- Do not mix event-study coefficients from different normalizations without explaining the reference period and event-time construction.
- If a command stores results in nonstandard matrices, post or extract the intended coefficient vector and covariance matrix before downstream tests or HonestDiD.
