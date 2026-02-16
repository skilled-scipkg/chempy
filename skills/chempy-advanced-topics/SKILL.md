---
name: chempy-advanced-topics
description: This skill should be used for high-level package framing (JOSS paper) and specialized analytic kinetics derivations that are too narrow for standalone topic skills.
---

# chempy: Advanced Topics

## High-Signal Playbook

### Simulation bootstrap (run once per environment)
```bash
python -m pip install -e .
python -m pip install -e ".[native,integrators]"  # optional speedups
```

```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "sympy", "quantities", "tabulate")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

### Route conditions
- Use this skill for package-level framing/capability questions tied to `joss-paper/paper.md`.
- Use this skill for symbolic/analytic CSTR derivation questions tied to `chempy/kinetics/tests/_derive_analytic_cstr_bireac.ipynb`.
- Route to `chempy-inputs-and-modeling` for production modeling/fitting workflows.
- Route to `chempy-examples-and-tutorials` for runnable notebook-first demos.

### Triage questions
- Is the request about citing or summarizing ChemPy capabilities?
- Is the request about deriving/validating closed-form CSTR kinetics?
- Is the reactor stoichiometry unary (`A -> B`) or binary (`2A -> nB`)?
- Do you need symbolic expressions, numerical validation, or both?
- Should results map directly to public APIs (`chempy.kinetics.integrated`) or remain derivation-only?

### Canonical workflow
1. Choose path: package framing (`paper.md`) vs analytic derivation notebook.
2. For capability claims, extract statements from `joss-paper/paper.md` and map each to concrete modules.
3. For CSTR derivation, reproduce the symbolic expression path in `_derive_analytic_cstr_bireac.ipynb`.
4. Build a `cstr=True` ODE system with `get_odesys` and capture feed keys from `extra["cstr_fr_fc"]`.
5. Compare integrated trajectory to `binary_irrev_cstr`/`unary_irrev_cstr` outputs.
6. Accept only if initial-condition and trajectory checks pass.

### Minimal working example
Sources: `chempy/kinetics/tests/_derive_analytic_cstr_bireac.ipynb`, `chempy/kinetics/integrated.py`

```python
import numpy as np
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys
from chempy.kinetics.integrated import binary_irrev_cstr

rsys = ReactionSystem.from_string("OH + OH -> H2O2; 'k'")
cstr, extra = get_odesys(rsys, include_params=False, cstr=True)
fr, fc = extra["cstr_fr_fc"]
params = {"k": 5, fr: 13, fc["OH"]: 42, fc["H2O2"]: 11}
res = cstr.integrate(np.linspace(0, 0.17), {"OH": 2, "H2O2": 3}, params)

ref = binary_irrev_cstr(
    res.xout,
    res.named_param("k"),
    res.named_dep("OH")[0],
    res.named_dep("H2O2")[0],
    res.named_param(fc["OH"]),
    res.named_param(fc["H2O2"]),
    res.named_param(fr),
)
assert np.all(np.isfinite(ref))
```

### Pitfalls and fixes
- Missing runtime dependencies (`pyodesys`, `sympy`, `quantities`) break advanced derivation/simulation paths.
  - Fix: run bootstrap commands and verify imports before debugging chemistry logic.
- `_derive_analytic_cstr_bireac.ipynb` is a derivation notebook, not a stable API surface.
  - Fix: use `chempy.kinetics.integrated.binary_irrev_cstr` / `unary_irrev_cstr` in production.
- Hard-coding CSTR feed keys is fragile.
  - Fix: always use `extra["cstr_fr_fc"]` keys from `get_odesys`.
- Wrong analytic function for stoichiometry yields misleading comparisons.
  - Fix: map `A -> B` to `unary_irrev_cstr`; `2A -> nB` to `binary_irrev_cstr`.
- Hyperbolic-function domains can break for non-physical parameter sets.
  - Fix: keep parameters positive/physical and validate early-time behavior.
- JOSS paper is a 2018 snapshot.
  - Fix: cross-check implementation specifics in current source modules before claiming behavior.

### Convergence/validation checks
- Verify analytic expressions reproduce initial conditions at `t=0`.
- Check numeric-analytic residuals remain near machine/solver tolerance.
- Confirm CSTR parameter naming and feed mapping from `extra["cstr_fr_fc"]`.
- For package summaries, map each capability claim to at least one concrete module/file.

## Scope
- Handle narrow, high-value topics that were previously single-doc skills.
- Keep this skill compact and routing-oriented; escalate to source only when docs are insufficient.

## Primary documentation references
- `joss-paper/paper.md`
- `chempy/kinetics/tests/_derive_analytic_cstr_bireac.ipynb`

## Workflow
- Start with the primary references above.
- If details are missing, inspect `references/doc_map.md` for complete advanced-topic inventory.
- For behavior-level questions, use `references/source_map.md` entry points.
- Cite exact documentation paths in responses.

## Tutorials and examples
- `examples`

## Test references
- `chempy/tests`
- `chempy/electrochemistry/tests`
- `chempy/kinetics/tests`
- `chempy/printing/tests`
- `chempy/properties/tests`
- `chempy/symmetry/tests`
- `chempy/thermodynamics/tests`
- `chempy/util/tests`

## Source entry points for unresolved issues
- CSTR ODE generation and feed-key wiring: `chempy/kinetics/ode.py`
- Closed-form CSTR kinetics APIs: `chempy/kinetics/integrated.py`
- Symbolic derivation notebook: `chempy/kinetics/tests/_derive_analytic_cstr_bireac.ipynb`
- Package surface and module exposure: `chempy/__init__.py`
- Core reaction-system model: `chempy/reactionsystem.py`
- Units/symbolics integration: `chempy/units.py`, `chempy/symbolic.py`
- Physical chemistry feature modules referenced by paper: `chempy/henry.py`, `chempy/equilibria.py`, `chempy/electrolytes.py`, `chempy/arrhenius.py`
- Prefer targeted source search (for example: `rg -n "<symbol_or_keyword>" chempy`).
