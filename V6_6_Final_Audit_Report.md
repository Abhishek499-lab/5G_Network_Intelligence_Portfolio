# 5G Network Intelligence — V6.6 Final Audit Report

## Build and reproducibility
- Source archive SHA-256: `abc729d696b5e0ba34a9c6ed35a851f2868d2b5071be5efb8a49691c23bb1a9b`
- Source CSV files: 83; eligible sessions: 82
- Frozen split: 61 training sessions / 21 previously inspected development sessions
- Split seed: 42; GroupShuffleSplit test size: 0.25
- Features: 33
- Models: two HistGradientBoosting regressors and one HistGradientBoosting classifier
- Forecast targets: first sample at or after 5s/15s, maximum overshoot 2.5s

## Development forecasting results

|   horizon_s |   n_rows |   n_sessions |   model_MAE_Mbps |   persistence_MAE_Mbps |   MAE_improvement_pct |   model_RMSE_Mbps |   persistence_RMSE_Mbps |   model_R2 |   negative_raw_prediction_count | evaluation_partition                      |
|------------:|---------:|-------------:|-----------------:|-----------------------:|----------------------:|------------------:|------------------------:|-----------:|--------------------------------:|:------------------------------------------|
|           5 |    38692 |           21 |          10.4068 |                12.7896 |              18.6312  |           27.9622 |                 39.6513 |   0.534323 |                             528 | previously inspected development sessions |
|          15 |    38431 |           21 |          12.0257 |                13.2647 |               9.34078 |           31.6581 |                 39.6077 |   0.411333 |                             475 | previously inspected development sessions |

These results are from the previously inspected development partition. They are not untouched independent validation. Negative raw regressor outputs are clipped to zero in inference.

## Warning proxy results

|   n_rows |   n_sessions |   proxy_prevalence |   ROC_AUC |   PR_AUC_average_precision |   Brier_score |   precision_at_050 |   recall_at_050 |   F1_at_050 |   balanced_accuracy_at_050 |   threshold | score_calibrated_probability   | label_type                                          |
|---------:|-------------:|-------------------:|----------:|---------------------------:|--------------:|-------------------:|----------------:|------------:|---------------------------:|------------:|:-------------------------------|:----------------------------------------------------|
|    34928 |           21 |          0.0971713 |  0.864319 |                   0.432201 |      0.111915 |           0.365929 |        0.701237 |    0.480905 |                   0.785229 |         0.5 | False                          | KPI-derived proxy; not operator-confirmed incidents |

The classifier predicts a KPI-derived proxy target, not operator-confirmed incidents. Its score is not calibrated as a probability. Metrics at threshold 0.50 are descriptive and threshold-dependent.

## Functional smoke test

- Rows scored: 622
- Alert episodes: 12
- Maximum risk score: 0.93697 (uncalibrated)
- Mode: offline uploaded CSV
- Result: software smoke test passed; this is not predictive validation.

## External transfer result retained

V6.5.1 NordicDat external transfer underperformed the persistence baseline at both horizons. This result remains in the portfolio narrative as a documented domain-transfer failure. NordicDat was not used for model fitting or tuning in V6.6.

## Important limitations

- Development sessions were inspected in earlier V5/V6 iterations; no untouched independent source test set remains.
- Original data are client-side measurement traces, not operator gNB counters.
- No CPU/memory counters or operator-confirmed incident labels.
- Warning labels are KPI-derived proxies; scores are uncalibrated.
- Alert explanations are KPI-based hypotheses, not verified root causes.
- Rolling features are sample-window based, not guaranteed elapsed-time windows.
- Inference is offline CSV processing; no Kafka, O-RAN, gNB, or live operator integration.
- NordicDat did not validate transfer performance; it exposed a substantial domain mismatch.

## Release decision
V6.6 is a reproducible offline research/portfolio prototype with an explicitly documented external-transfer limitation. Do not describe it as production-ready or operator-validated network failure prediction.
