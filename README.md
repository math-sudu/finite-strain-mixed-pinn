# Finite-strain tunnel creep: code and data

Code and data for **A condensed physics informed neural network coupling implicit
constitutive evolution with weak equilibrium for finite strain tunnel creep**.
The package covers the material, noncircular and field comparisons in the article.

Download [PINN_code_data.zip](https://github.com/math-sudu/finite-strain-mixed-pinn/raw/refs/heads/main/PINN_code_data.zip)
and extract it into an empty directory.

Both benchmark PINNs integrate the same viscous history locally and use its
current-step derivative. The condensed solver evaluates equilibrium with the
returned material stress; the mixed solver uses an auxiliary stress output and
a constitutive consistency penalty. The paired histories assess the displacement
and recorded-cost consequences of this choice. The laboratory M2 fits supply
the noncircular benchmark conditions. The field calculation instead uses its
own calibrated directional material and fixed-feature neural Galerkin space,
sharing the local-return and equilibrium construction.

Use Python 3.13 and install the packages in `requirements.txt`. Run commands
from this extracted directory. Numerical runs use CPU float64 and one compute
thread. The recorded times in the article refer to its original environment;
elapsed times and iterative optimization paths can vary with the platform.

```text
python -m pip install -r requirements.txt
python reproduce.py laboratory --list
python reproduce.py laboratory
python reproduce.py horseshoe
python reproduce.py field
python reproduce.py field-validation
```

`--list` prints the commands without running them. Each group performs its
full calculations and corresponding comparisons. `laboratory` recomputes all
conditional fits and all-stage
predictions. `horseshoe` includes the paired two-seed comparison, refined FE
histories, the additional r=0.45 condition and the matched M0 history comparison.
`field` calibrates six physical parameters using the 55 readings through
day 14.5277778 under `field_calibration_horseshoe.json`, then evaluates the
continuing response with `field_validation_horseshoe.json`. Every candidate
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

Five-day conditional forecasts start at days 15, 16, 17, 18 and 19. Each adds
the calculated change after the channel's latest available reading to that
measured value. The calibrated material and deformation history continues
through the window. Persistence holds the latest reading, and recent trend
uses a least-squares slope from the preceding five days. All methods use the
same target readings and last-reading references.

The reported forecast percentage is the average absolute prediction error,
with equal weight for the five origins, divided by the correspondingly averaged
absolute measured increment from the same last readings. Persistence is
100 percent. Cumulative errors use measured displacement from the first reading;
the final signed percentage describes endpoint bias. Laboratory percentages
retain the measured hold-increment denominator.

`field-validation` evaluates the saved calibrated history without fitting or
solving equilibrium again. It writes its outputs under
`results/history_creep/field/pinn_horseshoe/validation/`.
Engineering-model choices used the monitored record, so this is a retrospective
evaluation conditioned on subsequent measured pressure and its stated coverage
extensions. Original measurements are unchanged. Use a fresh extraction for
a new complete run; field and horseshoe solvers reject existing output histories.
Individual scripts expose their options through `--help`.

The three files in `parameters/` supply the reported conditional materials for
spatial calculations. They are fixed input parameters, so spatial groups can
run independently of the laboratory fitting group. Each neural trajectory
starts with its own relaxed material state; no FE labels or FE states are
supplied. The retained-core field calibration starts every neural solution from its own
relaxed state without external network initialization. Every candidate recomputes its complete
material history with its own parameters. Accepted fields and material
states are then saved at every physical step. The noncircular comparisons use
independently computed FE trajectories at matching times.

The r=0.45 horseshoe trajectory uses learning rate 1 for the first 6 h,
rate 0.1 through 48 h, rate 1 through 54 h, and objective scale 1000 thereafter.
The continuation commands read only that trajectory's own saved
states. This trajectory is used for accuracy and response; the paired timing
comparison uses the common r=0.10 settings.

`code/history_material.py` supplies shared stress, elasticity and spectral
operations. `history_isochoric.py` implements the common elastic-volume
M0/M1/M2 family, including the local implicit update and its current-step
derivative. `history_horseshoe.py` couples it to the condensed and mixed weak
PINNs or Q2 FE. `history_field.py` applies spatially variable measured pressure
to the source section, using `field_geometry.py` for the six arcs, monitoring
correspondence and contact-pressure arclength. `history_field_core.py` couples the retained core
and exterior through continuous neural Galerkin features, solving for their output
coefficients and evolving finite-strain material states. `calibrate_history_field.py`
fits the effective field parameters, and `summarize_history_field.py` evaluates
cumulative response and conditional increments. `isochoric_triaxial.py` independently integrates
stress-controlled specimen histories for fitting. The comparison programs
calculate the article's displacement, stress, viscous-state and observation
metrics; they do not generate figures.

See `DATA_SOURCES.md` for the provenance, units and uses of the
measurements. The laboratory and field materials are distinct. The calibrated
field model uses the 10.82 m high, 9.519 m wide design horseshoe and a relaxing
directional branch based on individual carbonaceous-slate specimens. Its pressure input
is zero before first coverage and held after last coverage; contact acts once,
independently of core removal. These extrapolation assumptions and negative
reading clipping affect only derived inputs, with original readings retained.
