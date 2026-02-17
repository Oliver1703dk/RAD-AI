# Model Registry View

> **Extends:** arc42 §5 — Building Block View

## Purpose

The SUM platform treats ML models as first-class architectural building blocks with their own lifecycle, versioning, and dependency graph. Unlike traditional software components that evolve through code commits, these models evolve through retraining on new data, hyperparameter adjustments, and dataset version changes. This view captures the complete model inventory, version history, training lineage, and performance baselines to ensure architectural traceability across retraining cycles.

## Model Inventory

| Model ID | Name | Task | Framework | Current Version | Status |
|----------|------|------|-----------|----------------|--------|
| MDL-001 | Route Optimizer | Multi-objective route ranking (regression + ranking) | XGBoost 2.0 (Gradient Boosted Machines) | v2.1 | Production |
| MDL-002 | Demand Predictor | Time-series forecasting | PyTorch 2.1 (LSTM architecture) | v1.4 | Production |
| MDL-003 | Anomaly Detector | Anomaly detection (ensemble) | scikit-learn 1.4 (Isolation Forest) + custom rule engine | v3.0 | Production |

## Model Details

### Route Optimizer (MDL-001)

**Purpose:** Serves as the core routing intelligence of the SUM platform. Given a user's origin, destination, and preferences, the model ranks candidate multimodal routes by predicted travel time, cost, comfort, and carbon footprint. It acts as the primary decision component for the passenger-facing route recommendation interface and the fleet allocation optimizer.

**Versioning:**

| Version | Training Date | Dataset Version | Key Metrics | Deployment Date | Notes |
|---------|--------------|-----------------|-------------|----------------|-------|
| v1.0 | 2024-03-15 | DS-ROUTE-v1 | Accuracy: 0.87, NDCG@5: 0.82 | 2024-04-01 | Initial release; bus and metro modes only |
| v1.5 | 2024-08-20 | DS-ROUTE-v2 | Accuracy: 0.90, NDCG@5: 0.86 | 2024-09-05 | Added ride-sharing and micro-mobility modes |
| v2.0 | 2025-04-10 | DS-ROUTE-v3 | Accuracy: 0.92, NDCG@5: 0.89 | 2025-04-25 | Weather features integrated; retrained with 14 months of multi-modal data |
| v2.1 | 2025-11-12 | DS-ROUTE-v4 | Accuracy: 0.93, NDCG@5: 0.91 | 2025-11-28 | Incorporated demand predictions from MDL-002 as input feature; carbon scoring recalibrated |

**Hyperparameters:**

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| n_estimators | 800 | Bayesian optimization over [200, 1500]; diminishing returns beyond 800 |
| max_depth | 8 | Cross-validated; deeper trees overfit on low-traffic zones |
| learning_rate | 0.05 | Standard schedule; validated against 0.01 and 0.1 alternatives |
| subsample | 0.8 | Reduces variance on heterogeneous zone data |
| colsample_bytree | 0.7 | Decorrelates trees given correlated traffic features |
| min_child_weight | 5 | Prevents leaf splits on rare mode combinations |

**Training Data Lineage:**

| Dataset | Source | Records | Features | Refresh Cadence |
|---------|--------|---------|----------|----------------|
| DS-ROUTE-v4 | Aggregated trip records from 3 transit operators, municipal traffic API, OpenWeather historical, GTFS schedules | 12.4M trip records | 47 features | Quarterly retrain; daily feature store refresh |
| Weather features | OpenWeather historical API | 730 days hourly | 8 features (temp, precip, wind, visibility, etc.) | Included in quarterly retrain dataset |
| Demand context | MDL-002 inference outputs (offline batch) | 72h forecast windows | 6 features (demand by mode/zone) | Daily refresh in feature store |

**Dependencies:**

- Upstream: Municipal traffic API (real-time), OpenWeather API (forecast + current), GTFS schedule database, MDL-002 demand forecasts, feature store (zone-level aggregates)
- Downstream: Passenger Information Display service, mobile app routing API, fleet allocation optimizer, operator dashboard

**Performance Baselines:**

| Metric | Baseline (v1.0) | Current (v2.1) | Alert Threshold |
|--------|----------|---------|----------------|
| Accuracy (top-1 route matches user choice) | 0.87 | 0.93 | < 0.88 |
| NDCG@5 (ranking quality) | 0.82 | 0.91 | < 0.84 |
| Mean ETA error (minutes) | 4.8 | 2.9 | > 5.0 |
| P95 inference latency | 120ms | 85ms | > 200ms |

---

### Demand Predictor (MDL-002)

**Purpose:** Provides 72-hour demand forecasts per transport mode (bus, metro, tram, ride-share, e-scooter), geographic zone (48 hexagonal zones covering the metro area), and 30-minute time window. These forecasts drive fleet pre-positioning, capacity planning, and serve as input features to MDL-001 for demand-aware routing.

**Versioning:**

| Version | Training Date | Dataset Version | Key Metrics | Deployment Date | Notes |
|---------|--------------|-----------------|-------------|----------------|-------|
| v1.0 | 2024-06-01 | DS-DEMAND-v1 | MAPE: 11.5%, RMSE: 142 | 2024-06-20 | Initial release; bus and metro only |
| v1.2 | 2024-11-15 | DS-DEMAND-v2 | MAPE: 9.8%, RMSE: 118 | 2024-12-01 | Added event calendar features; expanded to all 5 transport modes |
| v1.3 | 2025-05-22 | DS-DEMAND-v3 | MAPE: 8.9%, RMSE: 104 | 2025-06-05 | School/holiday schedule encoding; attention mechanism added |
| v1.4 | 2025-10-08 | DS-DEMAND-v4 | MAPE: 8.2%, RMSE: 96 | 2025-10-22 | Weather forecast integration; zone-level demographic weighting |

**Hyperparameters:**

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| hidden_size | 128 | Grid search over [64, 128, 256]; 128 balances accuracy and latency |
| num_layers | 3 | Ablation study showed marginal gain from 4th layer at 40% latency cost |
| dropout | 0.2 | Prevents overfitting on low-traffic zones with sparse data |
| sequence_length | 336 (7 days × 48 slots) | Captures weekly periodicity in commuter patterns |
| learning_rate | 0.001 | Adam optimizer; cosine annealing schedule over 100 epochs |
| batch_size | 256 | Memory-constrained on GPU allocation; 256 fits L4 GPU |

**Training Data Lineage:**

| Dataset | Source | Records | Features | Refresh Cadence |
|---------|--------|---------|----------|----------------|
| DS-DEMAND-v4 | Fare-gate counts, app trip completions, operator dispatch logs | 38.2M demand observations (3 years) | 32 features | Monthly retrain; hourly feature store refresh |
| Event calendar | Municipal open data portal, event permit database | 4,200 events over 3 years | 5 features (type, expected attendance, zone, duration, recurrence) | Weekly sync |
| Weather forecasts | OpenWeather 5-day/3-hour forecast API | 730 days of historical forecasts | 6 features | Included in monthly retrain |

**Dependencies:**

- Upstream: Fare-gate ingestion pipeline (PL-002), event calendar sync service, OpenWeather API, zone demographic data (annual census refresh)
- Downstream: MDL-001 (demand features for routing), fleet management system, operator dashboard (zone demand heatmaps), capacity planning reports

**Performance Baselines:**

| Metric | Baseline (v1.0) | Current (v1.4) | Alert Threshold |
|--------|----------|---------|----------------|
| MAPE (overall) | 11.5% | 8.2% | > 15% (triggers auto-quarantine) |
| RMSE (passengers/slot) | 142 | 96 | > 160 |
| Peak-hour MAPE | 14.2% | 9.7% | > 18% |
| Forecast horizon accuracy (48h–72h) | MAPE 16.8% | MAPE 12.1% | > 20% |

---

### Anomaly Detector (MDL-003)

**Purpose:** Provides real-time detection of safety-relevant anomalies across the transport infrastructure. Classified as **high-risk under EU AI Act Annex III** (safety component of transport infrastructure management). The ensemble combines unsupervised anomaly detection (Isolation Forest) for novel pattern discovery with a deterministic rule engine for known failure modes, producing severity-scored alerts with contributing-factor explanations for human operators.

**Versioning:**

| Version | Training Date | Dataset Version | Key Metrics | Deployment Date | Notes |
|---------|--------------|-----------------|-------------|----------------|-------|
| v1.0 | 2023-09-01 | DS-ANOMALY-v1 | Precision: 0.88, Recall: 0.79 | 2023-10-15 | Isolation Forest only; vibration and temperature sensors |
| v2.0 | 2024-05-20 | DS-ANOMALY-v2 | Precision: 0.93, Recall: 0.86 | 2024-06-10 | Added rule engine for known failure patterns; CCTV crowd density |
| v2.5 | 2025-01-14 | DS-ANOMALY-v3 | Precision: 0.95, Recall: 0.89 | 2025-02-01 | Vehicle telemetry integration; improved crowd-surge detection |
| v3.0 | 2025-09-03 | DS-ANOMALY-v4 | Precision: 0.96, Recall: 0.91 | 2025-09-18 | SCADA alert correlation; enhanced contributing-factor explanations for EU AI Act compliance |

**Hyperparameters:**

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| n_estimators (Isolation Forest) | 200 | Convergence analysis; marginal improvement beyond 200 trees |
| contamination | 0.02 | Estimated anomaly rate from 2 years of labeled incident data |
| max_features | 0.8 | Reduces correlation between isolation trees |
| ensemble_weight_if | 0.6 | Isolation Forest weight in ensemble score; tuned on validation set |
| ensemble_weight_rules | 0.4 | Rule engine weight; higher weight for known failure modes |
| severity_threshold | 0.5 | Operating threshold for alert generation; precision-recall calibrated with safety team |

**Training Data Lineage:**

| Dataset | Source | Records | Features | Refresh Cadence |
|---------|--------|---------|----------|----------------|
| DS-ANOMALY-v4 | IoT sensor network (1,200 sensors), CCTV edge analytics, SCADA system logs, vehicle telemetry | 2.8M sensor-minutes (normal) + 4,200 labeled anomaly events | 24 features | Bi-monthly retrain; real-time sensor ingestion |
| Labeled incidents | Operator incident reports (manually labeled by safety engineers) | 4,200 incidents over 3 years | 6 label dimensions (type, severity, root cause, zone, duration, resolution) | Continuous labeling; incorporated in bi-monthly retrain |
| Rule definitions | Safety engineering team; codified from regulatory standards and historical failure analysis | 87 rules across 5 failure categories | N/A (deterministic) | Updated on regulatory change or new failure pattern discovery |

**Dependencies:**

- Upstream: IoT sensor ingestion pipeline (PL-003), CCTV crowd analytics (edge compute), SCADA integration gateway, vehicle telemetry stream
- Downstream: Operator dashboard (real-time alert feed), safety incident management system, Passenger Information Displays (safety advisories), regulatory audit log, EU AI Act compliance reporting

**Performance Baselines:**

| Metric | Baseline (v1.0) | Current (v3.0) | Alert Threshold |
|--------|----------|---------|----------------|
| Precision | 0.88 | 0.96 | < 0.90 (high-risk: strict threshold) |
| Recall | 0.79 | 0.91 | < 0.85 (high-risk: missed anomalies are safety-critical) |
| F1 Score | 0.83 | 0.93 | < 0.87 |
| Mean detection latency | 45s | 12s | > 30s |
| False-positive rate (daily) | 8.2 per day | 2.1 per day | > 5 per day |

## Integration with Model Registry Tools

| Tool | Purpose | Link |
|------|---------|------|
| MLflow | Experiment tracking, model versioning, artifact storage, model staging/production promotion | `https://mlflow.sum-platform.internal` |
| DVC | Training dataset versioning and lineage tracking (DS-* datasets) | Integrated with GitLab repository `sum-platform/ml-pipelines` |
| Feast | Feature store management; online/offline feature serving | `https://feast.sum-platform.internal` |
| Grafana + Prometheus | Production model monitoring (latency, throughput, drift metrics) | `https://grafana.sum-platform.internal/d/ml-models` |

All models are registered in MLflow with their full training lineage, hyperparameters, and evaluation metrics. Model promotion from staging to production requires passing automated quality gates (performance above alert thresholds on a held-out validation set) and, for MDL-003, explicit sign-off from the safety engineering team. DVC tracks the exact dataset version used for each training run, ensuring full reproducibility.
