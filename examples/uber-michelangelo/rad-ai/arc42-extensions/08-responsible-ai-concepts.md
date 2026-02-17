# Responsible AI Concepts

> **Extends:** arc42 S8 -- Cross-Cutting Concepts

## Purpose

This section documents the cross-cutting responsible AI concerns that apply across Michelangelo's production models. Uber operates in a domain where ML decisions directly affect access to transportation, pricing fairness, financial safety, and marketplace equity. These concerns, including fairness, explainability, human oversight, and transparency, are first-class architectural quality attributes that influence model design, deployment patterns, and operational procedures. As Uber operates globally and serves markets with varying regulatory requirements, responsible AI is both an ethical obligation and a compliance necessity.

## Fairness

### Fairness Concerns by Model

| Model | Protected Attributes | Fairness Concern | Impact if Unaddressed |
|-------|---------------------|------------------|----------------------|
| Dynamic Pricing (MDL-PRICE) | Geographic zone (proxy for neighborhood demographics), time-of-day (shift workers vs. daytime commuters) | Surge pricing may systematically disadvantage riders in lower-income neighborhoods or during off-peak hours when supply is scarce | Inequitable access to affordable transportation; regulatory scrutiny in multiple jurisdictions |
| Marketplace Matching (MDL-MATCH) | Driver demographics (race, gender not used as features but may correlate with geographic patterns), rider demographics | Matching optimization may create disparities in driver earnings or rider wait times across geographic areas | Systematic earnings inequality for drivers; service quality disparities for riders |
| Demand Forecasting (MDL-DEMAND) | Geographic zone, population density, historical service coverage | Underestimation of demand in historically underserved areas leads to insufficient driver pre-positioning | Self-reinforcing under-service: low predicted demand leads to fewer drivers, longer wait times, fewer trips, confirming low demand prediction |
| Fraud Detection (MDL-FRAUD) | Payment method (cash vs. digital), device type, geographic region | Fraud scoring may disproportionately flag transactions from regions with less digital payment infrastructure | Legitimate users in certain regions experience higher friction, transaction delays, or blocked payments |
| DeepETA (MDL-ETA) | Geographic zone, route type (urban vs. suburban vs. rural) | ETA accuracy may vary across geographic regions due to training data density differences | Riders in data-sparse regions receive less accurate ETAs, leading to worse user experience |

### Fairness Metrics (Inferred from Public Information)

| Model | Metric | Target | Measurement Approach |
|-------|--------|--------|---------------------|
| MDL-PRICE | Surge price disparity ratio across zone income quintiles | Ratio < 1.2 between highest and lowest quintile average surge | Zone-level surge analysis stratified by census income data |
| MDL-PRICE | Price accessibility: fraction of zones where average surge exceeds affordability threshold | < 5% of zones at any given time | Real-time zone-level monitoring |
| MDL-MATCH | Wait time equity: max wait time ratio across zones | Ratio < 1.5 between worst-served and best-served zones | Weekly aggregation of per-zone P75 wait times |
| MDL-MATCH | Driver earnings equity: earnings distribution across driver demographics | Gini coefficient < target (zone-adjusted) | Monthly earnings analysis with demographic stratification |
| MDL-DEMAND | Forecast accuracy equity: MAPE gap across zone types | MAPE gap < 5% between urban core and suburban zones | Per-retrain evaluation on zone-stratified holdout |
| MDL-FRAUD | False positive rate equity: FPR across geographic regions | FPR gap < 2% between regions | Weekly FPR analysis stratified by region |
| MDL-ETA | ETA accuracy equity: MAE gap across regions | MAE gap < 1 minute between well-served and under-served regions | Per-retrain market-level evaluation |

*Note: Specific targets are inferred based on reasonable industry practice. Uber's actual internal fairness thresholds are not publicly documented.*

### Bias Detection Architecture

- **Pre-training:** Training data representation analysis before each retrain cycle. For MDL-PRICE, the training dataset is audited for geographic coverage: each zone must contribute minimum representation to prevent under-representation of low-demand areas. For MDL-DEMAND, historical data is checked for service-coverage bias (areas with historically low supply may have artificially low demand records).

- **In-training:** MDL-DEMAND applies zone-stratified sampling to ensure equitable representation during training. MDL-MATCH uses constrained optimization that includes fairness bounds alongside efficiency objectives. MDL-FRAUD applies threshold calibration per region to equalize false positive rates.

- **Post-training:** All models undergo zone-stratified evaluation before production promotion. Gallery's evaluation gates include fairness metrics alongside accuracy and latency checks. Models that fail fairness gates are blocked from deployment regardless of accuracy improvements.

- **In-production:** Continuous fairness monitoring via dashboards. Fairness metrics are tracked as part of the Model Excellence Score (MES). Alerts trigger when any fairness metric exceeds its target for consecutive measurement periods.

## Explainability

### Explainability Requirements

| Model | Explanation Scope | Method | Audience | Trigger |
|-------|------------------|--------|----------|---------|
| DeepETA (MDL-ETA) | Per-prediction (on-demand) | Segment-level contribution breakdown (which road segments contribute most to the residual) | ML engineers (debugging), operations (investigating systematic ETA errors) | On investigation request; not for every prediction (latency constraint) |
| Dynamic Pricing (MDL-PRICE) | Aggregate | Feature importance (SHAP for GBM); zone-level pricing factor decomposition | Product managers, regulators, public affairs | Quarterly regulatory reporting; on-demand for pricing investigations |
| Fraud Detection (MDL-FRAUD) | Per-decision | Top risk factors from GBM (SHAP TreeExplainer); neural network contribution scores | Fraud operations team, affected users (on appeal), regulators | Every flagged transaction (mandatory for user-facing fraud decisions) |
| Marketplace Matching (MDL-MATCH) | Aggregate | Matching factor analysis (supply-demand balance, predicted trip value, ETA contribution) | Operations teams, driver relations | Monthly marketplace health reports |
| Demand Forecasting (MDL-DEMAND) | On-demand | Feature importance decomposition; temporal pattern attribution | City operations teams, planning | When forecasts deviate significantly from actuals |

### Explainability Integration Points

| Component | Service | Latency Budget |
|-----------|---------|---------------|
| Fraud detection inference | Per-decision risk factor extraction (co-located with inference) | < 5ms overhead (GBM SHAP is fast for tree models) |
| Fraud operations dashboard | Risk factor display for human reviewers | Real-time (streamed with decision) |
| User-facing fraud appeal | Simplified explanation of why transaction was flagged | < 500ms (on-demand generation) |
| Pricing analysis tool | Aggregate SHAP analysis for pricing models | Batch (minutes); acceptable for reporting use case |
| ML debugging tools | Per-prediction SHAP / attention visualization for any model | On-demand; no production latency constraint |

## Human Oversight

### Oversight Mechanisms

| Mechanism | Scope | Trigger | Authority | Response SLA |
|-----------|-------|---------|-----------|-------------|
| Fraud transaction review | MDL-FRAUD: transactions with ambiguous fraud scores (0.4--0.6 range) | Fraud score in ambiguous zone | Fraud operations analyst (24/7 staffing) | < 5 min for high-value transactions; < 30 min for standard |
| Model deployment approval | All models: promotion from evaluation to production via Gallery | Model passes automated quality gates | Model owner + platform team lead (high-criticality models require additional approval) | Within 1 business day (standard); within 4 hours (critical models) |
| Pricing override | MDL-PRICE: surge pricing during emergencies or sensitive events | Emergency declaration by operations or surge exceeding safety threshold | City operations manager | Immediate (real-time override capability) |
| Safety incident review | Any model producing outputs flagged as potentially harmful | Automated safety signal or user report | Safety operations team | < 15 min for safety-critical flags |
| Fairness audit | All customer-facing models | Quarterly scheduled; ad-hoc on fairness alert | Cross-functional review board (ML, product, legal, public policy) | Quarterly report within 2 weeks of audit period end |

### Override and Escalation

- **Override mechanism:** Uber's operations teams can override ML-driven decisions through administrative tools. For pricing, city operations managers can cap or disable surge pricing in specific zones during emergencies (natural disasters, safety incidents, sensitive events). For fraud detection, fraud operations analysts can manually approve flagged transactions or escalate to additional review. For marketplace matching, operations can adjust matching parameters (e.g., increase matching radius) during supply shortages.

- **Override logging:** All overrides are recorded in immutable audit logs with timestamp, operator ID, original ML output, override action, and justification. Override logs are retained for compliance and model improvement purposes. Override patterns are analyzed monthly to identify systematic model weaknesses (e.g., if operators consistently override fraud scores for a specific transaction type, the model may need retraining).

- **Escalation path:** For fraud detection: (1) automated scoring with risk factor explanation; (2) fraud analyst review for ambiguous cases; (3) fraud team lead for high-value or complex cases; (4) legal team for regulatory or dispute-related cases. For pricing: (1) automated ML-driven surge; (2) city operations override for emergencies; (3) regional operations head for policy-level pricing decisions; (4) executive override for company-wide pricing changes.

## Transparency

### Disclosure Requirements

| Context | Disclosure | Format | Regulatory Driver |
|---------|-----------|--------|-------------------|
| Rider-facing fare estimate | Surge pricing indicator showing dynamic pricing is in effect | In-app UI: "Prices are higher due to increased demand" with multiplier | Various US state regulations; EU consumer protection |
| Driver-facing trip offer | Trip information including estimated earnings, distance, duration | In-app driver interface with trip details before acceptance | Driver transparency regulations (California AB5, EU gig worker directive) |
| Fraud-flagged transaction | Notification when payment is flagged or blocked, with appeal option | In-app notification with reason code and appeal link | PSD2 (EU payment regulation); consumer protection laws |
| Regulatory reporting | ML model inventory, fairness metrics, override statistics | Structured reports per jurisdiction | Varies by jurisdiction; EU AI Act (when applicable) |
| Data subject access requests | Description of ML processing applied to user data | Structured response per applicable privacy regulation | GDPR Art. 13, 14, 15; CCPA |

## Privacy

### Data Protection

| Data Category | Classification | Handling | ML Pipeline Use |
|---------------|---------------|----------|----------------|
| Trip records (origin, destination, route) | Personal data | Encrypted at rest and in transit; anonymized for aggregate features; user ID stripped before ML pipeline entry | Zone-level aggregates for demand forecasting and pricing; not individual trips |
| Driver location (GPS traces) | Personal data | Real-time location used for matching; historical traces aggregated to zone-level supply metrics | Zone-level supply features; individual driver features computed with privacy-preserving aggregation |
| Payment data (card details, transactions) | Sensitive personal data | PCI-DSS compliant handling; tokenized; fraud model receives derived features, not raw card data | Transaction features (amount range, frequency, payment method type) for fraud detection |
| User behavioral signals (app usage patterns) | Personal data | Aggregated to feature-level (e.g., "trips in last 7 days"); individual session data not retained in ML pipelines | Behavioral features for fraud detection and personalization |
| Communication data (messages between driver/rider) | Sensitive personal data | Not used in ML pipelines; retained for safety and dispute resolution only | Not used |
| Demographic inferences | Sensitive inferred data | Not used as model features; monitored for fairness evaluation only (does not enter training or serving) | Fairness monitoring only (post-hoc analysis, not model input) |

### Privacy-by-Design in Feature Engineering

Palette's feature engineering pipelines implement privacy-by-design principles:

1. **Aggregation at ingestion:** Individual-level data is aggregated to zone-level or cohort-level before entering the Feature Store for most use cases. Exception: fraud detection requires individual-level features (transaction history) with strict access controls.

2. **Differential privacy for sensitive aggregates:** Where zone-level aggregates could reveal individual behavior (very low-density zones), noise injection is applied before feature materialization.

3. **Feature access controls:** Palette implements role-based access to feature groups. Safety-critical features (fraud-related) are restricted to authorized model pipelines only. Feature access is audited.

4. **Data retention:** Feature values in the online store (Cassandra) have TTLs aligned with feature freshness requirements (not retention requirements). Offline feature history is retained for model reproducibility with configurable retention periods per regulatory jurisdiction.
