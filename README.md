# NAUD: Compressor Anomaly Detection (Model Component)

Predictive/anomaly detection model for Renaissance Innovation Week 2026, Challenge 4.
This repo covers the ML component: data, EDA, feature engineering, and model training.

## Dataset
MetroPT-3 (UCI #791)  real industrial air compressor sensor data, ~1.5M rows,
Feb-Sep 2020, 4 documented air leak failure events. Raw CSV is gitignored due to size;
download from https://archive.ics.uci.edu/dataset/791/metropt%2B3%2B

## Status
- [x] EDA complete - confirmed failure windows, quantified sensor deviation
- [x] Feature engineering - validated rolling-window features (see outputs/)
- [ ] Model training (Isolation Forest)
- [ ] Evaluation
- [ ] Evidence/explanation layer

## Key finding
Compressor loses normal on/off duty-cycling during every failure event - this is the
core signal the model is built around. See outputs/ for supporting plots.
