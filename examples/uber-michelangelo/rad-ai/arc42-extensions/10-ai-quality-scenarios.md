# AI Quality Scenarios

> **Extends:** arc42 S10 -- Quality Requirements
>
> **System:** Uber Michelangelo Platform
>
> **Last Updated:** 2026-02-17

## Purpose

Define quality scenarios for AI-specific quality attributes that traditional quality requirements (response time, throughput, availability) do not cover: model freshness, drift tolerance, explainability, fairness, and deployment safety. These scenarios complement standard arc42 quality requirements and provide measurable acceptance criteria for Michelangelo's production models.

## Quality Attribute Definitions

| Quality Attribute | Definition | Why It Matters for Michelangelo |
|-------------------|-----------|-------------------------------|
| Model Freshness | How recent the model's training data and parameters are relative to current data distributions | Road networks change, new restaurants open, driver supply patterns shift seasonally; stale models produce systematically biased predictions |
| Drift Tolerance | Acceptable degree of data or concept drift before intervention is required | Gradual shifts in trip patterns (new subway lines, neighborhood gentrification), sudden shocks (major events, weather, pandemics) silently degrade model quality |
| Explainability | Ability to provide understandable reasons for model outputs | Fraud detection decisions must be explainable for user appeals; pricing must be justifiable to regulators; ETA errors must be diagnosable by engineers |
| Fairness | Absence of systematic bias against protected groups or geographic regions | Pricing and matching models directly affect access to transportation; geographic bias creates service equity issues |
| Deployment Safety | Confidence that model updates will not degrade production quality | With 5,000+ models and 20,000+ monthly training jobs, unsafe deployments could cascade across Uber's product ecosystem |

## Quality Scenarios

### QS-01: Model Freshness -- DeepETA Road Network Change

| Element | Description |
|---------|-------------|
| **Source** | Map data provider pushes road network update (new road, closure, speed limit change) |
| **Stimulus** | Road network change affects routes in a major metropolitan market |
| **Artifact** | MDL-ETA (DeepETA, Linear Transformer) |
| **Environment** | Normal production operation |
| **Response** | Feature Store (Palette) refreshes road segment features within 24 hours. If MAE in affected market increases >10% from baseline, trigger expedited retraining with updated map data. Deploy retrained model via canary within 48 hours of data availability. |
| **Response Measure** | Map data reflected in features within 24 hours; retrained model deployed within 72 hours of road change; MAE in affected market returns to within 5% of baseline within 1 week |

### QS-02: Model Freshness -- Demand Forecasting Seasonal Shift

| Element | Description |
|---------|-------------|
| **Source** | Calendar-aware freshness monitor |
| **Stimulus** | Demand Forecasting model (MDL-DEMAND) approaches a known distribution shift period (holiday season, major local event, school term start/end) without a recent retrain |
| **Artifact** | MDL-DEMAND (LSTM / DeepAR) |
| **Environment** | Pre-event period identified from event calendar and historical demand patterns |
| **Response** | Trigger preemptive retraining using augmented dataset that includes historical event/holiday data. Alert Demand ML team for manual review of training data composition. Deploy retrained model at least 48 hours before event onset. |
| **Response Measure** | Retraining initiated at least 7 days before event; post-retrain MAPE on event holdout set <= 15%; model deployed 48 hours before event; during-event MAPE within 20% of normal-period MAPE |

### QS-03: Drift Tolerance -- Feature Store Data Drift (D3-Detected)

| Element | Description |
|---------|-------------|
| **Source** | D3 drift detection system (Prophet-based anomaly detection, running daily) |
| **Stimulus** | D3 detects PSI > 0.2 on a critical feature (e.g., zone_supply_demand_ratio) for 2 consecutive daily checks across multiple markets |
| **Artifact** | All models consuming the drifted feature (potentially MDL-PRICE, MDL-DEMAND, MDL-MATCH) |
| **Environment** | Normal production operation, post-deployment |
| **Response** | Stage 1: D3 raises alert to ML Platform team with drift magnitude, affected features, and affected downstream models. Stage 2: ML team investigates root cause (upstream data pipeline change, external event, genuine distribution shift). Stage 3: If genuine shift, trigger retraining for affected models with recent data. If pipeline issue, escalate to data engineering for pipeline fix. |
| **Response Measure** | Drift detected within 2 days of onset (D3 target: 20x improvement over manual detection); root cause identified within 1 business day; affected models retrained or pipeline fixed within 3 business days |

### QS-04: Drift Tolerance -- Concept Drift in Fraud Detection

| Element | Description |
|---------|-------------|
| **Source** | Fraud operations team observes increasing false negative rate (fraudulent transactions passing undetected) |
| **Stimulus** | New fraud pattern emerges (e.g., account takeover via phished credentials) that does not match historical patterns in training data |
| **Artifact** | MDL-FRAUD (GBM + Neural Network ensemble) |
| **Environment** | Production; adversarial concept drift |
| **Response** | Stage 1: Fraud operations escalates observed pattern with example transactions. Stage 2: Emergency fraud model retrain incorporating labeled examples of new pattern. Stage 3: Deploy via expedited canary (reduced observation period for fraud models). Stage 4: Update rule-based fraud rules as immediate mitigation while ML model retrains. |
| **Response Measure** | Rule-based mitigation deployed within 4 hours; retrained ML model deployed within 48 hours; false negative rate for new pattern reduced to < 5% within 1 week |

### QS-05: Explainability -- Fraud Detection User Appeal

| Element | Description |
|---------|-------------|
| **Source** | User whose transaction was blocked by MDL-FRAUD submits appeal |
| **Stimulus** | Request for explanation of why a specific transaction was flagged as potentially fraudulent |
| **Artifact** | MDL-FRAUD (GBM + Neural Network ensemble) |
| **Environment** | Production; user-facing context |
| **Response** | Return structured explanation containing: (1) top-3 risk factors that contributed to the fraud score (from GBM SHAP), (2) risk category (account behavior anomaly, transaction pattern, device anomaly), (3) recommended user actions (verify identity, contact support). Human fraud analyst reviews explanation for accuracy before delivery. |
| **Response Measure** | Explanation generated within 500ms; human review completed within appeal SLA (varies by jurisdiction); explanation covers >= 70% of fraud score variance; user satisfaction with explanation >= 60% (measured via post-appeal survey) |

### QS-06: Explainability -- Pricing Investigation by Regulator

| Element | Description |
|---------|-------------|
| **Source** | Regulatory inquiry regarding pricing practices in a specific jurisdiction |
| **Stimulus** | Request for explanation of surge pricing patterns, including how the ML model determines surge multipliers and whether pricing is equitable across neighborhoods |
| **Artifact** | MDL-PRICE (GBM); MDL-DEMAND (LSTM) |
| **Environment** | Regulatory compliance context |
| **Response** | Generate comprehensive pricing analysis containing: (1) aggregate SHAP feature importance for pricing model, (2) zone-level pricing distribution analysis stratified by neighborhood demographics, (3) supply-demand ratio correlation with surge levels, (4) comparison of ML-driven pricing vs. rule-based pricing counterfactual. Delivered as structured report reviewed by legal and policy teams. |
| **Response Measure** | Report generated within 5 business days; covers all requested zones and time periods; feature attributions validated against domain expert review; report cleared by legal before delivery |

### QS-07: Fairness -- Geographic Service Equity

| Element | Description |
|---------|-------------|
| **Source** | Fairness monitoring system (weekly batch analysis) |
| **Stimulus** | Average rider wait time in a zone quintile (grouped by income level) exceeds 1.5x the wait time in the best-served quintile, sustained for 2 consecutive weeks |
| **Artifact** | MDL-MATCH (Marketplace Matching), MDL-DEMAND (Demand Forecasting) |
| **Environment** | Production; continuous monitoring |
| **Response** | Stage 1: Alert fairness monitoring team and city operations. Stage 2: Root cause analysis: distinguish between model bias (matching model under-prioritizes zone), supply shortage (genuinely fewer drivers), or infrastructure issue (coverage gap). Stage 3: If model bias, retrain with zone-balanced optimization constraints. If supply issue, adjust driver incentive allocation. |
| **Response Measure** | Wait time disparity ratio maintained below 1.5 for 90% of weekly measurements; bias-to-mitigation time < 14 days for model-related issues; quarterly fairness report to operations leadership |

### QS-08: Deployment Safety -- Model Excellence Score

| Element | Description |
|---------|-------------|
| **Source** | Gallery deployment pipeline; Model Excellence Score (MES) framework |
| **Stimulus** | ML team submits new model version for production deployment |
| **Artifact** | Any model managed by Michelangelo |
| **Environment** | Deployment pipeline (pre-production) |
| **Response** | Model Excellence Score evaluated across four dimensions: (1) offline evaluation coverage (backtesting on recent 30 days), (2) shadow deployment coverage (parallel run on production traffic), (3) unit test coverage (feature pipeline tests, inference tests), (4) performance monitoring coverage (drift detection, accuracy tracking configured). Model blocked from deployment if MES below threshold. |
| **Response Measure** | 75%+ of critical online models achieve intermediate safety level; all deployments have offline evaluation; shadow deployment covers critical models; no model deployed without monitoring configuration |

### QS-09: Deployment Safety -- Canary Failure and Rollback

| Element | Description |
|---------|-------------|
| **Source** | Canary monitoring during model deployment |
| **Stimulus** | Canary model (receiving small traffic slice) shows metric degradation: error rate increase, latency spike, or accuracy drop compared to production baseline |
| **Artifact** | Any model in canary deployment stage |
| **Environment** | Production; canary deployment phase |
| **Response** | Automated rollback: canary traffic redirected to stable production model within 2 minutes. Alert to model team with canary metrics comparison. Failed canary model retained for investigation but removed from serving path. Incident logged for post-mortem. |
| **Response Measure** | Rollback completed within 2 minutes of detection; zero user-visible impact from canary failure (traffic slice is small); investigation report within 1 business day |

### QS-10: Robustness -- Feature Store Degradation

| Element | Description |
|---------|-------------|
| **Source** | Cassandra cluster performance degradation or partial outage |
| **Stimulus** | Feature Store (Palette) online serving latency exceeds SLA (P95 > 10ms) or availability drops below 99.9% |
| **Artifact** | All models depending on online features (MDL-ETA, MDL-PRICE, MDL-FRAUD, MDL-MATCH) |
| **Environment** | Production; degraded infrastructure |
| **Response** | Each model activates its fallback strategy: MDL-ETA uses routing engine estimate, MDL-PRICE uses rule-based tiers, MDL-FRAUD uses rule-based fraud checks, MDL-MATCH uses nearest-driver dispatch. All fallback activations logged. Platform team paged for Cassandra investigation. |
| **Response Measure** | Fallback activation within 30 seconds of SLA breach; zero silent failures (all predictions either use ML or explicit fallback, never stale features without flagging); system returns to ML-powered operation within 5 minutes of Feature Store recovery |

## Quality Scenario Summary

| ID | Attribute | Model(s) | Priority | Uber-Specific Context |
|----|-----------|----------|----------|----------------------|
| QS-01 | Model Freshness | MDL-ETA | High | Road network changes directly affect ETA accuracy |
| QS-02 | Model Freshness | MDL-DEMAND | High | Seasonal demand shifts are predictable but require proactive retraining |
| QS-03 | Drift Tolerance | All (via Feature Store) | Critical | D3 system exists specifically for this scenario |
| QS-04 | Drift Tolerance | MDL-FRAUD | Critical | Adversarial concept drift is inherent to fraud detection |
| QS-05 | Explainability | MDL-FRAUD | High | User-facing fraud decisions require explanations |
| QS-06 | Explainability | MDL-PRICE, MDL-DEMAND | Medium | Regulatory scrutiny of algorithmic pricing |
| QS-07 | Fairness | MDL-MATCH, MDL-DEMAND | High | Geographic service equity is a core business commitment |
| QS-08 | Deployment Safety | All | Critical | MES framework exists specifically for this scenario |
| QS-09 | Deployment Safety | All | Critical | At 20K+ training jobs/month, safe deployment is existential |
| QS-10 | Robustness | All | Critical | Feature Store is single point of dependency for all models |
