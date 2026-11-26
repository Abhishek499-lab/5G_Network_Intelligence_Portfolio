# Model Card — V6.6

## Intended use

Educational, research and portfolio demonstration of offline client-side 5G measurement analysis. Suitable for experimentation with short-horizon throughput forecasting and KPI-derived warning proxies.

## Out-of-scope use

Do not use for production network operations, SLA enforcement, emergency response, operator fault declaration, automated remediation, or customer-impact decisions.

## Models

- `forecast_5s.joblib`: HistGradientBoostingRegressor.
- `forecast_15s.joblib`: HistGradientBoostingRegressor.
- `degradation_risk_model.joblib`: HistGradientBoostingClassifier.

## Training/evaluation data

Source client-side measurement archive; 61 training sessions and 21 previously inspected development sessions. The development set is not an untouched test set.

## Target definitions

Forecast target: first observed downlink throughput at or after 5 or 15 seconds, allowing up to 2.5 seconds overshoot.

Warning target: a constructed KPI proxy based on downlink below 5 Mbps plus weak RSRP or SNR, sustained for at least 3 seconds, with an onset in the next 10 seconds.

## Performance

Forecasting and warning metrics are recorded in `../results/`. See the technical report for definitions and caveats. NordicDat external transfer underperformed persistence.

## Known limitations

- Client-side traces, not operator counters.
- Proxy labels, not verified incidents.
- Uncalibrated risk score.
- KPI explanation hints, not confirmed causes.
- Sample-window rolling features.
- No live streaming integration.
- Previously inspected development partition.
- Poor cross-domain transfer on NordicDat.

## Ethical and operational note

A threshold-positive prediction is not a confirmed network failure. Human review and independent validation are required before any operational interpretation.
