---
name: chempy-index
description: This skill should be used when users ask how to use chempy and the correct generated documentation skill must be selected before going deeper into source code.
---

# chempy Skills Index

## Route the request
- Classify the request into one of the generated topic skills listed below.
- Prefer abstract, workflow-level guidance for large scientific packages; do not attempt full function-by-function coverage unless explicitly requested.

## Simulation bootstrap (start here for runnable work)
- Run `python -m pip install -e .` before ODE or notebook workflows.
- For native/cvode workflows, run `python -m pip install -e ".[native,integrators]"`.
- Open `references/simulation_bootstrap.md` for a copy-paste startup workflow and checkpoints.

## Generated topic skills
- `chempy-examples-and-tutorials`: Worked notebooks, cookbook usage, runnable demo baselines.
- `chempy-inputs-and-modeling`: Reaction-system setup, parameterization, units-aware ODE workflows, model fitting.
- `chempy-advanced-topics`: Consolidated narrow topics (JOSS-paper framing + analytic CSTR derivations).

## Documentation-first inputs
- `joss-paper`

## Tutorials and examples roots
- `examples`

## Test roots for behavior checks
- `chempy/tests`
- `chempy/electrochemistry/tests`
- `chempy/kinetics/tests`
- `chempy/printing/tests`
- `chempy/properties/tests`
- `chempy/symmetry/tests`
- `chempy/thermodynamics/tests`
- `chempy/util/tests`

## Escalate only when needed
- Start from topic skill primary references.
- If those references are insufficient, search the topic doc map:
  `skills/chempy-examples-and-tutorials/references/doc_map.md`,
  `skills/chempy-inputs-and-modeling/references/doc_map.md`,
  `skills/chempy-advanced-topics/references/doc_map.md`.
- If documentation still leaves ambiguity, inspect the matching topic source map:
  `skills/chempy-examples-and-tutorials/references/source_map.md`,
  `skills/chempy-inputs-and-modeling/references/source_map.md`,
  `skills/chempy-advanced-topics/references/source_map.md`.
- Use targeted symbol search while inspecting source (e.g., `rg -n "<symbol_or_keyword>" chempy`).

## Source directories for deeper inspection
- `chempy`
