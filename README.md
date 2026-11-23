# GEMS Lab — Cement Hydration Tutorials

A teaching repository for simulating cement paste hydration using the **Parrot-Killoh kinetic model** coupled with **GEMS thermodynamic equilibrium calculations** via the [`xgems`](https://github.com/gemshub/xgems) Python library.

Students work by modifying the provided tutorial notebook with their own cement paste formulations.

## Getting Started

### Windows

1. Install [Anaconda](https://www.anaconda.com/download) and [Visual Studio Code](https://code.visualstudio.com/).
2. Open **Anaconda Prompt** from the Start menu and run the [environment setup commands](#create-the-conda-environment).
3. Open VS Code and follow [Running the tutorial in VS Code](#running-the-tutorial-in-vs-code).

### macOS

1. Install [Anaconda](https://www.anaconda.com/download) and [Visual Studio Code](https://code.visualstudio.com/).
2. Open **Terminal** and run the [environment setup commands](#create-the-conda-environment). If `conda` is not available, follow the Anaconda installer's instructions to initialize your shell and reopen Terminal.
3. Open VS Code and follow [Running the tutorial in VS Code](#running-the-tutorial-in-vs-code).

### Create the conda environment

Run these commands in Anaconda Prompt on Windows or Terminal on macOS:

```bash
conda create -n gems-lab -c conda-forge python numpy scipy matplotlib pandas ipykernel
conda activate gems-lab
conda install conda-forge::xgems
```

See the [xgems installation instructions](https://github.com/gemshub/xgems#installation-using-conda) for more information.

### Running the tutorial in VS Code

1. Install the **Python** and **Jupyter** extensions by Microsoft from the VS Code Extensions view.
2. Download or clone this repository. In VS Code, choose **File > Open Folder** and open the repository root (`gems_lab`), not just the notebook; the GEMS database path is relative to that folder.
3. Open `run_cement_hydration_pk_model.ipynb`. Select **Select Kernel** at the top right, choose **Python Environments**, and select the `gems-lab` conda environment.
4. Run the cells top-to-bottom using **Run All** or the cell run buttons. The notebook computes degrees of hydration, runs GEMS equilibrium calculations, and plots and exports phase volume fractions to `results_hydration.csv`.

To try your own formulation, edit the input recipe cell and rerun the notebook from the beginning.

## Repository Structure

```
gems_lab/
├── run_cement_hydration_pk_model.ipynb  # Tutorial notebook
├── run/
│   ├── hydration.py     # Parrot-Killoh model, run_hydration() loop, plotting helpers
│   └── GEMSCalc.py      # Pythonic wrapper around xgems.ChemicalEngine
└── gems_files/          # GEMS thermodynamic database for cement hydration
    ├── CemHyds-dat.lst  # Entry-point file (pass this path to GEMS())
    └── ...
```

## Defining Your Own Cement Formulation

The key inputs are clinker phase masses (g per 100 g of cement), water-to-cement ratio, gypsum content, temperature, relative humidity, and Blaine fineness.

### Option 1 — Direct phase masses

```python
clinker_phases = {
    "C3S": 64.6,   # g / 100 g cement
    "C2S":  9.3,
    "C3A":  7.4,
    "C4AF": 7.8,
}
CSH2 = 2.2   # gypsum
wc   = 0.5   # water-to-cement ratio
T    = 25    # °C
RH   = 1.0   # relative humidity (0–1)
fineness = 385  # Blaine fineness, cm²/g
```

### Option 2 — Bogue conversion from oxide analysis

If you have an oxide composition instead, use `bouges_composition()`:

```python
from run.hydration import bouges_composition
clinker_phases = bouges_composition(CaO=65, SiO2=20, Al2O3=5, Fe2O3=3, SO3=2)
```

### Running the simulation

```python
from run.hydration import parrot_killoh, run_hydration, to_phase_first_dict, phase_plot

DoH = parrot_killoh(wc, RH, T, fineness)
vol_frac, mass_frac, density = run_hydration(clinker_phases, wc, CSH2, T, DoH)

gems_vol_frac = to_phase_first_dict(vol_frac)
phase_plot(gems_vol_frac).show()
```

## Model Overview

| Step | Function | Description |
|------|----------|-------------|
| Kinetics | `parrot_killoh()` | Solves Parrot-Killoh ODEs to get degree of hydration `DoH[phase][t]` |
| Equilibrium | `run_hydration()` | For each time step, constrains unreacted clinker via lower bounds and calls GEMS |
| Post-processing | `to_phase_first_dict()` | Transposes `{time: {phase: val}}` → `{phase: [vals]}` for plotting/export |

**Time steps** (days): `0, 0.1, 1, 3, 7, 28, 30, 60, 90, 365, 730, 1000, 2000`

## Phase Name Reference

| Symbol | Phase |
|--------|-------|
| C3S | Alite (Ca₃SiO₅) |
| C2S | Belite (Ca₂SiO₄) |
| C3A | Aluminate (Ca₃Al₂O₆) |
| C4AF | Ferrite (Ca₄Al₂Fe₂O₁₀) |
| Gp | Gypsum (CaSO₄·2H₂O) |
| H2O@ | Water (xgems species name) |

## Authors

- R.A. Patel (original implementation, 2021)
- G.D. Miron (modifications)
