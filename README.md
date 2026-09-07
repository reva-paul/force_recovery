# Identifiability of Nongravitational Force Recovery from GRACE-FO Satellite Orbits Using Physics-Informed Neural Networks

Code accompanying the manuscript *"Identifiability of Nongravitational Force Recovery
from GRACE-FO Satellite Orbits Using Physics-Informed Neural Networks"* (Reva Paul),
submitted to AGU *Earth and Space Science*.

This repository recovers the full three-dimensional nongravitational acceleration on
GRACE-FO satellite C from GPS navigation data alone, over a 14-day arc, using an inverse
physics-informed neural network (iPINN), and validates the recovered force against the
independent ACT1B accelerometer product.

## Repository contents

| Notebook | Paper section | Purpose |
|---|---|---|
| `gracefo_inverse_pinn.ipynb` | §4, §5.1–5.2 | Main iPINN estimator: recovers RTN nongravitational acceleration; reproduces Table 1 and the ablation in Table 2 (set `CFG.ABLATION`; see below). |
| `act1b_reference.ipynb` | §5.4 | Builds the maneuver-screened, bias-bounded ACT1B along-track reference used for plots/reporting (the `ACT1B_FT_REFERENCE` value). |
| `compare_act1b.ipynb` | §5.3–5.4 | Timing and magnitude comparison against the ACT1B accelerometer; reproduces Table 3 and Figures 2–3. |
| `gracefo_neural_ODE.ipynb` | §4.6, §5.6 | Unconstrained Neural ODE baseline; reproduces Table 5 and Figure 5. |
| `synthetic_validation.ipynb` | §5.5 | Known-answer synthetic test (true radial force ≡ 0); reproduces Table 4 and Figure 4. |
| `propagation_comparison.ipynb` | §5.7 | Forward-propagation test under each recovered force; reproduces Figure 6. |

### Reproducing the ablation table (Table 2)

`gracefo_inverse_pinn.ipynb` produces one row of the ablation table per run,
selected by the `CFG.ABLATION` string near the top of the notebook. Set it to each
of `full`, `no_gravity`, `no_rtn`, `no_ode`, `no_anchor` in turn and re-run to
regenerate all rows. The default is `full`, which reproduces the headline Table 1
result.

## Data

This study uses **GRACE-FO Level-1B Release v4.0** products for satellite C
(2025-11-01 to 2025-11-14), which are **not redistributed here** and must be obtained
from the NASA/JPL Physical Oceanography Distributed Active Archive Center (PO.DAAC):

- **GNV1B** (GPS navigation solutions) and **ACT1B** (accelerometer product):
  NASA JPL (2019), GRACE-FO Level-1B Release v4.0 in ASCII, PO.DAAC.
  DOI: [10.5067/GFL1B-ASJ04](https://doi.org/10.5067/GFL1B-ASJ04)

After downloading, place the Level-1B files in a local `data/` directory (ignored by
git) and set the data path at the top of each notebook. <!-- TODO: confirm the exact
folder layout your notebooks expect and update this line. -->

## Setup

```bash
# Python 3.10+ recommended
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab                       # or: jupyter notebook
```

Then open each notebook and run all cells (Kernel → Restart & Run All).

## Reproducing the results

Run the notebooks in the following order (the iPINN notebook produces intermediate
outputs used by the comparison and propagation notebooks):

1. `act1b_reference.ipynb` (builds the ACT1B reference used by the comparison)
2. `gracefo_inverse_pinn.ipynb`
3. `gracefo_neural_ODE.ipynb`
4. `compare_act1b.ipynb`
5. `synthetic_validation.ipynb`
6. `propagation_comparison.ipynb`

<!-- All intermediate-file dependencies are documented below. -->

### Why this order matters

The comparison and propagation notebooks read force arrays that the model
notebooks write to the data folder — they do not recompute the forces
themselves:

- `gracefo_inverse_pinn.ipynb` writes `v7_<ablation>_time.npy`, `v7_<ablation>_Fr.npy`,
  `v7_<ablation>_Ft.npy`, `v7_<ablation>_Fn.npy`, and `v7_<ablation>_Ft_std.npy`
  (e.g. `v7_full_Ft.npy` for the default run).
- `gracefo_neural_ODE.ipynb` writes the Neural ODE force arrays under the
  `node_v2_` prefix.
- `compare_act1b.ipynb` loads `v7_full_*.npy` and errors out if they are absent,
  so the iPINN notebook must run first.
- `propagation_comparison.ipynb` loads **both** `v7_full_*.npy` and `node_v2_*.npy`,
  so both model notebooks must run before it.

Frame transforms and anchors are cached (`_eci_cache_*.npz`, `_anchor_*.npz`) in the
data folder and reused automatically on later runs. All of these generated files are
git-ignored.

## Citation

If you use this code, please cite both the paper and the archived code release:

```bibtex
@article{paul_grace_fo_ipinn,
  author  = {Paul, Reva},
  title   = {Identifiability of Nongravitational Force Recovery from {GRACE-FO}
             Satellite Orbits Using Physics-Informed Neural Networks},
  year    = {2026},
  note    = {Submitted to AGU Earth and Space Science Journal}
}
```
<!-- TODO: after archiving to Zenodo, add the software DOI citation here. -->

## License

Released under the MIT License. See [LICENSE](LICENSE).
