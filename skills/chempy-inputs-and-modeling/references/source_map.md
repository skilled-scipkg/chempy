# chempy source map: Inputs and Modeling

Generated from source roots:
- `chempy`

Use this map only after exhausting the topic docs in `doc_map.md`.

## Preflight for simulation runs
- Baseline dependencies: `python -m pip install -e .`
- Optional native/cvode acceleration: `python -m pip install -e ".[native,integrators]"`
- Dependency probe:

```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "pyneqsys", "quantities", "tabulate", "sympy")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

## Fast source navigation
- `rg -n "ReactionSystem|from_string|get_odesys|get_native|fit_arrhenius_equation|least_squares|irls" chempy`
- `rg -n "def test_.*(reactionsystem|odesys|native|arrhenius|electrolytes|regression)" chempy/**/tests/*.py`

## Function-level entry points and checks
| Path | Functions / methods to inspect first | Validation checkpoint |
| --- | --- | --- |
| `chempy/reactionsystem.py` | `ReactionSystem.from_string`, `ReactionSystem.rates`, `ReactionSystem.categorize_substances` | `python -m pytest -q chempy/tests/test_reactionsystem.py -k "from_string or rates"` |
| `chempy/kinetics/ode.py` | `get_odesys`, `chained_parameter_variation`, `_create_odesys` | `python -m pytest -q chempy/kinetics/tests/test_ode.py -k "get_odesys or cstr"` |
| `chempy/kinetics/rates.py` | `RateExpr`, `Arrhenius`, `Eyring`, `SinTemp`, `mk_Radiolytic` | `python -m pytest -q chempy/kinetics/tests/test_rates.py -k "ArrheniusMassAction or EyringMassAction or mk_Radiolytic"` |
| `chempy/kinetics/_native.py` | `get_native`, `_get_comp_conc`, `_get_subst_comp` | `python -m pytest -q chempy/kinetics/tests/test__native.py -k get_native` |
| `chempy/kinetics/arrhenius.py` | `arrhenius_equation`, `fit_arrhenius_equation`, `ArrheniusParam` | `python -m pytest -q chempy/kinetics/tests/test_arrhenius.py -k arrhenius_equation` |
| `chempy/util/regression.py` | `least_squares`, `irls`, `least_squares_units` | `python -m pytest -q chempy/util/tests/test_regression.py` |
| `chempy/electrolytes.py` | `ionic_strength`, `A`, `B`, `davies_log_gamma` | `python -m pytest -q chempy/tests/test_electrolytes.py -k "ionic_strength or A"` |
| `chempy/units.py` | `to_unitless`, `allclose`, `get_derived_unit` | `python -m pytest -q chempy/tests/test_units.py -k dimensionality` |

## Minimal modeling smoke test (requires `pyodesys`)
```bash
python - <<'PY'
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys

rsys = ReactionSystem.from_string("A -> B; 'k'")
odesys, _ = get_odesys(rsys, include_params=False)
res = odesys.integrate([0, 1, 2], {"A": 1.0, "B": 0.0}, {"k": 0.3}, integrator="scipy")
assert res.yout[-1, 0] < res.yout[0, 0]
print("modeling smoke test: ok")
PY
```
