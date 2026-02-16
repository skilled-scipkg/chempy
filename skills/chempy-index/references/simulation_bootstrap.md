# chempy simulation bootstrap

Use this when you need a reliable starting point before choosing a topic skill.

## 1. Install runtime dependencies
```bash
python -m pip install -e .
python -m pip install -e ".[native,integrators]"  # optional
```

## 2. Verify critical modules
```bash
python - <<'PY'
import importlib.util as iu
mods = ("pyodesys", "pyneqsys", "quantities", "tabulate", "sympy")
print({m: bool(iu.find_spec(m)) for m in mods})
PY
```

## 3. Run one ODE smoke simulation
```bash
python - <<'PY'
from chempy import ReactionSystem
from chempy.kinetics.ode import get_odesys

rsys = ReactionSystem.from_string("A -> B; 'k'")
odesys, _ = get_odesys(rsys, include_params=False)
res = odesys.integrate([0, 1, 2], {"A": 1.0, "B": 0.0}, {"k": 0.3}, integrator="scipy")
assert res.yout[-1, 0] < res.yout[0, 0]
print("smoke simulation: ok")
PY
```

## 4. Validation checkpoints
- Core model parsing/rates: `python -m pytest -q chempy/tests/test_reactionsystem.py -k "from_string or rates"`
- Electrolyte sanity checks: `python -m pytest -q chempy/tests/test_electrolytes.py -k "ionic_strength or A"`
- Kinetics closed-form checks: `python -m pytest -q chempy/kinetics/tests/test_integrated.py -k "pseudo_irrev or binary_irrev"`

## 5. Route to the right skill
- Examples/cookbook usage: `skills/chempy-examples-and-tutorials/SKILL.md`
- Inputs/model design/fitting: `skills/chempy-inputs-and-modeling/SKILL.md`
- CSTR analytic derivation or package framing: `skills/chempy-advanced-topics/SKILL.md`
