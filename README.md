# Multi-objective optimization of follow-up schedules in amblyopia treatment

Analysis code for the manuscript "Jointly Optimizing Visit Intervals and Plateau-Declaration Rules in Amblyopia Treatment: A Simulation-Based Multi-Objective Evolutionary Approach".

The pipeline (1) harmonizes public Pediatric Eye Disease Investigator Group (PEDIG) trial data, (2) fits a Bayesian nonlinear mixed-effects model of amblyopic-eye visual acuity trajectories, (3) simulates virtual patient cohorts with noisy, line-rounded acuity measurements, and (4) optimizes visit intervals and plateau-declaration rules with multi-objective evolutionary algorithms.

## Data

The trial data are not redistributed here. Download the public datasets from the Jaeb Center for Health Research (https://public.jaeb.org/pedig/stdy) and unzip each study into

```
<root>/raw/<STUDY>/extracted/
```

with the study folder names `ATS1`, `ATS2A`, `ATS2B`, `ATS3`, `ATS05`, `ATS06`, `ATS10`, `ATS12`, `ATS13`, `ATS15`, `ATS16`. The scripts locate the data tables inside `extracted/` by file name.

Publications based on these data must include the study-specific attribution given in each study's README and the statement: "The analyses content and conclusions presented herein are solely the responsibility of the authors and have not been reviewed or approved by PEDIG."

## Requirements

Python 3.10 with the packages in `requirements.txt`. Figures use Times New Roman; if it is not installed system-wide, put the TTF files (`times.ttf`, `timesbd.ttf`, `timesi.ttf`, `timesbi.ttf`) in a `fonts/` folder next to `src/`.

## Running

```
bash run_all.sh <root> [n_jobs]
```

runs every step in order. The optimization steps use `n_jobs` worker processes (default 32) and save each run under `results/*/runs/`, so an interrupted step resumes where it stopped. On a 32-core node, the static optimization step takes about 12 hours, mostly because of SMS-EMOA; the other optimization steps take a few hours each. All random seeds are fixed in the scripts.

| Step | Script | Output (below `<root>`) | Manuscript |
|---|---|---|---|
| 1 | `harmonize_aux.py` | `derived/aux_*.csv` | Table 1 (auxiliary trials) |
| 2 | `build_trajectories.py` | `derived/traj.csv`, `pts.csv`, `retest.csv` | Section 2.1, Table 1 |
| 3 | `fit_trajectory_model.py` | `results/trajectory_model/` | Section 3.1, Table 3 (RQ1) |
| 4 | `optimize_static.py` | `results/static/` | Section 3.2, Table 4 (RQ2); Section 3.3 (RQ3, part) |
| 4 | `check_simulator.py` | printed | Section 3.2 (simulator check) |
| 5 | `compare_algorithms.py` | `results/algorithms/` | Section 3.3, Table 5, Table S1 (RQ3) |
| 6 | `optimize_adaptive.py` | `results/adaptive/` | Section 3.4, Table S2 (RQ4) |
| 7 | `sensitivity_and_replay.py` | `results/sensitivity/` | Section 3.5, Tables 6 and 7 |
| 8 | `figure_1_workflow.py`, `figures_2_4_5.py`, `figure_3_front.py` | `results/figures/` | Figures 1-5 |

`optimize_static.py` also serves as a module: the simulator (`cohort`), the vectorized evaluator of a schedule (`evaluate`), the epsilon-constraint selection rule (`select`) and the held-out evaluation (`test_eval`) are imported by the later steps.

## Encoding of a schedule

A static schedule is the integer vector `(d1, ..., d6, L, m, Tmin)`: the first six intervals between visits in weeks (`d6` is repeated until the 52-week horizon), the improvement threshold in lines (0.1 logMAR), the number of consecutive non-improving visits that declares a plateau, and the minimum treatment time before a plateau can be declared. The comparator is `(5, 5, 5, 5, 5, 5, 1, 2, 0)`.
