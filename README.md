# Predicting Baseline Car Setups for Unseen Circuits

MSc thesis, Cranfield University, 2025. A proof of concept for predicting good
starting setups at a track a team has never visited, by treating the circuit as
a combination of sections from tracks with known telemetry and setup data.

**Status:** research prototype. The pipeline runs end to end and the early
stages work well, but the final setup windows are too wide to be actionable
with the data available (see Results).


## Motivation

At national and regional levels, setup decisions mostly rely on race engineers'
experience, often justified by the idea that the available data is too poor for
predictive tools. This project tests whether section-level data from known
tracks can support setup prediction at a new one.


## Pipeline

1. **Target track section prediction.** The track is split into sections, each
   running from one braking point to the next (a corner plus its preceding
   straight). Geometry features (distance, heading, turning angle) come from
   track coordinates. A Random Forest Regressor, trained on preceding straight
   length and corner severity from the training tracks, predicts braking
   distance and so places each section start. This runs in two passes: detect
   geometric corners, then shift each start back to the likely braking point.
2. **Section comparison.** Each section is described by a 9-dimensional feature
   vector (entry and exit straight length, cornering percentage, corner radius
   statistics, elevation change). Features are min-max normalised, and lengths
   and radii are log-scaled so that differences between small values count for
   more than the same differences between large ones. A weighted Euclidean
   distance gives a similarity score against every training section, with a hard
   filter on the number of direction changes so simple corners are never matched
   to complex chicanes.
3. **Performance optimisation.** A Gaussian Process Regression model (Matern
   kernel) is trained per target section. Inputs are six composite setup
   parameters (roll stiffness, roll distribution, pitch damping, heave damping,
   downforce, centre of pressure); the output is a section speed metric. The
   similarity score from step 2 sets the per-sample noise, so data from closely
   matching sections constrains the model more. Predictions are made over a
   "smart grid" bounded around the observed setups, summed into a lap
   prediction, filtered to a 1% tolerance of the best, and clustered into
   distinct setup windows.
4. **Report.** For each target section and for the whole lap: the setup
   windows, the best predicted time, and its delta to the overall best.


## Experiment

- **Training tracks:** Brands Hatch GP, Donington GP, Hungaroring, Silverstone
  GP, Spa, Zandvoort. **Target track:** Monza.
- **Data:** [12 setups, each with one telemetry lap, across the six tracks:
  confirm wording]. 14 of the 57 training sections were matched to Monza
  sections.


## Results

| Stage | Outcome |
|---|---|
| Section prediction | Close to the actual braking, throttle-lift and steering events; braking distance probably overestimated in two Monza sections. |
| Section comparison | Validated by using Silverstone as both training and target track: every section matched its counterpart at 100%. Monza matches scored 69-87%, with long and short corners matching well and the chicane matching poorly. |
| Setup optimisation | Physically sensible trends (fast corners favoured higher downforce and a more rearward centre of pressure), but most parameters were insensitive: windows were too wide, one containing 3,584 setups. |
| Lap time | Predicted Monza lap time within about 5% of regional-series fastest laps. |

**Conclusion:** The method is viable as a pipeline but not yet reliable. The
main suspected cause is data quantity (twelve setups for six variables), not
data quality.


## Limitations and next steps

- **More data:** more laps per setup and more tracks. A series like GB3 has
  50+ setup and track combinations available.
- **Single-track mode:** drop the section step and optimise whole laps on one
  circuit, which removes the need for a large cross-track dataset.
- **Better time deltas:** use combined best sectors rather than a whole-lap
  delta spread by time share, and control for tyre condition.
- **Similarity metric:** feature engineering struggled with shape similarity;
  try curvature-signal (Fourier) or direct geometric matching.
- **Optimisation target:** the section speed metric looks brittle and should be
  replaced or validated.
- **Section definition:** the distance thresholds are track-dependent, so full
  automation was not achieved.


## Data

- Telemetry and setup data came from a team in a Formula feeder series and is
  confidential, so it is **not included**. Setup parameters were normalised and
  combined into the six composite parameters to protect the team's IP.
- Track coordinates come from Assetto Corsa track files, extracted with an
  adaptation of the open-source Blender add-on ioImportACCSV by lebluem. Game
  files are not redistributed here. (https://github.com/leBluem/io_import_accsv)
- To run the pipeline without the real data, set `USE_DUMMY_DATA = True` in the
  `Tuning` class. This creates virtual setup data and telemetry for each track
  so that the process 


## Tech

Python, NumPy, pandas, scikit-learn (Random Forest Regression, Gaussian Process
Regression).

## Author

José Albuquerque Vale, LinkedIn: josealbuquerquevale, jose.a.vale@outlook.com 
