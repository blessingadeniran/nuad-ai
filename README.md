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
- [x] Investigated false positives — found early-warning signal (see below)

## Key finding
In every documented failure, the compressor loses its normal on/off duty-cycling and
runs continuously instead — this loss-of-rhythm is the core signal the model is built
around (see `outputs/` for supporting plots). `H1_cycle_transitions` showed a 50–1000x
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

## Early-warning behavior (reframing "false positives")
Of the 13,323 alerts flagged outside the four labeled failure windows, 5,554 (42%)
occurred within 24 hours of a known failure, and 2,287 (17%) within just 6 hours.
This suggests a meaningful share of "false positives" are early-warning signals —
the compressor showing degraded behavior before the officially documented failure
window begins, which the labels do not credit but which is the intended behavior
of a predictive maintenance system. The remaining alerts, scattered further from any
known failure, likely include genuine false positives — real precision remains an
area for improvement with more diverse training data.

## Full pipeline example (single alert, real held-out data)
```
{'severity': 'ESCALATE',
 'top_features': [('Oil_temperature_roll_mean', 3.29), ('TP2_roll_std', 1.55), ('H1_roll_std', 1.43)],
 'flag_rate': 1.0}
```

## Repo structure
```
├── README.md
├── notebooks/
│   └── eda.ipynb
├── data/                  (gitignored — raw + processed CSVs)
├── models/
│   ├── isolation_forest_v1.pkl
│   └── compressor_pipeline_v2.pkl
├── outputs/
│   ├── Failure_1.png ... Failure_4.png
│   ├── sensor_overview.png
│   └── failure_sensor_summary.csv
```

## Known limitations / honest next steps
- Only 4 documented failures exist in this dataset, all air leaks — model is validated
  on this failure type only, not on other failure modes (e.g. oil leaks).
- Precision against strict labeled windows is moderate (10%), though a substantial
  share of flagged alerts appear to be genuine early warnings rather than noise.
- Severity thresholds are calibrated against this dataset's observed distribution,
  not yet validated against real operator judgment.
- Trained and validated on MetroPT-3 (metro compressor data) as a proxy for oil & gas
  compression equipment — same underlying physics (duty-cycling, motor load, leak
  dynamics), but not yet tested on real Renaissance sensor data.
