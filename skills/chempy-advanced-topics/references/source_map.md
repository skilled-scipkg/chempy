# chempy source map: Advanced Topics

Generated from source roots:
- `chempy`

Use this map only after exhausting the topic docs in `doc_map.md`.

## Preflight for advanced simulations
- Baseline dependencies: `python -m pip install -e .`
- Optional native/cvode acceleration: `python -m pip install -e ".[native,integrators]"`
- Dependency probe:

```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "sympy", "quantities", "tabulate")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

## Fast source navigation
- `rg -n "binary_irrev_cstr|unary_irrev_cstr|get_odesys|cstr|EqSystem|Henry_H_at_T|ionic_strength" chempy`
- `rg -n "def test_.*(integrated|ode|reactionsystem|equilibria|henry|electrolytes)" chempy/**/tests/*.py`

## Function-level entry points and checks
| Path | Functions / methods to inspect first | Validation checkpoint |
| --- | --- | --- |
| `chempy/kinetics/integrated.py` | `unary_irrev_cstr`, `binary_irrev_cstr`, `binary_irrev` | `python -m pytest -q chempy/kinetics/tests/test_integrated.py -k binary_irrev` |
| `chempy/kinetics/ode.py` | `get_odesys` (`cstr=True` path), `_create_odesys` | `python -m pytest -q chempy/kinetics/tests/test_ode.py -k cstr` |
| `chempy/kinetics/tests/_derive_analytic_cstr_bireac.ipynb` | symbolic derivation and parameter-symbol mapping for CSTR checks | `jupyter nbconvert --to notebook --execute chempy/kinetics/tests/_derive_analytic_cstr_bireac.ipynb --output /tmp/derive_analytic_cstr_bireac.executed.ipynb` |
| `chempy/reactionsystem.py` | `ReactionSystem.rates`, `ReactionSystem.from_string` | `python -m pytest -q chempy/tests/test_reactionsystem.py -k "rates__cstr or from_string"` |
| `chempy/equilibria.py` | `EqSystem` methods inherited from `ReactionSystem` for equilibrium workflows | `python -m pytest -q chempy/tests/test_equilibria.py -k EqSystem` |
| `chempy/henry.py` | `Henry_H_at_T`, `Henry`, `HenryWithUnits` | `python -m pytest -q chempy/tests/test_henry.py` |
| `chempy/electrolytes.py` | `ionic_strength`, `A`, `B`, `limiting_activity_product` | `python -m pytest -q chempy/tests/test_electrolytes.py -k "ionic_strength or A"` |
| `chempy/__init__.py` | exported surface for high-level capability claims | `python -m pytest -q chempy/tests/test_core.py` |

## CSTR analytic-vs-numeric smoke test (requires `pyodesys`)
```bash
python - <<'PY'
import numpy as np
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys
from chempy.kinetics.integrated import binary_irrev_cstr

rsys = ReactionSystem.from_string("OH + OH -> H2O2; 'k'")
odesys, extra = get_odesys(rsys, include_params=False, cstr=True)
fr, fc = extra["cstr_fr_fc"]
params = {"k": 5.0, fr: 13.0, fc["OH"]: 42.0, fc["H2O2"]: 11.0}
res = odesys.integrate(np.linspace(0, 0.17, 6), {"OH": 2.0, "H2O2": 3.0}, params)
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
print("advanced CSTR smoke test: ok")
PY
```
