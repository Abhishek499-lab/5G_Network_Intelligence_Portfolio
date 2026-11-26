# Technical Report — 5G Network Intelligence V6.6

## 1. Objective

Build an offline prototype that estimates short-horizon downlink throughput, scores a near-term KPI-defined degradation proxy, and groups persistent positive scores into alert episodes.

## 2. Data and split

The source archive contains 83 CSV files, of which 82 sessions were eligible after the feature-pipeline requirements. A deterministic group split (seed 42, test size 0.25) produced 61 training sessions and 21 development sessions. The 21 development sessions had already been inspected in earlier project iterations; therefore they are not an untouched final test set.

The traces are client-side measurements. They are not operator gNB telemetry and do not provide operator-confirmed incident labels or verified fault causes.

## 3. Features and models

The pipeline constructs 33 predictors from downlink/uplink throughput, RSRP, RSRQ, SNR, CQI, RSSI, speed, ping, packet loss, handover indicators, rolling sample-window statistics, lags, changes and relative throughput. The model family is HistGradientBoosting: two regressors for nominal 5-second and 15-second throughput forecasts, and one classifier for the KPI-derived warning proxy.

The forecast target is the first observation at or after the nominal horizon, allowing at most 2.5 seconds of overshoot. Rolling features remain sample-window-based and are not guaranteed to represent exact elapsed-time windows.

## 4. Forecasting results

| Horizon | Rows | Sessions | Model MAE | Persistence MAE | MAE improvement | Model RMSE | Persistence RMSE | R² |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 5 s | 38,692 | 21 | 10.4068 Mbps | 12.7896 Mbps | 18.6312% | 27.9622 Mbps | 39.6513 Mbps | 0.5343 |
| 15 s | 38,431 | 21 | 12.0257 Mbps | 13.2647 Mbps | 9.3408% | 31.6581 Mbps | 39.6077 Mbps | 0.4113 |

These results are retrospective development metrics and must not be presented as independent validation.

## 5. Warning proxy results

The proxy is defined using downlink below 5 Mbps together with weak RSRP or SNR, sustained for at least 3 seconds, with an onset target in the next 10 seconds. On 34,928 rows across 21 development sessions, prevalence was 9.72%, ROC-AUC 0.8643, average precision 0.4322, Brier score 0.1119, precision at score threshold 0.50 was 0.3659, recall 0.7012, F1 0.4809, and balanced accuracy 0.7852.

These metrics describe prediction of a constructed KPI proxy. They do not establish prediction of real operator faults. The score is not calibrated as a probability; threshold-dependent metrics are descriptive.

## 6. Smoke test

The current uploaded original-format smoke-test CSV yielded 622 scored rows, 206 threshold-positive rows and 12 alert episodes. The V6.6 bundle reference output reports 202 threshold-positive rows and the same 12 alert episodes. This four-row discrepancy remains an artifact-level inconsistency; regenerate all smoke-test artifacts from one frozen run before publishing a single canonical count. Ping and packet-loss were unavailable throughout this smoke-test file. This confirms that the inference and alert-generation code executed on that file; it does not measure forecasting accuracy because future target observations were not provided in the smoke test.

## 7. External transfer experiment

The NordicDat experiment was kept separate from fitting and tuning. The V6.5.1 audit found that the source model underperformed the persistence baseline at both horizons. The experiment also exposed a large throughput-distribution difference and a proxy mapping of NordicDat SINR into the source SNR feature slot. The failure is retained in the project record rather than hidden or used as a tuning target.

## 8. Limitations and appropriate claims

Appropriate: offline client-measurement CSV prototype; short-horizon throughput forecasting; KPI-proxy degradation scoring; persistence-based alert episode grouping; exploratory diagnostic hints; reproducible source-only model fitting.

Not appropriate: production-ready network monitoring; live/real-time operator integration; gNB/O-RAN counter analytics; operator-confirmed failure prediction; calibrated failure probabilities; verified root-cause analysis; robust cross-domain generalisation.

## 9. Conclusion

V6.6 is a useful engineering and research portfolio prototype because it includes a reproducible pipeline, model evaluation against a persistence baseline, explicit proxy-label semantics, an offline inference smoke test, and a documented external transfer failure. Its strongest contribution is a transparent, testable workflow with clear limits—not a claim of production-grade 5G fault prediction.
