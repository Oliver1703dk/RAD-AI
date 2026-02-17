# AI Quality Scenarios

> **Extends:** arc42 S10 -- Quality Requirements
>
> **System:** Smart Urban Mobility (SUM) Ecosystem
>
> **Last Updated:** 2026-02-17

## Purpose

Define quality scenarios for AI-specific quality attributes that traditional scenarios (response time, throughput, availability) do not cover: model freshness, drift tolerance, explainability, fairness, and robustness. These scenarios complement standard arc42 quality requirements and provide measurable acceptance criteria for the SUM platform's three AI components.

## Quality Attribute Definitions

| Quality Attribute | Definition | Why It Matters for SUM |
|-------------------|-----------|------------------------|
| Model Freshness | How recent the model's training data and parameters are relative to current data distributions | Urban traffic patterns shift with infrastructure changes, events, and seasonal ridership; stale models route commuters suboptimally or miss emerging demand patterns |
| Drift Tolerance | Acceptable degree of data or concept drift before intervention is required | Gradual shifts in commuter behavior (e.g., new metro line opening, post-pandemic mode preferences) silently degrade predictions if undetected |
| Explainability | Ability to provide understandable reasons for model outputs | MDL-003 is high-risk under EU AI Act Annex III; city authorities require justifiable anomaly alerts before triggering emergency protocols |
| Fairness | Absence of systematic bias against protected groups or geographic zones | Municipal mandate requires equitable transit service across all city zones, including historically underserved neighborhoods |
| Robustness | Resilience to adversarial inputs, edge cases, and distribution shifts | Sensor failures, GPS spoofing, and extreme weather events produce out-of-distribution inputs that must not cause silent failures |

## Quality Scenarios

### QS-01: Model Freshness -- Route Optimizer

| Element | Description |
|---------|-------------|
| **Source** | Scheduled freshness monitor (cron job, daily at 02:00 UTC) |
| **Stimulus** | Route Optimizer (MDL-001) has not been retrained for more than 30 days |
| **Artifact** | MDL-001 v2.1 (Gradient Boosted Machine) |
| **Environment** | Normal production operation |
| **Response** | Trigger scheduled retraining pipeline; notify ML engineering team via Slack #sum-ml-ops channel; log freshness violation in model registry |
| **Response Measure** | Retraining completes within 6 hours; new model candidate passes validation gate (accuracy >= 0.92) before promotion; maximum staleness SLA: 35 days |

### QS-02: Model Freshness -- Demand Predictor

| Element | Description |
|---------|-------------|
| **Source** | Freshness monitor with calendar-aware scheduling |
| **Stimulus** | Demand Predictor (MDL-002) approaches a known distribution shift period (public holidays, school breaks, major city events) without a recent retrain |
| **Artifact** | MDL-002 v1.4 (LSTM) |
| **Environment** | Pre-holiday or pre-event period (identified from city event calendar integration) |
| **Response** | Trigger preemptive retraining using augmented training set that includes historical holiday/event data; alert ML team for manual review of training data composition |
| **Response Measure** | Retraining initiated at least 7 days before event; post-retrain MAPE on holiday holdout set <= 12%; model deployed at least 48 hours before event onset |

### QS-03: Drift Tolerance -- Demand Predictor Data Drift

| Element | Description |
|---------|-------------|
| **Source** | Drift monitoring agent (runs hourly) |
| **Stimulus** | Population Stability Index (PSI) exceeds 0.2 on any of the top-10 input features for MDL-002 (e.g., passenger_count_zone, weather_severity, time_of_day_encoding) |
| **Artifact** | MDL-002 v1.4 (LSTM) |
| **Environment** | Normal production operation, post-deployment |
| **Response** | Stage 1: Flag feature(s) for investigation, log drift scores to monitoring dashboard. Stage 2: If PSI > 0.2 persists for 3 consecutive hourly checks, escalate to ML team and evaluate switching to conservative fallback (historical average baseline) |
| **Response Measure** | Drift detected within 1 hour of onset; MAPE degradation < 3 percentage points before intervention; fallback model activated within 30 minutes of escalation |

### QS-04: Drift Tolerance -- Route Optimizer Concept Drift

| Element | Description |
|---------|-------------|
| **Source** | Accuracy monitoring agent (evaluates against ground truth daily) |
| **Stimulus** | Route Optimizer (MDL-001) rolling 7-day accuracy drops below 0.90 (baseline: 0.93), indicating concept drift (e.g., new road closure, transit schedule change) |
| **Artifact** | MDL-001 v2.1 (GBM) |
| **Environment** | Production; infrastructure or schedule change detected in upstream data |
| **Response** | Trigger drift-initiated retraining with recent 14-day window weighted 3x; deploy via canary (5% traffic) before full rollout; notify transit operators of temporary accuracy reduction |
| **Response Measure** | Concept drift detected within 24 hours; retrained model recovers accuracy >= 0.92 within 48 hours of detection; zero user-facing service interruption during retraining |

### QS-05: Explainability -- Anomaly Detector (High-Risk)

| Element | Description |
|---------|-------------|
| **Source** | City transport authority safety officer; EU AI Act compliance auditor |
| **Stimulus** | Request for explanation of a specific anomaly alert (e.g., crowd surge detected at Central Station, alert ID ANM-2026-0342) |
| **Artifact** | MDL-003 v3.0 (Isolation Forest + rules ensemble) |
| **Environment** | Production; during post-incident review or scheduled quarterly audit |
| **Response** | Return structured explanation containing: (1) top-5 contributing features with isolation path depths and rule activations, (2) comparison to the 3 nearest normal observations, (3) confidence score with calibrated uncertainty interval, (4) human-readable narrative summary suitable for non-technical stakeholders |
| **Response Measure** | Explanation generated within 500ms of request; explanation fidelity score >= 0.92 (measured by local faithfulness metric); explanation covers >= 80% of prediction variance; audit trail retained for 5 years per EU AI Act Annex IV |

### QS-06: Explainability -- Route Optimizer for City Authority

| Element | Description |
|---------|-------------|
| **Source** | City transport authority (aggregate reporting) |
| **Stimulus** | Monthly request for system-level explanation of route recommendations across zones (e.g., "Why were 40% of commuters routed through Zone B instead of Zone C this month?") |
| **Artifact** | MDL-001 v2.1 (GBM) |
| **Environment** | Production; monthly reporting cycle |
| **Response** | Generate aggregate SHAP summary plots showing global feature importance; produce zone-level breakdown of dominant routing factors (traffic density, weather impact, transit capacity); deliver as automated PDF report via reporting pipeline |
| **Response Measure** | Report generated within 2 hours of request; covers all 12 city zones; feature attributions validated against domain expert review quarterly |

### QS-07: Fairness -- Equitable Service Across City Zones

| Element | Description |
|---------|-------------|
| **Source** | Fairness monitoring system (weekly batch analysis) |
| **Stimulus** | Service quality disparity ratio between any two city zones exceeds 1.15 (i.e., average route quality in one zone is more than 15% worse than another zone), measured by average travel time deviation from optimal |
| **Artifact** | MDL-001 v2.1 (Route Optimizer) and MDL-002 v1.4 (Demand Predictor) jointly |
| **Environment** | Production; continuous monitoring with weekly aggregation |
| **Response** | Stage 1: Alert fairness monitoring team and city equity officer. Stage 2: Analyze root cause -- distinguish between model bias (requires retraining with zone-balanced sampling) vs. infrastructure disparity (requires city authority escalation). Stage 3: If model bias confirmed, activate zone-balanced retraining within 72 hours |
| **Response Measure** | Zone service disparity ratio maintained below 1.15 for 95% of weekly measurements; bias-to-mitigation time < 14 days; quarterly fairness audit report submitted to city authority |

### QS-08: Fairness -- Demand Predictor Demographic Equity

| Element | Description |
|---------|-------------|
| **Source** | Quarterly fairness audit (automated pipeline + manual review) |
| **Stimulus** | Demand prediction MAPE varies by more than 3 percentage points across zones with different socioeconomic profiles (e.g., Zone A MAPE: 7.1%, Zone D MAPE: 11.8%) |
| **Artifact** | MDL-002 v1.4 (LSTM) |
| **Environment** | Production; quarterly audit cycle |
| **Response** | Investigate data representation across zones; augment training data for underrepresented zones using synthetic oversampling or transfer learning from well-represented zones; retrain and validate per-zone MAPE before deployment |
| **Response Measure** | Per-zone MAPE variance < 3 percentage points; all zones achieve MAPE <= 10%; corrective retrain completed within 21 days of audit finding |

### QS-09: Robustness -- Sensor Failure Handling

| Element | Description |
|---------|-------------|
| **Source** | Corrupted or missing real-time sensor data (traffic cameras, IoT occupancy sensors, weather stations) |
| **Stimulus** | More than 20% of expected input features are missing or flagged as anomalous by data validation layer (e.g., GPS drift, sensor timeout, implausible values) |
| **Artifact** | All models (MDL-001, MDL-002, MDL-003) |
| **Environment** | Production; degraded data quality conditions |
| **Response** | Data validation layer rejects malformed inputs; models switch to degraded-mode operation using imputed values from last-known-good state for MDL-001/MDL-002; MDL-003 increases sensitivity threshold (lower anomaly score cutoff) to maintain safety margin; all degraded-mode predictions flagged with low-confidence indicator |
| **Response Measure** | Zero silent failures on corrupted inputs; degraded-mode activation within 500ms; all degraded-mode predictions logged for post-hoc review; system returns to normal mode within 5 minutes of data quality restoration |

### QS-10: Robustness -- Adversarial Input Resistance for Anomaly Detector

| Element | Description |
|---------|-------------|
| **Source** | Adversarial or out-of-distribution input (e.g., spoofed sensor readings, coordinated false crowd reports) |
| **Stimulus** | Input vector falls outside the convex hull of training data distribution (detected by Mahalanobis distance > 3.5 from centroid) or input rate exceeds 10x normal from a single source |
| **Artifact** | MDL-003 v3.0 (Anomaly Detector) |
| **Environment** | Production; potential adversarial conditions |
| **Response** | Flag input as potentially adversarial; do not suppress anomaly detection (fail-safe: treat ambiguous signals as potential anomalies rather than dismissing them); route flagged inputs to human operator for verification; log full input vector and source metadata for forensic analysis |
| **Response Measure** | 100% of OOD inputs detected and logged; false-negative rate on adversarial inputs < 1%; human operator notified within 30 seconds; forensic log retention: 2 years |

## Quality Scenario Summary

| ID | Attribute | Model(s) | Priority | EU AI Act Relevant |
|----|-----------|----------|----------|--------------------|
| QS-01 | Model Freshness | MDL-001 | High | No |
| QS-02 | Model Freshness | MDL-002 | High | No |
| QS-03 | Drift Tolerance | MDL-002 | High | No |
| QS-04 | Drift Tolerance | MDL-001 | Medium | No |
| QS-05 | Explainability | MDL-003 | Critical | Yes -- Annex IV Art. 11, 13 |
| QS-06 | Explainability | MDL-001 | Medium | No |
| QS-07 | Fairness | MDL-001, MDL-002 | High | Yes -- Annex IV Art. 10 |
| QS-08 | Fairness | MDL-002 | Medium | No |
| QS-09 | Robustness | All | Critical | Yes -- Annex IV Art. 15 |
| QS-10 | Robustness | MDL-003 | Critical | Yes -- Annex IV Art. 15 |
