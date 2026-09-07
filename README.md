## Identifiability of Nongravitational Force Recovery from GRACE-FO Satellite Orbits Using Physics-Informed Neural Networks

This is the code for the paper *"Identifiability of Nongravitational Force Recovery
from GRACE-FO Satellite Orbits Using Physics-Informed Neural Networks"* (Reva Paul),
submitted to AGU Earth and Space Science.

The code recovers the three-dimensional nongravitational acceleration on GRACE-FO
satellite C from GPS navigation data alone, over a 14-day arc, using an inverse
physics-informed neural network (iPINN). It then checks that recovered force against
the satellite's independent ACT1B accelerometer. The findings suggest that accurate
orbit reconstruction does not guarantee physically meaningful force recovery, as 
demonstrated by an unconstrained Neural ODE model.

## Contents

- `gracefo_inverse_pinn.ipynb` (§4, §5.1–5.2): the main inverse PINN estimator. This recovers the
  RTN nongravitational acceleration and produces Table 1; also produces the ablation
  rows of Table 2 (see below).
- `act1b_reference.ipynb` (§5.4): builds the bias-bounded ACT1B along-track reference
  (the `ACT1B_FT_REFERENCE` value used for reporting).
- `gracefo_neural_ODE.ipynb` (§4.6, §5.6): the unconstrained Neural ODE baseline
  (Table 5, Figure 5).
- `compare_act1b.ipynb` (§5.3–5.4): compares the recovered in-track force to ACT1B in
  timing and size (Table 3, Figures 2–3).
- `synthetic_validation.ipynb` (§5.5)z: the known-answer test where the true radial
  force is zero by construction (Table 4, Figure 4).
- `propagation_comparison.ipynb` (§5.7): propagates the orbit forward under each
  recovered force (Figure 6).

## Data

The study uses GRACE-FO Level-1B Release v4.0 products for satellite C, spanning
2025-11-01 to 2025-11-14. These belong to NASA/JPL and are not redistributed here;
download them from the PO.DAAC archive:

> NASA JPL (2019), GRACE-FO Level-1B Release v4.0 in ASCII, PO.DAAC.
> https://doi.org/10.5067/GFL1B-ASJ04

You need the GNV1B (GPS navigation) and ACT1B (accelerometer) files. Once downloaded,
put them in a folder called `data/` in the repository root. For example,
`data/GNV1B_2025-11-01_C_04.txt` through `data/GNV1B_2025-11-14_C_04.txt`. That folder
is the default (`CFG.DATA_FOLDER_PATH = "data"`), and it's git-ignored, so nothing you
download gets committed. The notebooks also write their caches, plots, and output
arrays back into the same folder, so you can expect it to fill up as you run things.

## Setup

```bash
# Python 3.10+ recommended
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab                       # or: jupyter notebook
```

Then open a notebook and run all cells (Kernel → Restart & Run All).

## Running order

The notebooks aren't fully independent, since the comparison and propagation notebooks read
force arrays that the model notebooks save to `data/`, rather than recomputing them. So
run them in this order:

1. `act1b_reference.ipynb`
2. `gracefo_inverse_pinn.ipynb`
3. `gracefo_neural_ODE.ipynb`
4. `compare_act1b.ipynb`
5. `synthetic_validation.ipynb`
6. `propagation_comparison.ipynb`

Importantly, the iPINN notebook writes `v7_full_Ft.npy` (and the matching `_Fr`, `_Fn`,
`_time`, `_Ft_std` arrays), and the Neural ODE notebook writes the same under a
`node_v2_` prefix. `compare_act1b.ipynb` needs the `v7_full_*` arrays and will stop with
an error if they're missing. `propagation_comparison.ipynb` needs both sets. The frame
transforms and anchors are cached to `_eci_cache_*.npz` and `_anchor_*.npz` and reused
automatically, so any later runs will skip the slower astropy step.

## Reproducing the ablation table (Table 2)

`gracefo_inverse_pinn.ipynb` produces one row of Table 2 per run, chosen by the
`CFG.ABLATION` string near the top. Set it to `full`, `no_gravity`, `no_rtn`, `no_ode`,
and `no_anchor` in turn and re-run to get all the rows. It's left on `full` by default,
which is the headline Table 1 result.

## Citing this

If you use the code, please cite both the paper and the archived release:

```bibtex
@article{paul_grace_fo_ipinn,
  author = {Paul, Reva},
  title  = {Identifiability of Nongravitational Force Recovery from {GRACE-FO}
            Satellite Orbits Using Physics-Informed Neural Networks},
  year   = {2026},
  note   = {Submitted to AGU Earth and Space Science Journal}
}

@software{paul_grace_fo_ipinn_code,
  author    = {Paul, Reva},
  title     = {Nongravitational force recovery from GRACE-FO orbits with a
               physics-informed neural network},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.22648369}
}
```

## License

MIT — see [LICENSE](LICENSE).
