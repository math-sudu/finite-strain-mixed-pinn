# Finite-strain tunnel creep: code and data

Code and data for **A condensed physics informed neural network coupling implicit
constitutive evolution with weak equilibrium for finite strain tunnel creep**.
The package covers the noncircular PINN comparison, its material inputs and the
field application in the article.

Download [PINN_code_data.zip](https://github.com/math-sudu/finite-strain-mixed-pinn/raw/refs/heads/main/PINN_code_data.zip)
and extract it into an empty directory.

Both benchmark PINNs integrate the same viscous history locally and use its
current-step derivative. The condensed solver evaluates equilibrium with the
returned material stress; the mixed solver uses an auxiliary stress output and
a constitutive consistency penalty. The paired histories assess displacement
accuracy and the number of objective evaluations. The pressure-dependent material fit at r=0.10
supplies the noncircular benchmark conditions. The field calculation uses its
own calibrated directional material and fixed-feature neural Galerkin space,
sharing the local-return and equilibrium construction.

Use Python 3.13 and install the packages in `requirements.txt`. Run commands
from this extracted directory. Numerical runs use CPU float64 and one compute
thread. Iterative optimization paths and elapsed times can vary with the platform.

```text
python -m pip install -r requirements.txt
python reproduce.py laboratory --list
python reproduce.py laboratory
python reproduce.py horseshoe
python reproduce.py field
python reproduce.py field-response
```

`--list` prints the commands without running them. Each group performs its
full calculations and corresponding comparisons. `laboratory` reproduces the
pressure-dependent phyllite fit at the prescribed relaxed shear fraction r=0.10
and evaluates the corresponding specimen histories. `horseshoe` includes the
paired two-seed comparison and refined FE histories at r=0.10.
`field` calibrates six physical parameters using the 55 readings through
day 14.5277778 under `field_calibration_horseshoe.json`, then evaluates the
continuing cumulative response. Every candidate
solves the complete neural equilibrium and material history from its own
relaxed state. The field calculation uses the documented +860 horseshoe,
with ground contact on the initial-support outer contour and monitoring on
the inner contour through a normal displacement mapping. It retains the
nearly elastic matrix fraction and the assumed days-13-to-15 bottom-core removal.
Each group writes under `results/history_creep/`.

The four observables use their actual fractional timestamps and each
instrument's own first-reading reference. The subsequent cumulative evaluation
contains the 43 readings covered by the saved calibrated history. All 99 source
readings remain in the data; the final SL01 reading at day 25.055556 lies beyond
that common evaluation interval.

The cumulative percentage error is the sum of absolute differences between
calculated and measured displacement, divided by the sum of absolute measured
displacement from the channel's first reading, multiplied by 100. Calibration
and later periods are evaluated separately. Laboratory percentages retain
the measured hold-increment denominator.

`field-response` summarizes the saved calculated values at the actual reading
times without fitting or solving equilibrium again. It writes
`results/history_creep/field/pinn_horseshoe/cumulative_response.json`.
Engineering-model choices used the monitored record, so this is a retrospective
evaluation conditioned on subsequent measured pressure and its stated coverage
extensions. Original measurements are unchanged. Use a fresh extraction for
a new complete run; field and horseshoe solvers reject existing output histories.
Individual scripts expose their options through `--help`.

The file `parameters/m2_r010.json` supplies the reported material for the
horseshoe comparison. These parameters are fixed, so the spatial comparison can
run independently of the laboratory fitting group. Each neural trajectory
starts with its own relaxed material state; no FE labels or FE states are
supplied. The retained-core field calibration starts every neural solution from its own
relaxed state without external network initialization. Every candidate recomputes its complete
material history with its own parameters. Accepted fields and material
states are then saved at every physical step. The noncircular comparisons use
independently computed FE trajectories at matching times.

`code/history_material.py` supplies shared stress, elasticity and spectral
operations. `history_isochoric.py` implements the elastic volume and shear branches
with constant or stress-dependent mobility, including the local implicit update and its current-step
derivative. `history_horseshoe.py` couples it to the condensed and mixed weak
PINNs or Q2 FE. `history_field.py` applies spatially variable measured pressure
to the source section, using `field_geometry.py` for the six arcs, monitoring
correspondence and contact-pressure arclength. `history_field_core.py` couples the retained core
and exterior through continuous neural Galerkin features, solving for their output
coefficients and evolving finite-strain material states. `calibrate_history_field.py`
fits the effective field parameters, and `summarize_field_response.py` evaluates
the cumulative response. `isochoric_triaxial.py` independently integrates
stress-controlled specimen histories for fitting. The comparison programs
calculate displacement errors and objective-evaluation counts for the numerical
comparison and cumulative errors for the field response.

See `DATA_SOURCES.md` for the provenance, units and uses of the
measurements. The laboratory and field materials are distinct. The calibrated
field model uses the 10.82 m high, 9.519 m wide design horseshoe and a relaxing
directional branch based on individual carbonaceous-slate specimens. Its pressure input
is zero before first coverage and held after last coverage; contact acts once,
independently of core removal. These extrapolation assumptions and negative
reading clipping affect only derived inputs, with original readings retained.
