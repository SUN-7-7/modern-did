# modern-did

`modern-did` is a Codex skill for modern difference-in-differences workflows in Stata, R, and Python.

It helps Codex choose and implement appropriate estimators for staggered adoption, heterogeneous treatment effects, event studies, TWFE diagnostics, and sensitivity analysis.

## What It Covers

- Callaway and Sant'Anna: `csdid`
- Sun and Abraham: `eventstudyinteract`
- Borusyak, Jaravel, and Spiess: `did_imputation`
- de Chaisemartin and D'Haultfoeuille: `did_multiplegt` and `did_multiplegt_dyn`
- Goodman-Bacon decomposition: `bacondecomp`
- TWFE weight diagnostics: `twowayfeweights`
- Rambachan and Roth sensitivity analysis: `HonestDiD` and Stata `honestdid`

## Skill Structure

```text
modern-did/
  SKILL.md
  agents/
    openai.yaml
  references/
    workflow.md
    stata-commands.md
    r-python-commands.md
    readme-sources.md
```

`SKILL.md` is the Codex entry point. The reference files are loaded only when needed:

- `workflow.md`: estimator selection, reporting checklist, and interpretation guidance
- `stata-commands.md`: Stata installation and command templates
- `r-python-commands.md`: R and Python command templates
- `readme-sources.md`: upstream README/help-file provenance and caveats

## Install Locally

Copy or clone this folder into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/SUN-7-7/modern-did.git ~/.codex/skills/modern-did
```

Then start a new Codex thread and ask for modern DID help, or invoke it explicitly:

```text
Use $modern-did to choose a DID estimator for my staggered adoption panel.
```

## Notes

This skill is workflow guidance, not a statistical package. It points Codex toward the right estimator, command syntax, diagnostics, and reporting choices. Always verify package versions and command help files in your own Stata, R, or Python environment before final analysis.
