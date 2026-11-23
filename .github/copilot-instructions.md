# Copilot Instructions

## Purpose

Teaching repository for cement paste hydration simulation using the **Parrot-Killoh kinetic model** coupled with **GEMS** thermodynamic equilibrium calculations via the `xgems` Python library. Students modify `run_cement_hydration_pk_model.ipynb` with their own cement paste formulations.

## Repository Structure

- `run_cement_hydration_pk_model.ipynb` — Tutorial notebook: define a cement recipe, run hydration kinetics, compute and plot phase volume fractions and masses over time.
- `run/hydration.py` — Kinetics and orchestration: Parrot-Killoh ODE solver, `run_hydration()` loop, and plotting helpers.
- `run/GEMSCalc.py` — Pythonic wrapper around `xgems.ChemicalEngine` (the `GEMS` class).
- `gems_files/` — GEMS thermodynamic database for cement hydration (`CemHyds-dat.lst` is the entry-point file passed to `GEMS()`).

## Architecture

The workflow is:

1. **Parrot-Killoh** (`parrot_killoh(wc, RH, T, fineness)`) — solves an ODE for each clinker phase (C3S, C2S, C3A, C4AF) to get degree-of-hydration `DoH[phase][time_step]`.
2. **GEMS loop** (`run_hydration(...)`) — for each time step, sets species lower bounds to `mass * (1 - DoH)` to constrain unreacted clinker, then calls `gemsk.equilibrate()`. Returns `vol_frac`, `mass_frac` (dicts keyed `{time: {phase: value}}`), and `density` list.
3. **Post-processing** — `to_phase_first_dict()` transposes to `{phase: [values over time]}` for `phase_plot()` and CSV export.

The `GEMS` class wraps `xgems.ChemicalEngine`. It manages the bulk composition vector `b` (element amounts in moles) and exposes Pythonic properties (`phase_volumes`, `phase_masses`, `aq_composition`, `pH`, etc.). Species amounts are added via `add_species_amt()` / `add_multiple_species_amt()`, which internally decompose by formula matrix.

## Key Conventions

- **Composition units in the notebook**: g per 100 g of cement. Converted to kg by multiplying by `1e-3` before passing to `GEMS`.
- **Time steps**: defined by `output_times` in `hydration.py` (days): `[0, 0.1, 1, 3, 7, 28, 30, 60, 90, 365, 730, 1000, 2000]`. Always starts at 0 for initial conditions.
- **Phase names** use cement chemistry shorthand: C3S, C2S, C3A, C4AF (clinker); `Gp` (gypsum); `H2O@` (water in xgems species list).
- **Bogue conversion**: `bouges_composition(CaO, SiO2, Al2O3, Fe2O3, SO3)` in `hydration.py` converts oxide analysis to clinker phase fractions.
- **GEMS initialization**: always call `GEMS('gems_files/CemHyds-dat.lst')` from the repo root. After building a composition with `add_*` methods, call `equilibrate()`. Use `warm_start()` between time steps, `cold_start()` for a fresh calculation.
- **Suppressing/activating phases**: use `supress_phase(name)` / `activate_phase(name)` to exclude phases from equilibrium calculations.
- **Result dicts are time-step-first** by default; use `to_phase_first_dict()` before plotting or exporting.

## Running the Tutorial

Run `run_cement_hydration_pk_model.ipynb` cells top-to-bottom from the repo root. Dependencies: `xgems`, `numpy`, `scipy`, `matplotlib`, `pandas`. See `README.md` for setup instructions and a guide to defining new formulations.
