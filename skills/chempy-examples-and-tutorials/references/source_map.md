# chempy source map: Examples and Tutorials

Generated from source roots:
- `chempy`

Use this map only after exhausting the topic docs in `doc_map.md`.

## Preflight for runnable examples
- Baseline dependencies: `python -m pip install -e .`
- Optional native/cvode acceleration: `python -m pip install -e ".[native,integrators]"`
- Dependency probe:

```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "quantities", "tabulate", "ipywidgets")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

## Fast source navigation
- `rg -n "get_odesys|SinTemp|Eyring|binary_irrev_cstr|ionic_strength|calc_salcs|Reducible" chempy`
- `rg -n "def test_.*(ode|integrated|rates|electrolytes|salcs|representations|regression)" chempy/**/tests/*.py`

## Function-level entry points and checks
| Path | Functions / methods to inspect first | Validation checkpoint |
| --- | --- | --- |
| `chempy/kinetics/ode.py` | `get_odesys`, `_create_odesys`, `chained_parameter_variation` | `python -m pytest -q chempy/kinetics/tests/test_ode.py -k "time_dep or cstr"` |
| `chempy/kinetics/integrated.py` | `pseudo_irrev`, `binary_irrev`, `unary_irrev_cstr`, `binary_irrev_cstr` | `python -m pytest -q chempy/kinetics/tests/test_integrated.py -k "pseudo_irrev or binary_irrev"` |
| `chempy/kinetics/rates.py` | `SinTemp`, `Arrhenius`, `Eyring`, `mk_Radiolytic` | `python -m pytest -q chempy/kinetics/tests/test_rates.py -k "ArrheniusMassAction or EyringMassAction"` |
| `chempy/electrolytes.py` | `A`, `B`, `limiting_log_gamma`, `davies_log_gamma` | `python -m pytest -q chempy/tests/test_electrolytes.py -k "A or B or limiting_log_gamma"` |
| `chempy/reactionsystem.py` | `ReactionSystem.from_string`, `ReactionSystem.rates` | `python -m pytest -q chempy/tests/test_reactionsystem.py -k "from_string or rates"` |
| `chempy/symmetry/salcs.py` | `calc_salcs_projection`, `calc_salcs_func` | `python -m pytest -q chempy/symmetry/tests/test_salcs.py -k calc_salcs` |
| `chempy/symmetry/representations.py` | `Reducible.decomp`, `Reducible.vibe_modes`, `Reducible.from_atoms` | `python -m pytest -q chempy/symmetry/tests/test_representations.py -k "decomp or from_atoms"` |
| `chempy/util/regression.py` | `least_squares`, `irls` | `python -m pytest -q chempy/util/tests/test_regression.py` |

## Minimal tutorial-style smoke test (requires `pyodesys`)
```bash
python - <<'PY'
import math
from collections import defaultdict
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys
from chempy.kinetics.rates import SinTemp

rsys = ReactionSystem.from_string("2 HNO2 -> H2O + NO + NO2; EyringParam(dH=85e3, dS=10)")
st = SinTemp(unique_keys="Tbase Tamp Tangvel Tphase".split())
odesys, _ = get_odesys(rsys, include_params=False, substitutions={"temperature": st})
res = odesys.integrate(
    [0, 10, 20],
    defaultdict(lambda: 0, HNO2=1, H2O=55),
    {"Tbase": 300, "Tamp": 10, "Tangvel": 2 * math.pi / 10, "Tphase": -math.pi / 2},
    integrator="scipy",
)
assert res.yout.shape[0] == 3
print("examples smoke test: ok")
PY
```
