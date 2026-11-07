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
use a shared chronological data split and identical test prediction times.

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

## Data availability

The building monitoring data are confidential and cannot be distributed. Private
configuration, trained models and generated results are also excluded. Running the
complete workflow therefore requires authorized access to those inputs.
