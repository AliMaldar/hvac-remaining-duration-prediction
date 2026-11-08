# HVAC Remaining-Duration Prediction

This repository provides seven research notebooks for predicting the time remaining
until a room completes an HVAC heating or cooling recovery event. Predictions are
made at startup and updated during the event as measurements become available.

## Prediction approaches

- **Direct ET:** Extra Trees regressors predict remaining duration directly.
- **Temperature ET:** Extra Trees regressors predict future temperature changes;
  the resulting temperature trajectories are used to estimate remaining duration.
- **Shared NN:** One neural network predicts temperature-change rates across rooms
  and both operating modes. Rates are integrated into temperature trajectories,
  with training that includes rate, temperature and duration errors.

The ET models are fitted separately for each room and operating mode. Comparisons
use identical test prediction times and chronological event assignments within each
operating mode. This is an offline, retrospective benchmark: reconstructed inputs
can incorporate later observations, and shared-NN cooling evaluation also uses
heating training/validation records from later calendar dates.

## Notebook workflow

Run the notebooks in numerical order.

| Notebook | Purpose |
|---|---|
| 01 | Clean observations, detect events, construct features and targets, and create the chronological split. |
| 02 | Train the temperature ET models and the shared neural network. |
| 03 | Train direct duration regressors and evaluate the three prediction approaches. |
| 04 | Analyze prediction coverage, event-averaged errors and paired bootstrap uncertainty. |
| 05 | Compare neural model components and training objectives. |
| 06 | Assess sensitivity to duration-calculation settings and observed endpoint definitions. |
| 07 | Evaluate neural training stability across five random seeds. |

## Requirements and execution

The notebooks retain their Google Colab setup and require Python 3.12 or newer,
NumPy, pandas, Matplotlib, SciPy, scikit-learn, joblib and PyTorch. Neural training
benefits from a GPU; Notebook 07 retains the reference GPU and batch-size checks.

Before running, set the private workspace and configuration paths in the notebook
kernel environment:

```python
import os

os.environ["HVAC_PRIVATE_WORKSPACE"] = "/path/to/private/workspace"
os.environ["HVAC_PRIVATE_SETTINGS"] = "/path/to/private/settings.json"
```

Keep both locations outside the public repository. The workspace holds the private
inputs and generated artifacts. The settings file contains the building-specific
configuration and reference values required by the notebooks.

## Input and settings contract

The raw directory is `Datasets/` inside the private workspace. Indoor observations
must include a UTC `timestamp` and `<prefix>_T` (°C) and `<prefix>_V` (0–1)
channels for each selected space. Optional `_S` and `_O` channels are retained as
diagnostics. AHU and plant tables require a timestamp plus the source columns
selected by Notebook 01's candidate lists; preserve their units and meanings.
Generated event tables contain identifiers, start/end times and durations;
duration rows additionally contain the requested time, preceding feature time,
65 state inputs and the unrounded remaining-time label. Trajectory rows add a
positive horizon and temperature/rate targets. Later notebooks consume these
exports and the shared split map; do not construct independent splits.

The JSON file has a top-level `private_values` object. Its keys are literal strings
containing dots, not nested paths. Required settings are read explicitly by
`private_value(...)` in each notebook. Extra legacy settings may remain unused.

| Key or group | Type and meaning |
|---|---|
| `nb01.selected_room_prefixes` | List of sensor prefixes to include. |
| `nb01.rooms_all` | Object mapping private space labels to lists of sensor prefixes. |
| `nb01.heating_room_target_c` | Object mapping prefixes to fixed heating references in °C. |
| `nb01.fixed_cooling_target_c` | Numeric cooling reference in °C. |
| `nb01.building_latitude_deg`, `nb01.building_longitude_deg` | Numeric coordinates for calendar/daylight proxies. |
| `nb01.cooling_season_start`, `nb01.cooling_season_end` | Local-time date/time strings delimiting the cooling cohort. |
| `inputs.literal_002` / `_003` / `_004` / `_005` | Filename strings for AHU / plant / optional valve / indoor observations. Each key includes the `inputs.literal_` prefix. |
| `inputs.literal_010`–`_014` | Source-column names for cold-water return, chiller supply, buffer supply, light proxy and cooling-plant power, respectively. |
| `inputs.literal_001` | Private fallback input-directory name used by Notebook 06. |
| `inputs.literal_015`, `_021`, `_023` | Private diagnostic text or source-description strings used by Notebook 01/06. |
| `shared.notebook03_run_id` | Directory selector for the saved Notebook 03 run. |
| `nb04.expected_full_test`, `nb04.expected_common`, `nb07.expected_full`, `nb07.expected_common` | Per-mode objects containing expected `anchors` and `events` counts for run-integrity checks. |
| `nb04.expected_3dp` | Expected metric checks keyed by tuple; JSON encodes tuple-key dictionaries as `{"__tuple_key_dict__": [[key_list, value], ...]}`. |
| `nb06.expected_event_counts` | Per-mode integer event counts for endpoint-reconstruction checks. |

For example, `"nb01.rooms_all": {"Space_A": ["ZONE_A"]}` is an invented
mapping, not building data. Expected counts/metrics must describe the authorized
cohort and saved run; they are integrity checks, not predictor variables. This
interface description does not supply the confidential settings or create a
publicly runnable reproduction of the reported benchmark.

## Recorded environment and source identity

The saved five-seed experiment records Python 3.13.15, NumPy 2.1.3, pandas 2.2.3,
scikit-learn 1.6.1, joblib 1.6.0 and PyTorch 2.11.0+cu130, CUDA 13.0 and cuDNN
version code 92700, on a Tesla T4 GPU (about 14.6 GB). Mixed precision was enabled;
strict deterministic kernels were disabled. Effective batch sizes were 256
trajectory groups and 256 duration rows per mode. This is the recorded environment
for Notebook 07; a complete separate environment record for the original
seed-42 training was unavailable. Plotting additionally uses Matplotlib and SciPy;
their versions were not recorded in that manifest. Seeded runs need not be bitwise
identical across hardware/software configurations.

Notebook-source SHA-256: `d12e8b8f184b0e99d2bee8655f37c5dc9469f269006a85be3e1bdfabf402fe75`.
This hashes the seven notebook files in filename order, appending the UTF-8
filename, a null byte, the exact file bytes and a null byte for each file. It
identifies this source snapshot independently of README changes. A remote commit
or release tag has not been verified here. The accompanying paper-figure builder
is an internal reporting script and is not part of this eight-file release.

## Data availability

The building monitoring data are confidential and cannot be distributed. Private
configuration, trained models and generated results are also excluded. Running the
complete workflow therefore requires authorized access to those inputs.
