# Fed-batch Fermentation Kinetics Fitting

This repo fits fermentation kinetics models to experimental fed-batch data (biomass `X`, substrate `S`, product `P` over time `t`, from [Dataset1.csv](Dataset1.csv), digitized from the paper referenced in the `wpd_*` files, DOI [10.3390/life12050621](https://doi.org/10.3390/life12050621)).

Each notebook proposes a different ODE model for how `X`, `S`, `P` evolve, then uses `scipy.optimize.differential_evolution` (a global optimizer) to find the model parameters that best fit the data, and `scipy.integrate.solve_ivp` to simulate the fitted model. There's a substrate feed event partway through the run (`S` jumps back up around t≈150), so each model solves the ODEs in two segments: before and after the feed.

- [S1fit-wide-fb.ipynb](S1fit-wide-fb.ipynb) — simplest model (logistic growth, linear product formation)
- [S2fit-LSODA2-wide-fb.ipynb](S2fit-LSODA2-wide-fb.ipynb) — extended model with substrate/product inhibition terms, solved with LSODA
- [S3fit-wide.ipynb](S3fit-wide.ipynb) — Monod-kinetics variant

Each notebook prints the estimated parameters and goodness-of-fit (R², RMSE), plots the fit against the experimental data, and writes a simulated dataset CSV (e.g. `optimized_model_dataset1.csv`).

Fitting runs `differential_evolution` for up to 2000 generations, so each notebook can take several minutes to finish.

## Setup

You'll need Python installed (3.10+ recommended). Check what you have with:

```bash
python --version
```

Then, from the repo root:

```bash
python -m venv .venv
```

Activate it — pick the line for your shell:

```bash
.venv\Scripts\activate        # Windows cmd/PowerShell
source .venv/bin/activate     # macOS/Linux/Git Bash
```

Install dependencies:

```bash
pip install -r requirements.txt
```

## Running

Open the notebooks in Jupyter:

```bash
jupyter notebook
```

Then open any of the `.ipynb` files and run all cells (Run → Run All Cells).

Or run one from the command line without opening a browser:

```bash
jupyter nbconvert --to notebook --execute --output-dir outputs/S1_OUTPUT S1fit-wide-fb.ipynb
```

(swap in `S2fit-LSODA2-wide-fb.ipynb` / `outputs/S2_OUTPUT` or `S3fit-wide.ipynb` / `outputs/S3_OUTPUT` for the other models.)

## Outputs

Each run's results go into `outputs/<notebook>_OUTPUT/`, so results from different people/runs don't overwrite each other and don't get mixed up with the source notebooks:

```
outputs/
  S1_OUTPUT/
    S1fit-wide-fb.ipynb          # executed notebook — parameters, R²/RMSE, and plots embedded as cell outputs
    optimized_model_dataset1.csv # simulated t, X, S, P curve from the fitted model
    RUN_INFO.txt                 # who ran it and when
  S2_OUTPUT/  ...
  S3_OUTPUT/  ...
```

The CSV filename comes from the notebook's own `to_csv(...)` call (currently the same name in all three notebooks) — since each model writes into its own subfolder, this is safe, but keep it in mind if you change the notebooks' output filenames.

Commit the contents of `outputs/` (they're small and are the actual point of "running" this) — just not `.venv/` (see [.gitignore](.gitignore)).
