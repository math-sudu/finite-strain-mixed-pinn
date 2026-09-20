# Finite-strain mixed PINN: compact reproduction package

Code and trained parameters for **Path dependent ground response to horseshoe
tunnel unloading using a finite strain mixed PINN**.

Repository: <https://github.com/math-sudu/finite-strain-mixed-pinn>.
Download [PINN_code_data.zip](https://github.com/math-sudu/finite-strain-mixed-pinn/blob/main/PINN_code_data.zip),
extract it, and open a terminal in the extracted `PINN_code_data/` directory.

## Run

Use Python 3.13. The package was checked with the versions in requirements.txt.
A CPU installation of PyTorch is sufficient.

```sh
python -m pip install -r requirements.txt
python verify_release.py
python reproduce.py
```

The verification command checks file hashes and all thirteen retained trained
states without evaluating fields. The reproduction command sequentially
reconstructs their material history, generates the reporting arrays and CSV,
checks regional budgets against small original reference values, and renders
the thirteen figures. It performs no optimization or new loading steps.

To generate data only, use `python reproduce.py --data-only`. CUDA-enabled
PyTorch users may select `--device cuda:0`; CPU is the default. Replot already
generated data with:

```sh
python figures/pinn_mechanism_endpoints_01/render.py
```

Original typography uses Times New Roman, which is not redistributed. An
alternative installed serif font can be selected in tools/figure_style.json.

## What is retained

- `data/model_path.pt` contains the first eight trained network states and the
  reference to the complete field chain.
- `data/fields/` contains five linked field nodes: the Cartesian state at
  pressure 0.725 and four configuration increments. Their numerical parameters
  and field identities are unchanged.
- `data/config.json` and the six-arc geometry define the material, network,
  loading, reporting quadrature and spatial domain.
- `code/` contains the numerical implementation and data-generation functions.
  `source_snapshots/` stores each historical source version once;
  `data/stage_configurations.json` associates stages with those source versions.
- `data/expected_budgets.json` holds a small set of original numerical values
  used to check regeneration. The original three validation observations in
  `data/validation_checks.jsonl` support the recorded-acceptance figure.
- `release.json` contains checksums of the distributed inputs and code.

The retained pressure order is 1.0, 0.98, 0.95, 0.90, 0.85, 0.80, 0.75, 0.7375,
0.725, 0.7125, 0.7000, 0.6875, 0.6750. The first state is precompressed at 1.0.
The configuration increments require their complete ancestor chain.
Material history starts once from the virgin state and advances through all
thirteen states; it is not reset between the four reported endpoints.

The trained parameters avoid repeating the original optimization. Optimizer
states, dense coordinate matrices, intermediate candidates, duplicate source
copies, material-history caches, sampled field arrays, CSV exports and figures
are not distributed. Their absence does not change the retained trained path.
This package regenerates the reported results; it is not an interrupted-training
restart archive.

## Generated data

Outputs are written under figures/pinn_mechanism_endpoints_01/.
`plot_data.npz` contains 71,392 ordered reporting points: 70,656 volume points
and 736 wall points. Coordinates, normals, arc identities and natural area/arc
weights are generated from the original six-arc quadrature code.
`expanded_fields.npz` contains the 130,272-point wider near-field grid.

The four endpoint prefixes e0_ through e3_ correspond to pressures 0.7125,
0.7000, 0.6875 and 0.6750. Displacement u_m is in metres; F, cp and kappa are
dimensionless deformation gradient, plastic metric and accumulated plastic
strain. Stresses use the configured p0 units (p0 = 1), with negative compression.
dk, du_m and dF use each state's actual accepted parent. Category 0 denotes
subthreshold material, 1 previously plastic material and 2 newly plastic
material, using the 1e-7 threshold. Q10 is the fixed frame at pressure 0.7125.

table1.csv contains four endpoints by six regions; the last three blocks are
the eighteen incremental rows reported in the paper. knee_profiles.csv contains
wall profiles, and spatial_support.csv describes spatial support. Volume and
wall statistics use different natural measures (m2 and m); the visualization
grid does not replace the weighted budget quadrature.

The original solve used PyTorch 2.8.0+cu126, float64, deterministic algorithms
and a Tesla PG503-216 GPU. CPU regeneration can differ by floating-point
roundoff; the retained reference budgets provide a direct numerical check.
The repository is private during submission preparation. Public access will be
enabled after the author confirms manuscript submission. No DOI is assigned.
