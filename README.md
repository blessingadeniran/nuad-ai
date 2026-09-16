# NAUD: Compressor Anomaly Detection (Model Component)

Predictive/anomaly detection model for Renaissance Innovation Week 2026, Challenge 4.
This repo covers the ML component: data, EDA, feature engineering, model training,
and the full detect → explain → classify pipeline.

## Dataset
MetroPT-3 (UCI #791) — real industrial air compressor sensor data, ~1.5M rows,
Feb-Sep 2020, 4 documented air leak failure events. Raw CSV is gitignored due to size;
download from https://archive.ics.uci.edu/dataset/791/metropt%2B3%2B

## Status
- [x] EDA complete — confirmed failure windows, quantified sensor deviation
- [x] Feature engineering — validated rolling-window features (see outputs/)
- [x] Model training (Isolation Forest) — trained on normal data, tested on held-out Failure 4
- [x] Evaluation — see Results below
- [x] Evidence/explanation layer — per-alert feature attribution
- [x] Severity classification — MONITOR / INVESTIGATE / ESCALATE, thresholds set from
      real data quantiles (25th/75th percentile of deviation magnitude)
- [x] Packaged into a reusable pipeline class (`CompressorAnomalyPipeline`)
- [x] Validated across all 4 documented failures

## Key finding
In every documented failure, the compressor loses its normal on/off duty-cycling and
runs continuously instead — this loss-of-rhythm is the core signal the model is built
around (see `outputs/` for supporting plots). `H1_cycle_transitions` showed a 50-1000x
drop during failures compared to normal operation, consistent across all 4 events.

## Model & Evaluation
Isolation Forest, trained only on normal operation data (never shown failure examples),
tested on Failure 4 — an event fully held out from training:

- Raw predictions (5th percentile anomaly-score threshold): 83% recall, 6% precision
- With persistence smoothing (flag only if ≥50% of last 10 min anomalous): **90% recall, 10% precision**

Persistence smoothing improved both recall and precision by filtering single-row noise,
keeping only sustained anomalous behavior.

## Pipeline validated across all 4 documented failures
| Failure | Total alerts | ESCALATE | INVESTIGATE | MONITOR |
|---|---|---|---|---|
| 1 | 8,537 | 31 | 8,376 | 130 |
| 2 | 2,360 | 0 | 2,356 | 4 |
| 3 | 17,195 | 73 | 16,976 | 146 |
| 4 (held-out) | 1,465 | 1,378 | 87 | 0 |

Severity thresholds (ESCALATE ≥3.2 std devs, INVESTIGATE ≥2.0, MONITOR below) were set
from the actual 75th/25th percentiles of deviation magnitude observed across the full
test set — not arbitrary guesses. The held-out failure (4) is classified almost entirely
ESCALATE, consistent with it showing the largest oil temperature deviation (+3.17 std)
of all four failures — the severity layer tracks real differences in failure intensity.

## Full pipeline example (single alert, real held-out data)
{'severity': 'ESCALATE',
 'top_features': [('Oil_temperature_roll_mean', 3.29), ('TP2_roll_std', 1.55), ('H1_roll_std', 1.43)],
 'flag_rate': 1.0}

## Known limitations / honest next steps
- Only 4 documented failures exist in this dataset, all air leaks — model is validated
  on this failure type only, not on other failure modes (e.g. oil leaks).
- Precision remains moderate (10%) — real deployment would generate meaningful false
  alarms alongside true detections; acceptable trade-off given recall priority, but
  worth improving with more diverse training data.
- Severity thresholds are calibrated against this dataset's observed distribution,
  not yet validated against real operator judgment.