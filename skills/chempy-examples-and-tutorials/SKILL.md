---
name: chempy-examples-and-tutorials
description: This skill should be used when users ask about examples and tutorials in chempy; it prioritizes documentation references and then source inspection only for unresolved details.
---

# chempy: Examples and Tutorials

## High-Signal Playbook

### Simulation bootstrap (run once per environment)
```bash
python -m pip install -e .
python -m pip install -e ".[native,integrators]"  # optional speedups
```

```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "quantities", "tabulate", "ipywidgets")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

### Route conditions
- Use this skill when the user asks for runnable notebook patterns, quick demos, or "show me a working ChemPy setup".
- Route to `chempy-inputs-and-modeling` when the user needs deeper parameter-estimation workflow or architecture decisions.
- Route to `chempy-advanced-topics` for paper-level capability claims or analytic CSTR derivation details.

### Triage questions
- Is the user after kinetics integration, electrolyte properties, symmetry analysis, or equilibrium examples?
- Does the run require units, temperature dependence, or CSTR feed terms?
- Does the user need an interactive notebook flow (`ipywidgets`) or a script-friendly minimal example?
- Is there an analytic reference solution to compare against?
- Do we need symbolic expressions, or only numerical trajectories?

### Canonical workflow
1. Pick the nearest notebook in `examples/` by physical problem type.
2. Run the smallest baseline cell sequence first (imports, model definition, one integration with `integrator="scipy"`).
3. Keep reaction strings and parameter keys exactly as in the notebook until baseline reproduces.
4. Add units/substitutions only after the baseline run is stable.
5. For CSTR/time-dependent cases, inspect generated parameter names and expressions before sweeping parameters.
6. Validate with built-in comparisons (analytic-vs-numeric or autonomous-vs-nonautonomous checks) before plotting/reporting.
7. Only then generalize to user-specific chemistry and plotting/reporting.

### Minimal working example
Source: `examples/kinetics_ode_time_dependent.ipynb`

```python
import math
from collections import defaultdict
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys
from chempy.kinetics.rates import SinTemp

rsys = ReactionSystem.from_string(
    """
2 HNO2 -> H2O + NO + NO2; EyringParam(dH=85e3, dS=10)
2 NO2 -> N2O4; EyringParam(dH=70e3, dS=20)
"""
)
st = SinTemp(unique_keys="Tbase Tamp Tangvel Tphase".split())
odesys, _ = get_odesys(
    rsys,
    include_params=False,
    substitutions={"temperature": st},
)
res = odesys.integrate(
    [0, 10, 20, 30],
    defaultdict(lambda: 0, HNO2=1, H2O=55),
    dict(Tbase=300, Tamp=10, Tangvel=2 * math.pi / 10, Tphase=-math.pi / 2),
    integrator="scipy",
)
assert res.yout.shape[0] == 4
```

### Pitfalls and fixes
- Missing runtime dependencies (`pyodesys`, `tabulate`, `quantities`) prevent notebook examples from running.
  - Fix: run the bootstrap commands above and verify module availability before model debugging.
- CSTR examples add synthetic parameter keys (`feedratio`, `fc_<substance>`) that are easy to miss.
  - Fix: read `extra["cstr_fr_fc"]` from `get_odesys(..., cstr=True)` (`examples/kinetics_cstr.ipynb`, `chempy/kinetics/ode.py`).
- Time-dependent temperature substitutions fail if parameter keys do not match provided dict keys.
  - Fix: keep `unique_keys` and parameter dict names aligned (`examples/kinetics_ode_time_dependent.ipynb`, `chempy/kinetics/rates.py`).
- Debye-Huckel constant `A` must be converted for base-10 log forms.
  - Fix: divide by `ln(10)` as shown in `examples/demo_debye_huckel.ipynb`.
- Deprecated imports can route users to stale APIs.
  - Fix: prefer `chempy.electrolytes` over `chempy.debye_huckel` (`chempy/debye_huckel.py`).
- Symmetry helpers are strict about valid point groups and representation shape.
  - Fix: verify group labels and dimensions before calling decomposition/SALC routines (`chempy/symmetry/representations.py`, `chempy/symmetry/salcs.py`).
- Notebook-native UX (`%matplotlib inline`, widgets) may not translate directly to batch scripts.
  - Fix: strip notebook magics and call plotting/integration functions directly.

### Convergence/validation checks
- For autonomous conversion, verify `len(autsys.dep) == len(odesys.dep) + 1`.
- For CSTR, compare numeric integration against analytic references in `chempy.kinetics.integrated`.
- Check solver diagnostics (`result.info` when available) and reject runs with failed/stiff-step churn.
- For electrolyte sweeps, confirm physically plausible trends of `A(T)` and `B(T)` and charge neutrality assumptions.
- For symmetry examples, cross-check decompositions with `print_table(...)` ordering.

## Scope
- Handle questions about worked examples, tutorials, and cookbook usage.
- Keep responses abstract and architectural for large codebases; avoid exhaustive per-function documentation unless requested.

## Primary documentation references
- `examples/symmetry.ipynb`
- `examples/kinetics_ode_time_dependent.ipynb`
- `examples/kinetics_decay_chain_units_varied_params.ipynb`
- `examples/kinetics_cstr.ipynb`
- `examples/demo_debye_huckel.ipynb`
- `examples/aqueous_radiolysis.ipynb`
- `examples/ammonical_cupric_solution.ipynb`
- `examples/_switch_between_reduced_ode_systems.ipynb`
- `examples/_surface_reactions_homogeneous.ipynb`
- `examples/_regression.ipynb`
- `examples/_ode_system_sympy_symbols.ipynb`
- `examples/_kinetics_radiolysis_analytic.ipynb`

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
- Kinetics ODE generation and CSTR wiring: `chempy/kinetics/ode.py`
- Analytic kinetics references: `chempy/kinetics/integrated.py`
- Temperature/rate expressions: `chempy/kinetics/rates.py`
- Debye-Huckel and ionic-strength equations: `chempy/electrolytes.py`
- Symmetry representation decomposition: `chempy/symmetry/representations.py`
- SALC construction details: `chempy/symmetry/salcs.py`
- Native codepath used by interactive modeling examples: `chempy/kinetics/_native.py`
- Prefer targeted source search (for example: `rg -n "<symbol_or_keyword>" chempy`).
