# AI-Powered-Adaptive-Battery-Thermal-Management-System-ML-model-
# ThermoAI: AI-Powered Adaptive Battery Thermal Management System

ThermoAI is a proof-of-concept project exploring machine-learning-based prediction of lithium-ion battery temperature and using that prediction to inform a cooling recommendation.

The current workflow trains a temperature-prediction model using publicly available NASA battery data. A separate rule-based controller compares the predicted temperature with a user-defined target and recommends whether cooling should be activated, increased, or monitored.

> **Project status:** Proof of concept. The model and cooling recommendations have not been validated on the project's own battery pack or cooling hardware.

## Objectives

- Predict battery temperature 60 seconds into the future.
- Evaluate predictions against held-out battery data.
- Produce an initial cooling recommendation using the prediction and a chosen target temperature.
- Establish a foundation for a future adaptive battery thermal-management prototype.

## Current workflow

1. Download and extract the NASA battery dataset.
2. Parse discharge-cycle measurements from MATLAB `.mat` files.
3. Prepare features including current temperature, voltage, current, ambient temperature, elapsed time, and lagged readings.
4. Train a regression model to predict temperature 60 seconds ahead.
5. Evaluate the model and plot actual versus predicted temperatures and feature importance.
6. Save the trained model.
7. Enter battery readings into an interactive cell to receive a predicted temperature and a rule-based cooling recommendation.

```text
Public battery dataset
        |
        v
Data extraction and preprocessing
        |
        v
ML model: predict temperature in 60 seconds
        |
        v
Compare prediction with chosen target temperature
        |
        v
Rule-based cooling recommendation
```

## Dataset

### NASA Li-ion Battery Aging Dataset

- **Source:** NASA Dashlink
- **Dataset page:** https://c3.ndc.nasa.gov/dashlink/resources/133/
- **Purpose:** Train and evaluate the initial temperature-prediction model.
- **Format:** MATLAB `.mat` files containing battery cycle and measurement data.

The notebook downloads the archive and searches the extracted folders for `.mat` files. If the source URL or archive structure changes, inspect the response and archive contents before continuing.

This is public research data, not measurements from the project's own 18650 2S2P pack or cooling loop.

## Requirements

The notebook uses Python and these libraries:

- NumPy
- pandas
- SciPy
- scikit-learn
- Matplotlib
- requests
- joblib

In Google Colab, install any missing packages:

```python
!pip install numpy pandas scipy scikit-learn matplotlib requests joblib
```

## Run in Google Colab

1. Open Google Colab and create a notebook.
2. Run the setup and imports cell.
3. Run the dataset download/extraction cell and confirm at least one `.mat` file is found.
4. Run the parsing cell and confirm that discharge records are extracted into a non-empty DataFrame.
5. Run preprocessing and feature-engineering cells.
6. Train and evaluate the model.
7. Inspect the actual-versus-predicted plot, feature-importance plot, and numerical metrics.
8. Save the model and feature list.
9. Run the interactive prediction cell.

**Run cells in order.** Do not continue if download, extraction, or parsing fails. If the MATLAB structure differs from the parser's assumptions, inspect the `.mat` file and adjust the parser.

## Interactive prediction

The current interactive cell asks for:

- Current battery temperature (°C)
- Battery voltage (V)
- Battery current (A)
- Ambient temperature (°C)
- Elapsed discharge time (seconds)
- Desired battery temperature target (°C)

It outputs the current temperature, predicted temperature after 60 seconds, the chosen target, and a basic recommendation to activate/increase cooling, monitor, or continue normal monitoring.

**Important:** The recommendation uses simple rules. The model does not predict the ideal coolant temperature, pump speed, or cooling power.

### Input-feature caveat

The trained model uses historical lag features. If the interactive cell fills these features by repeating the current measurement, the input is only an approximation and prediction quality may be reduced. A better version should use actual historical readings from a time-series buffer and ensure that the feature names and order match training exactly.

## Model evaluation

Record the actual results produced by your notebook:

- **MAE:** Mean absolute error, in °C
- **RMSE:** Root mean squared error, in °C
- **R²:** Coefficient of determination

Do not put assumed or fabricated scores in a report. Compare the model against a simple baseline, such as predicting that the temperature 60 seconds later will equal the current temperature. Split data by battery where possible so that test batteries were not used for training.

A plot close to the diagonal is encouraging, but it does not by itself prove generalization or reliable performance.

## Limitations

- The model uses public NASA data, not the project's own battery pack.
- Cell chemistry, test conditions, sampling intervals, and operating profiles may differ from the prototype.
- The model predicts battery temperature, not ideal coolant temperature or pump commands.
- The cooling recommendation is rule-based and has not been validated against a physical cooling system.
- Repeating current readings for lag features is a temporary approximation, not a substitute for real historical data.
- Results must be re-evaluated if the dataset, features, prediction horizon, or model changes.

## Safety

- Do not deliberately overheat, short-circuit, or stall a lithium-ion battery to generate data.
- Operate cells within manufacturer-specified voltage, current, and temperature limits.
- Use suitable cell protection/BMS, fusing, current limiting, and supervised testing.
- Keep coolant and electrical connections appropriately isolated.
- Do not use this proof-of-concept model as a safety device or as the sole protection against overheating or thermal runaway. Retain independent hardware protection and a safe shutdown strategy.

## Future work

- Document the exact dataset files and cycles used.
- Add baseline comparisons and robust battery-level validation.
- Use real time-series history for lag features.
- Examine public thermal-management datasets that include coolant temperatures and cooling-state measurements.
- Develop and validate the cooling controller separately from the prediction model.
- Integrate sensors, controller, pump driver, cooling plate, and hardware protections only after safe bench validation.

## References

1. NASA, *Li-ion Battery Aging Datasets*, NASA Dashlink: https://c3.ndc.nasa.gov/dashlink/resources/133/
2. scikit-learn documentation: https://scikit-learn.org/stable/
3. SciPy MATLAB file I/O documentation: https://docs.scipy.org/doc/scipy/reference/io.html


## Model Performance and Evaluation

The ThermoAI temperature prediction model was evaluated on held-out test data using regression metrics and inference latency measurements.

### Prediction Performance

| Metric | Result |
|---|---:|
| Mean Absolute Error (MAE) | 0.1696 °C |
| Root Mean Squared Error (RMSE) | 0.3462 °C |
| R² Score | 0.9995 |
| Predictions within ±1 °C | 97.86% |
| Predictions within ±2 °C | 99.49% |

### Inference Latency

| Metric | Result |
|---|---:|
| Median latency | 101.7951 ms |
| Mean latency | 101.8069 ms |
| 95th-percentile latency | 114.3251 ms |
| Maximum observed latency | 319.4423 ms |

Latency was measured in the Google Colab environment using repeated single-sample prediction calls. These values represent model inference time in that environment and may differ on other hardware.

### Interpretation

- **Low MAE:** The model's average absolute prediction error was approximately 0.17 °C on the evaluated test data.
- **Low RMSE:** The model showed relatively small prediction errors, including a penalty for larger errors.
- **High R²:** The model explained a very high proportion of the variation in the test-set temperature targets.
- **Fast inference:** The median measured inference time was approximately 102 ms per prediction call.

### Important Limitations

These are preliminary results obtained using a public NASA battery dataset. They do not establish equivalent performance on the project's own 18650 battery pack or cooling system.

Before treating these metrics as final, validate the model against a persistence baseline (predicting that the future temperature equals the current temperature) and verify that training and testing are separated by battery and that no future information leaks into the input features.

The R² score is a regression metric and should not be interpreted as classification accuracy.

The cooling recommendation is currently rule-based. The model predicts battery temperature; it does not directly predict the optimal coolant temperature or pump speed.

