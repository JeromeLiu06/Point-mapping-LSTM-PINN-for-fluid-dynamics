# Point-Mapping LSTM-PINN for Fluid Dynamics

Training code and numerical reference data for *Systematic Evaluation of
Point-Mapping LSTM Architectures for Physics-Informed Fluid-Flow
Reconstruction*. The release covers Experiments A–G and the finite-element
reference for the conventional lid-driven cavity (Case IV).

## Packages and entry points

Download the ZIP files attached to the same release. Extract each into a
common parent directory. The paths in the examples are relative to the
extracted package named in the table.

| Experiment | Release file | What it contains and how to start |
| --- | --- | --- |
| A | `Experiment_A_CT-LSTM-PINN.zip` | Three manufactured cases, seven model scripts per case, 80,000 updates at seed 0. The Case I CT run starts with `python case1_tgv/train_ct-lstm.py`. |
| B and C | `Experiment_BC_VR-LSTM-PINN_S-LSTM-PINN.zip` | Training module `epjplus_bc/`, frozen inputs, Case IV predictions and reevaluation, and two Experiment B loss histories. Use `python -m epjplus_bc B --case 4 --model lstm1 ...` or the C command below. |
| D | `Experiment_D_Seed_Repeats_40k.zip` | Four cases, feedforward PINN and Param-PINN repeats, and an additional S-LSTM option. Table S14 uses the two feedforward models at seeds 0–2 and 40,000 updates. Start with `python train.py --case 4 --model Param-PINN --seed 0`. |
| E | `Experiment_E_S-LSTM-PINN_Sequence_Length.zip` | Four cases, S-LSTM windows of 1, 2 and 4 points at 30,000 updates, and 300-update cost pilots at window length 8. Start with `python train.py --case 1 --length 4`. |
| F | `Experiment_F_VR-LSTM-PINN_S-LSTM-PINN.zip` | Both LSTM variants, four frozen inputs, and the 24-run matrix (4 cases × 2 models × 3 seeds). Use `python train_all_lstm.py --list` to inspect the matrix. Each full run uses 80,000 updates. |
| G | `Experiment_G_CT-LSTM-PINN.zip` | CT-LSTM-PINN, four case configurations, and eight run scripts at seeds 0 and 1 (80,000 updates each). Copy the four F `data/locked/*.npz` inputs into G's `data/locked/`, then run, for example, `python train_case1_tgv_seed0.py`. |
| Case IV FEM | `Case4_FEM.zip` | Taylor–Hood solver, N=64/128/256 velocity references, mesh convergence, and checksums. Run `python verify_fem.py` after extracting the `Case4_FEM_v4/` directory. |

Install the relevant package's requirements in a Python environment with a
CUDA-enabled PyTorch build for full neural-network runs. The A, D, and E
scripts use NumPy, PyTorch, and Matplotlib. The FEM package lists the solver
dependencies in its own `requirements.txt`.

## Experiment B and C

From the B/C package, with the FEM package extracted alongside it:

```sh
python -m epjplus_bc B --case 4 --model lstm1 --device cuda --fem-reference ../Case4_FEM_v4/levels/N256/velocity_reference_grid.npz --output runs/B/VR-LSTM-PINN
python -m epjplus_bc B --case 4 --model lstm2 --device cuda --fem-reference ../Case4_FEM_v4/levels/N256/velocity_reference_grid.npz --output runs/B/S-LSTM-PINN
python -m epjplus_bc C --case 1 --model lstm1 --device cuda --output runs/C/CaseI/VR-LSTM-PINN
```

B defaults to 40,000 updates for all four models. In the steady cavity,
S-LSTM-PINN uses an ordered window of four spatial points. C defaults to
80,000 updates. The B/C package README gives the command to recompute the
eight archived Case IV scores, and its smoke script checks the training
and evaluation paths.

## Experiment F

From the F package:

```sh
python train_all_lstm.py --check-inputs
python train_all_lstm.py --smoke-test --device cuda:0
python train_all_lstm.py --case case3_rbc --model lstm2 --seed 0 --device cuda:0
python train_all_lstm.py --device cuda:0
```

The model keys are `lstm1` for VR-LSTM-PINN and `lstm2` for
S-LSTM-PINN. They share the embedding, two width-192 recurrent layers,
prediction head, optimizer settings, sampling, and frozen data. VR resets
the recurrent state for every physical point. S propagates the state
through each four-point window. The smoke command uses seed 0 and one
update per case and model. The package contains the full-grid smoke report.

## Reference data and paper metrics

- Use the N=256 FEM v4 velocity grid specified in the B/C README for
  Case IV in Experiments B and C. The Case IV F/G inputs contain the
  audited v4 velocities already sampled on the evaluation grid. Preserve
  the package-specific Case IV inputs and their SHA-256 values.
- In F/G, Case III uses the 120 × 120 grid at times 0, 0.5, 1, 1.5, and 2.
  The paper's mean relative L2 is the arithmetic mean of the individual
  field errors, available under `metrics.field` in the F/G final metrics.
  For the main Case IV ranking, average the separate relative errors in
  u and v. Use the two velocity errors for Table S14 and Figure S9 as well.
- D and E include the same full N=256 FEM data file for their evaluators.
  E reports pressure RMSE separately as a diagnostic. The length-8
  timing pilots are separate from the completed length-1/2/4 comparison.
- The archives include training inputs and the B/C Case IV result
  archive. The F and B/C smoke reports summarize validation runs.
  Full training checkpoints and all figures and result histories are
  separate from these release packages.

See the per-package README or run matrix for model keys, data paths,
output directories, and the exact command for each experiment.
