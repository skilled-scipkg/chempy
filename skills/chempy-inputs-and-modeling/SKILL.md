---
name: chempy-inputs-and-modeling
description: This skill should be used when users ask about inputs and modeling in chempy; it prioritizes documentation references and then source inspection only for unresolved details.
---

# chempy: Inputs and Modeling

## High-Signal Playbook

### Simulation bootstrap (run once per environment)
```bash
python -m pip install -e .
python -m pip install -e ".[native,integrators]"  # optional speedups
```

```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "pyneqsys", "quantities", "tabulate")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

### Route conditions
- Use this skill when the user is defining reaction systems, parameterizing kinetics/thermodynamics, or fitting model parameters.
- Route to `chempy-examples-and-tutorials` for notebook-first walkthroughs and cookbook-style demos.
- Route to `chempy-advanced-topics` for paper-level capability/citation questions or analytic CSTR derivations.

### Triage questions
- Are you building a new kinetic model, or fitting parameters to data?
- Do you need unit-aware modeling (`chempy.units`) end-to-end?
- Are temperature-dependent rates required (for example Eyring/Arrhenius)?
- Do you need C++-native integration speedups (`get_native`) or pure symbolic/numeric integration?
- Do substances need explicit composition for conservation checks/reduction?
- What are the required outputs: concentration traces, fitted parameters, or both?

### Canonical workflow
1. Start from a minimal `ReactionSystem` (usually `ReactionSystem.from_string(...)`) and explicit `Substance` definitions when conservation matters.
2. Build ODEs with `chempy.kinetics.ode.get_odesys` and decide on `include_params`, unit registry, and substitutions.
3. Provide initial concentrations and parameter dicts with consistent units.
4. Integrate first with `integrator="scipy"` to get a robust baseline; switch to `cvode` after baseline validation.
5. If speed is needed and dependencies are available, compile a native solver via `chempy.kinetics._native.get_native`.
6. For parameter estimation, use `chempy.util.regression` / `chempy.kinetics.arrhenius.fit_arrhenius_equation` and validate residual behavior.
7. Confirm mass/conservation invariants and rerun with perturbed key parameters for robustness.

### Minimal working example
Source: `examples/interactive_kinetic_modelling.ipynb`

```python
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys

rsys = ReactionSystem.from_string(
    '''
A -> B; 'k1'
B + C -> P; 'k2'
'''
)

odesys, _ = get_odesys(
    rsys,
    include_params=False,
)
res = odesys.integrate(
    [0, 60, 120],
    {"A": 1e-6, "B": 0.0, "C": 1e-6, "P": 0.0},
    {"k1": 5.8 / 60.0, "k2": 4.0e3},
    integrator="scipy",
)
assert res.yout[-1, 0] < res.yout[0, 0]
```

Optional native acceleration:

```python
from chempy.kinetics._native import get_native

native = get_native(rsys, odesys, "cvode")
fast = native.integrate(
    120.0,
    {"A": 1e-6, "B": 0.0, "C": 1e-6, "P": 0.0},
    {"k1": 5.8 / 60.0, "k2": 4.0e3},
    integrator="cvode",
)
```

### Pitfalls and fixes
- Missing composition metadata reduces model checks/reduction opportunities.
  - Fix: define `Substance(..., composition=...)` as in `examples/protein_binding_unfolding_4state_model.ipynb`.
- Missing runtime dependencies (`pyodesys`, `quantities`, `tabulate`) block many examples.
  - Fix: run the bootstrap commands above and recheck imports before debugging model code.
- `get_native(...)` fails if `pyodesys.native` is unavailable.
  - Fix: fall back to direct `odesys.integrate(...)` or install native backend support (`chempy/kinetics/_native.py`).
- Invalid substitution keys to `get_odesys` raise a hard error.
  - Fix: ensure substitution keys exist in rate expressions (`chempy/kinetics/ode.py`).
- Parameter dict mismatch when `include_params=False` causes integration failures.
  - Fix: pass every required key (`k*`, temperature keys, CSTR feed keys) from `odesys.param_names`.
- Data-fitting notebook has strict shape assumptions and visible outlier discussion.
  - Fix: validate replicate counts and filter obvious outliers before regression (`examples/_kinetic_model_fitting.ipynb`).
- Ionic-strength workflows can silently be non-physical if not charge neutral.
  - Fix: heed `ionic_strength` warnings and verify charge balance (`chempy/electrolytes.py`).

### Convergence/validation checks
- Check solver success diagnostics (`result.info`) before interpreting trends.
- Compare native and non-native integrations on the same setup for numerical agreement.
- Validate conserved totals using composition invariants in complex models.
- For regressions, inspect residual structure and stability under removal of suspicious series.
- Stress-test sensitivity by perturbing key kinetic parameters and initial concentrations.

## Scope
- Handle questions about inputs, system setup, models, and physical parameterization.
- Keep responses abstract and architectural for large codebases; avoid exhaustive per-function documentation unless requested.

## Primary documentation references
- `examples/protein_binding_unfolding_4state_model.ipynb`
- `examples/interactive_kinetic_modelling.ipynb`
- `examples/_kinetic_model_fitting.ipynb`

## Workflow
- Start with the primary references above.
- If details are missing, inspect `references/doc_map.md` for the complete topic document list.
- Use tutorials/examples as executable usage patterns when available.
- Use tests as behavior or regression references when available.
- If ambiguity remains after docs, inspect `references/source_map.md` and start with the ranked source entry points.
- Cite exact documentation file paths in responses.

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
- ODE construction, substitutions, CSTR flags: `chempy/kinetics/ode.py`
- Native-code path and integrator constraints: `chempy/kinetics/_native.py`
- Rate expressions (Arrhenius/Eyring/SinTemp): `chempy/kinetics/rates.py`
- Arrhenius fitting helpers: `chempy/kinetics/arrhenius.py`
- Regression/IRLS internals: `chempy/util/regression.py`
- Ionic-strength and Debye-Huckel helpers: `chempy/electrolytes.py`
- Core reaction-system behavior: `chempy/reactionsystem.py`
- Prefer targeted source search (for example: `rg -n "<symbol_or_keyword>" chempy`).
