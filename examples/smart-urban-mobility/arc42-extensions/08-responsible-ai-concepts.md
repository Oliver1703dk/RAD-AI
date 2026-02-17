# Responsible AI Concepts

> **Extends:** arc42 §8 — Cross-Cutting Concepts

## Purpose

This section documents the cross-cutting responsible AI concerns that apply across all three AI components of the SUM platform. These concerns — fairness, explainability, human oversight, transparency, and privacy — are first-class architectural quality attributes that influence component design, deployment patterns, and operational procedures. For the Anomaly Detector (MDL-003), which is classified as high-risk under EU AI Act Annex III, these requirements carry regulatory force and are subject to conformity assessment.

## Fairness

### Fairness Metrics

| Model | Protected Attributes | Metric | Target | Current | Measurement Frequency |
|-------|---------------------|--------|--------|---------|----------------------|
| Route Optimizer (MDL-001) | Geographic zone (proxy for socioeconomic status), mobility impairment flag | Demographic Parity of route quality score across zones | Quality score gap < 0.08 between highest and lowest-income zone quintiles | 0.06 | Per retrain (quarterly) + monthly production monitoring |
| Route Optimizer (MDL-001) | Transport mode preference (car-free households) | Equal opportunity: car-free households receive comparable ETA accuracy | ETA error gap < 1.5 min vs. car-owning baseline | 1.1 min | Per retrain (quarterly) |
| Demand Predictor (MDL-002) | Geographic zone (suburban vs. urban), transport mode | Equalized error rates across zones | MAPE gap < 3% between urban and suburban zones | 2.4% | Per retrain (monthly) + weekly production monitoring |
| Anomaly Detector (MDL-003) | Infrastructure age (older infrastructure in lower-income areas), zone socioeconomic index | Equalized false-negative rates across zones | False-negative rate gap < 2% between zone quintiles | 1.3% | Per retrain (bi-monthly) + continuous production monitoring |

### Bias Detection Architecture

- **Pre-training:** Dataset representation analysis conducted before each retrain cycle. For MDL-001, route training data is audited for geographic coverage — each of the 48 zones must contribute at least 1% of training trips (minimum representation threshold). For MDL-003, labeled incident data is stratified by infrastructure age to ensure older infrastructure is not underrepresented in anomaly examples. Representation reports are versioned alongside datasets in DVC.

- **In-training:** MDL-001 applies sample weighting to upweight underrepresented zone-mode combinations during gradient boosting. MDL-002 uses zone-stratified cross-validation folds to ensure error metrics are evaluated per zone rather than only in aggregate. MDL-003 is not bias-corrected in-training due to the safety-critical nature — the priority is maximum recall across all zones rather than equalized metrics, with post-training calibration applied instead.

- **Post-training:** All three models undergo slice-based evaluation on protected-attribute subgroups before promotion from staging to production. For MDL-003, the safety engineering team reviews per-zone performance breakdowns and must sign off that no zone falls below the minimum recall threshold of 0.85. Threshold calibration is applied per zone for MDL-003 to equalize false-negative rates across infrastructure-age quintiles.

- **In-production:** Continuous fairness monitoring via Grafana dashboards. Weekly automated reports compare key metrics across protected-attribute slices. Alerts trigger if any fairness metric exceeds its target threshold for two consecutive measurement periods. Monthly fairness review meeting with the data protection officer and a representative from the city transport authority's equity office.

## Explainability

### Explainability Requirements

| Model | Explanation Scope | Method | Audience | Trigger |
|-------|------------------|--------|----------|---------|
| Route Optimizer (MDL-001) | Per-decision | SHAP values (TreeSHAP for GBM) | Commuters (simplified), operators (full) | Every route recommendation; commuter-facing explanation on request via app |
| Route Optimizer (MDL-001) | Aggregate | Global feature importance (SHAP summary) | City transport authority, ML engineering team | Quarterly model review report |
| Demand Predictor (MDL-002) | Aggregate | Feature importance ranking, temporal attention weights visualization | Transit operators, city transport authority | Monthly forecast accuracy report; on-demand for capacity planning discussions |
| Demand Predictor (MDL-002) | Per-decision (on-demand) | SHAP values for individual zone-mode-window predictions | ML engineering team (debugging), transit operators (anomalous forecasts) | When operator flags an unexpected forecast for investigation |
| Anomaly Detector (MDL-003) | Per-decision (mandatory) | Contributing-factor decomposition (rule engine: exact rule match; Isolation Forest: feature contribution scores), severity score breakdown | Human operators, safety engineers, EU AI Act auditors | Every alert (regulatory requirement for high-risk system) |
| Anomaly Detector (MDL-003) | Aggregate | Quarterly anomaly type distribution, false-positive analysis, per-zone detection rates | City transport authority, regulatory body | Quarterly EU AI Act compliance report |

### Explainability Integration Points

| Component | Explainability Service | API Surface | Latency Budget |
|-----------|----------------------|-------------|---------------|
| Route Optimizer inference service | TreeSHAP computation (co-located with model serving) | Internal gRPC `ExplainRoute(route_id)` → top-5 SHAP feature contributions | < 50ms (computed alongside inference; negligible overhead for tree-based SHAP) |
| Mobile app / web frontend | Explanation rendering service | REST `GET /api/v1/routes/{id}/explanation` → human-readable explanation text | < 200ms (includes SHAP computation + natural language template rendering) |
| Demand Predictor inference service | SHAP explanation endpoint (on-demand, not computed by default) | REST `POST /api/v1/demand/explain` with zone, mode, time_window → feature contributions | < 2s (acceptable for on-demand debugging; not on critical path) |
| Anomaly Detector inference service | Contributing-factor engine (computed for every alert) | Internal gRPC `GetAlertExplanation(alert_id)` → contributing factors with scores, rule matches, confidence breakdown | < 100ms (mandatory for every alert; co-located with inference) |
| Operator dashboard | Explanation display component | WebSocket push with alert + explanation payload | Real-time (streamed with alert; no additional latency) |
| Audit log service | Explanation archival | Async write; explanation payload stored alongside prediction for regulatory traceability | Non-blocking; eventual consistency within 5s |

## Human Oversight

### Oversight Mechanisms

| Mechanism | Scope | Trigger | Authority | Response SLA |
|-----------|-------|---------|-----------|-------------|
| Anomaly alert review (human-in-the-loop) | All MDL-003 alerts with severity > 0.5 | Every alert above threshold | Trained transport operator (shift-based, 24/7 coverage) | < 5 min for severity > 0.8; < 15 min for severity 0.5–0.8 |
| Ambiguous anomaly escalation | MDL-003 alerts with severity 0.4–0.6 (ambiguous zone) | Anomaly score in ambiguous range | Senior operator or safety engineer | < 15 min for review; escalation to safety engineer if unresolved in 30 min |
| Route quality spot check | Random sample of MDL-001 route recommendations (2% daily) | Automated random selection | Transport planning analyst | Next business day review; findings reported in weekly quality meeting |
| Demand forecast review | MDL-002 forecasts where actual demand deviates >25% from prediction | Automated deviation detection | Transit operator demand planner | Within 4 hours during operational hours; next morning for overnight deviations |
| Model retraining approval | All model version promotions to production | Completion of automated quality gates | ML engineering team lead (MDL-001, MDL-002); safety engineering team lead + ML lead (MDL-003) | Within 2 business days (MDL-001/002); within 5 business days (MDL-003, includes safety review) |
| Quarterly model audit | All three models — performance, fairness, drift | Calendar-scheduled | City transport authority AI governance committee | Audit report delivered within 10 business days of quarter end |

### Override and Escalation

- **Override mechanism:** Human operators can override any AI-generated output through the operator dashboard. For MDL-001, operators can manually adjust route recommendations or suppress specific routes. For MDL-002, operators can apply manual demand adjustments for specific zones (e.g., known events not yet in the calendar). For MDL-003, operators can dismiss false-positive alerts or manually raise alert severity. All overrides require a free-text justification field (minimum 20 characters).

- **Override logging:** Every override is recorded in an immutable audit log with the following fields: timestamp, operator ID, model ID, original AI output, override action, justification text, and downstream effects (e.g., which fleet allocations changed). Override logs are retained for 5 years (EU AI Act Annex IV record-keeping requirement for high-risk systems). Override frequency and patterns are reviewed in the monthly fairness meeting and quarterly model audit.

- **Escalation path:** For MDL-003 (high-risk), the escalation path is: (1) automated alert with contributing-factor explanation displayed to on-duty operator; (2) if operator cannot resolve within SLA, automatic escalation to shift supervisor; (3) if severity > 0.9 or infrastructure failure suspected, parallel notification to safety engineering on-call and city transport authority emergency coordinator; (4) if model is producing sustained anomalous outputs (>10 unresolved alerts in 30 minutes), automatic circuit-breaker activates rule-based-only mode and pages ML engineering on-call. For MDL-001 and MDL-002, escalation is to the ML engineering team via standard incident management (PagerDuty).

## Transparency

### Disclosure Requirements

| Context | Disclosure | Format | Regulation |
|---------|-----------|--------|------------|
| Mobile app route recommendation | "Routes are ranked by an AI system using real-time traffic, weather, and demand data. Tap any route to see why it was recommended." | In-app UI label above route list; expandable explanation card per route | EU AI Act Art. 50 (transparency obligations for AI systems interacting with natural persons) |
| Operator dashboard anomaly alerts | "Alert generated by AI anomaly detection system (MDL-003 v3.0). Contributing factors and confidence score shown below. All alerts require human review." | Dashboard header banner; per-alert metadata panel | EU AI Act Art. 14 (human oversight for high-risk systems); Annex IV (technical documentation) |
| Demand forecast reports | "Forecasts generated by AI demand prediction model (MDL-002 v1.4). Accuracy: MAPE 8.2% overall. See methodology appendix for model details." | Report footer; methodology appendix in quarterly capacity planning report | EU AI Act Art. 13 (transparency for high-risk systems); transit operator data-sharing agreements |
| Public-facing service disruption notices | When anomaly detector triggers a public-facing advisory: "This service advisory was initiated based on automated infrastructure monitoring. Human operators have reviewed and confirmed this advisory." | Public announcement text template; passenger information display format | EU AI Act Art. 50; city transport authority public communication policy |
| Data subject access requests | Description of AI processing of personal data (anonymized trip data used for model training; real-time location used for route optimization) | Structured response template per GDPR Art. 15; machine-readable format available | GDPR Art. 13, 14, 15; EU AI Act Art. 13 |
| Regulatory conformity documentation | Complete technical documentation for MDL-003 including training data description, model architecture, performance metrics, fairness assessment, and risk management measures | PDF report following EU AI Act Annex IV structure; updated per model version | EU AI Act Annex IV (mandatory for high-risk systems) |

## Privacy

### Data Protection

| Data Category | Classification | Handling | Retention | Legal Basis |
|---------------|---------------|----------|-----------|-------------|
| Trip records (fare-gate tap-in/tap-out) | Personal data (pseudonymized card ID) | Pseudonymized at ingestion: card ID replaced with rotating hash (salt rotated monthly). Used for demand aggregation only. Individual trip sequences not used for model training — only zone-mode-timeslot aggregates. | Raw: 90 days (operational need). Aggregated: 3 years (model training). | GDPR Art. 6(1)(f) — legitimate interest (public transport optimization); Data Protection Impact Assessment completed. |
| Mobile app route requests | Personal data (user account, origin/destination) | End-to-end TLS in transit. Origin/destination generalized to zone level before logging. User ID stripped before any data enters ML pipelines. Real-time inference uses ephemeral session context only (not persisted). | Session context: duration of request only. Anonymized route choice logs: 2 years (feedback loop). | GDPR Art. 6(1)(b) — performance of contract (provision of routing service). |
| IoT sensor data | Non-personal (infrastructure telemetry) | No personal data content. Encrypted at rest (AES-256) in TimescaleDB. Access restricted to ML engineering and safety engineering teams. | Raw: 1 year. Aggregated features: 5 years (regulatory requirement for high-risk system evidence). | N/A (non-personal). Retained under EU AI Act Annex IV record-keeping obligations for MDL-003. |
| CCTV crowd density estimates | Anonymized (derived metric only) | Edge compute processes raw CCTV frames on-premise; only aggregate density metric (persons/m²) is transmitted. No images, no facial features, no individual tracking data leaves edge nodes. Raw frames not retained. | Density metrics: 1 year. Raw CCTV frames: not retained (processed and discarded at edge within 5s). | GDPR Art. 6(1)(e) — public interest (transport safety); DPIA completed; proportionality assessment documented. |
| Operator override logs | Personal data (operator ID, justification text) | Encrypted at rest. Access restricted to audit function and AI governance committee. Operator IDs are internal employee identifiers. | 5 years (EU AI Act Annex IV record-keeping for high-risk system human oversight evidence). | GDPR Art. 6(1)(c) — legal obligation (EU AI Act record-keeping); employment contract provisions. |
| Model training datasets (DS-*) | Anonymized / aggregated | All training data is aggregated to zone-mode-timeslot level. No individual-level records in any training dataset. Dataset lineage tracked in DVC; data deletion requests can be propagated through re-aggregation pipeline. | Dataset versions retained for model reproducibility: 5 years for MDL-003 (regulatory); 3 years for MDL-001/002 (operational). | GDPR Art. 6(1)(f) — legitimate interest; Recital 50 (further processing for statistical purposes compatible with original purpose). |

### Data Protection Impact Assessment (DPIA)

A DPIA has been conducted for the SUM platform covering all three AI components, with particular attention to MDL-003 due to its high-risk classification. Key findings:

- **Necessity and proportionality:** The use of AI is proportionate to the legitimate aims of transport optimization and safety. Data minimization principles are applied at each pipeline stage (zone-level aggregation, pseudonymization at ingestion, edge processing for CCTV).
- **Risks to data subjects:** Primary risk is re-identification through trip pattern analysis. Mitigated by monthly salt rotation for pseudonymization, aggregation to zone level before ML pipeline entry, and prohibition on individual-level sequence analysis.
- **Safeguards:** Technical measures (encryption, pseudonymization, access controls, edge processing) and organizational measures (DPO review of all model retraining data, annual privacy audit, data subject rights procedures) are documented and operational.
- **DPO sign-off:** The Data Protection Officer has reviewed and approved the DPIA. Next review scheduled for Q2 2026 or upon significant system change, whichever is earlier.
