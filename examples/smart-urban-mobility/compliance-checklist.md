# EU AI Act Annex IV — Compliance Checklist

> Maps EU AI Act Annex IV technical documentation requirements to RAD-AI framework sections.
>
> **Usage:** Copy this checklist into your project. For each requirement, check the box when the corresponding RAD-AI section is filled in and covers the requirement.

## Instructions

1. Fill in the system name and risk classification
2. Work through each requirement, filling in the corresponding RAD-AI section
3. Mark each item as complete when documented
4. Review with your compliance team

---

**System Name:** Smart Urban Mobility Platform (SUM)
**Risk Classification:** [x] High-risk  [ ] Limited-risk  [ ] Minimal-risk
**Assessment Date:** 2026-01-15
**Assessor:** ML Architecture Team

---

## 1. General Description of the AI System

- [x] **1.1** Intended purpose of the AI system
  - **RAD-AI Section:** arc42 §1 (Introduction & Goals)
  - **Evidence:** See `arc42-extensions/03-ai-boundary-delineation.md`, AI Components Inventory table — documents all three AI components (MDL-001 Route Optimizer, MDL-002 Demand Predictor, MDL-003 Anomaly Detector) with their task descriptions, input/output domains, and intended operational purpose within the SUM platform.

- [x] **1.2** Name and version of the AI system
  - **RAD-AI Section:** arc42 §1 (Introduction & Goals)
  - **Evidence:** See `arc42-extensions/05-model-registry-view.md`, Model Inventory table — lists each AI component by Model ID, name, framework, current version (MDL-001 v2.3.1, MDL-002 v1.4.0, MDL-003 v3.1.0), and production status.

- [x] **1.3** Description of interaction with hardware or software not part of the AI system
  - **RAD-AI Section:** arc42 §3 (Context & Scope) + AI Boundary Delineation
  - **Evidence:** See `arc42-extensions/03-ai-boundary-delineation.md`, External AI Dependencies table and System Boundary Diagram — documents integration with the city transport authority GTFS-RT feed, municipal IoT sensor network, third-party weather API, and ride-share operator APIs. See also `c4-diagrams/ai-component-stereotypes.md`, Container Diagram showing API Gateway, external data sources, and transit operator interfaces.

- [x] **1.4** Versions of relevant software or firmware
  - **RAD-AI Section:** Model Registry View (version history)
  - **Evidence:** See `arc42-extensions/05-model-registry-view.md`, Versioning tables per model — documents version history with training dates, dataset versions, key metrics, and deployment dates for MDL-001 (XGBoost 1.7, scikit-learn 1.3), MDL-002 (PyTorch 2.1), and MDL-003 (scikit-learn 1.3, custom rules engine v2.0). See also Integration with Model Registry Tools table listing MLflow v2.8 and DVC v3.2.

## 2. Detailed Description of System Elements

- [x] **2.1** Development process and methods
  - **RAD-AI Section:** arc42 §5 (Building Block View) + Model Registry View
  - **Evidence:** See `arc42-extensions/05-model-registry-view.md`, Model Details sections — documents the development methodology per model including hyperparameter optimization (grid search for MDL-001, Bayesian optimization for MDL-002), training infrastructure (GPU cluster for MDL-002, CPU for MDL-001/MDL-003), and validation strategy (k-fold cross-validation, temporal holdout). See `arc42-extensions/09-ai-adr-template.md` for AI-ADR-001 through AI-ADR-003 recording model selection rationale.

- [x] **2.2** Design specifications and system architecture
  - **RAD-AI Section:** arc42 §5–§7 + C4 diagrams + AI Component Stereotypes
  - **Evidence:** See `c4-diagrams/ai-component-stereotypes.md`, Container Diagram with AI stereotypes — shows all system components annotated with `<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, and `<<HITL>>` stereotypes. See `arc42-extensions/03-ai-boundary-delineation.md`, System Boundary Diagram showing deterministic and non-deterministic regions. See `c4-diagrams/non-determinism-boundary.md`, Boundary Overview diagram and Boundary Interfaces table.

- [x] **2.3** Description of computational resources used
  - **RAD-AI Section:** arc42 §7 (Deployment View)
  - **Evidence:** See `arc42-extensions/12-operational-ai-view.md`, Retraining Process table — documents compute resources for training (4x NVIDIA A100 GPU nodes for MDL-002, 32-core CPU instances for MDL-001/MDL-003) and inference (Kubernetes cluster, 3 replicas per model, auto-scaling policy). See `arc42-extensions/05-model-registry-view.md`, Hyperparameters tables specifying batch sizes and memory requirements per model.

## 3. Data and Data Governance

- [x] **3.1** Description of training data — characteristics, requirements, and data governance measures
  - **RAD-AI Section:** Data Pipeline View + Data Lineage Overlay
  - **Evidence:** See `arc42-extensions/06-data-pipeline-view.md`, Pipeline Inventory and Pipeline Details sections — documents all data pipelines (PL-001 Traffic Ingestion, PL-002 Demand Data, PL-003 Sensor Anomaly Stream) with schedules, SLAs, and data quality gates. See `c4-diagrams/data-lineage-overlay.md`, Data Source Inventory table listing all 7 data sources with freshness requirements and privacy classifications.

- [x] **3.2** Description of data collection and provenance
  - **RAD-AI Section:** Data Lineage Overlay (provenance tracking)
  - **Evidence:** See `c4-diagrams/data-lineage-overlay.md`, Lineage Diagram and Lineage Details tables — traces data provenance from GTFS-RT feeds (DS-001), IoT sensor streams (DS-002), historical ridership databases (DS-003), weather API (DS-004), and crowd-sourced incident reports (DS-005) through transformation layers to model consumption. Schema Registry section documents schema versions at each boundary.

- [x] **3.3** Information about data preparation (annotation, labeling, cleaning)
  - **RAD-AI Section:** Data Pipeline View (preprocessing stages)
  - **Evidence:** See `arc42-extensions/06-data-pipeline-view.md`, Pipeline Details sections per pipeline — documents preprocessing stages including outlier removal (z-score > 3), missing value imputation (temporal interpolation for sensor data), GPS coordinate validation, schema enforcement, and feature engineering (142 features for MDL-001, 87 time-series features for MDL-002). Data Quality Checkpoints tables specify validation rules at each stage.

- [x] **3.4** Formulation of assumptions about training data
  - **RAD-AI Section:** AI-ADR (dataset characteristics section)
  - **Evidence:** See `arc42-extensions/09-ai-adr-template.md`, AI-ADR-001 through AI-ADR-003, Dataset Characteristics tables — documents training dataset sizes (MDL-001: 2.4M trip records, MDL-002: 18 months hourly demand, MDL-003: 156K labeled events including 12K anomalies), feature counts, class balance, data freshness assumptions, and known biases (e.g., underrepresentation of low-ridership suburban zones in MDL-002, seasonal event sparsity for MDL-003).

## 4. Training Methodology and Techniques

- [x] **4.1** Description of training methodologies and techniques used
  - **RAD-AI Section:** AI-ADR (model selection and training decisions)
  - **Evidence:** See `arc42-extensions/09-ai-adr-template.md`, AI-ADR-001 (GBM selection for route optimization with gradient boosting over random forest and neural alternatives), AI-ADR-002 (LSTM architecture for demand forecasting with sequence-to-sequence temporal modeling), AI-ADR-003 (Isolation Forest + rule-based hybrid for anomaly detection with ensemble decision rationale). Each ADR documents training methodology, cross-validation strategy, and regularization approaches.

- [x] **4.2** Description of training choices, optimization techniques
  - **RAD-AI Section:** AI-ADR (considered alternatives, hyperparameters)
  - **Evidence:** See `arc42-extensions/09-ai-adr-template.md`, Considered Alternatives tables per ADR — documents all evaluated model architectures with pros/cons tradeoffs. See `arc42-extensions/05-model-registry-view.md`, Hyperparameters tables per model documenting learning rates, tree depths, regularization parameters, LSTM hidden dimensions, isolation forest contamination thresholds, and the rationale for each choice (grid search, Bayesian optimization, domain-expert priors).

## 5. Risk Assessment and Management

- [x] **5.1** Description of known and foreseeable risks
  - **RAD-AI Section:** arc42 §11 (Risks) + AI Debt Register
  - **Evidence:** See `arc42-extensions/11-ai-debt-register.md`, all debt categories — documents boundary erosion risks (BE-001: MDL-003 tightly coupled to sensor preprocessing), entanglement risks (EN-001: MDL-001 route features derived from MDL-002 demand forecasts creating cascading failure), hidden feedback loops (FL-001: route suggestions influence future demand patterns), data dependency debt (DD-001: dependency on third-party weather API with no SLA guarantee), and model staleness risks (MS-001: seasonal drift in demand patterns). Debt Summary Dashboard provides severity overview.

- [x] **5.2** Description of risk management measures
  - **RAD-AI Section:** AI Debt Register (mitigation plans) + Non-Determinism Boundary
  - **Evidence:** See `arc42-extensions/11-ai-debt-register.md`, Mitigation Plan columns per debt item — documents mitigations including circuit breakers for cascading failures, fallback to rule-based routing, human-in-the-loop escalation for MDL-003 anomaly alerts, and retraining schedules. See `c4-diagrams/non-determinism-boundary.md`, Degradation Behavior table and Confidence Thresholds table — documents fallback strategies (rule-based defaults, cached predictions, human escalation) for each AI component at defined confidence levels.

## 6. Lifecycle Changes

- [x] **6.1** Description of changes made to the system through its lifecycle
  - **RAD-AI Section:** Model Registry View (version history, deployment history)
  - **Evidence:** See `arc42-extensions/05-model-registry-view.md`, Versioning tables per model — full version history including MDL-001 (v1.0 through v2.3.1, 7 versions over 14 months), MDL-002 (v1.0 through v1.4.0, 5 versions with architecture change at v1.2), MDL-003 (v1.0 through v3.1.0, major version bump at v2.0 when rule engine was integrated alongside Isolation Forest). Each version entry includes training date, dataset version, key metric changes, deployment date, and change notes.

## 7. Performance and Accuracy

- [x] **7.1** Description of performance metrics
  - **RAD-AI Section:** AI Quality Scenarios + Model Registry View (performance baselines)
  - **Evidence:** See `arc42-extensions/10-ai-quality-scenarios.md`, Quality Scenarios for Model Freshness, Drift Tolerance, Explainability, Fairness, and Robustness — each with source, stimulus, artifact, environment, response, and response measure. See `arc42-extensions/05-model-registry-view.md`, Performance Baselines tables per model — documents primary metrics (MDL-001: route optimality score 0.91, MDL-002: MAPE 8.3%, MDL-003: F1 0.94 with recall 0.97 for safety-critical events) with baselines, current values, and alert thresholds.

- [x] **7.2** Description of accuracy levels and expected foreseeable unintended outcomes
  - **RAD-AI Section:** AI Quality Scenarios + Non-Determinism Boundary (degradation behavior)
  - **Evidence:** See `arc42-extensions/10-ai-quality-scenarios.md`, Robustness and Drift Tolerance scenarios — documents expected degradation under adversarial inputs and distribution shifts. See `c4-diagrams/non-determinism-boundary.md`, Degradation Behavior table — specifies behavior when models are unavailable (rule-based fallback), produce low-confidence outputs (human review queue for MDL-003, cached responses for MDL-001/MDL-002), or return errors (stale prediction with warning flag). Propagation Rules table documents cascading confidence composition across model dependencies.

## 8. Human Oversight

- [x] **8.1** Description of human oversight measures
  - **RAD-AI Section:** Responsible AI Concepts (human oversight) + AI Component Stereotypes (`<<HITL>>`)
  - **Evidence:** See `arc42-extensions/08-responsible-ai-concepts.md`, Human Oversight section, Oversight Mechanisms table — documents human-in-the-loop review for MDL-003 anomaly alerts with confidence below 0.85 (domain expert review, < 15 min response SLA for safety-critical alerts), periodic audit review of MDL-001/MDL-002 outputs (quarterly by transport authority). See `c4-diagrams/ai-component-stereotypes.md`, Container Diagram showing `<<HITL>>` components: Safety Review Panel for MDL-003 and Compliance Audit Dashboard for all models.

- [x] **8.2** Description of how the system supports human oversight
  - **RAD-AI Section:** Responsible AI Concepts (override and escalation)
  - **Evidence:** See `arc42-extensions/08-responsible-ai-concepts.md`, Override and Escalation section — documents the override mechanism (transport authority operators can override any MDL-003 anomaly classification via the Safety Dashboard with mandatory justification logging), override logging (all overrides recorded with operator ID, timestamp, original prediction, override decision, and rationale in immutable audit log), and escalation path (MDL-003 high-severity anomalies auto-escalate to on-call safety officer within 5 minutes; unresolved anomalies escalate to city emergency coordinator within 30 minutes).

## 9. Post-Market Monitoring

- [x] **9.1** Description of post-market monitoring system
  - **RAD-AI Section:** Operational AI View (monitoring architecture, drift detection)
  - **Evidence:** See `arc42-extensions/12-operational-ai-view.md`, Monitoring Architecture section — Drift Detection table documents monitoring for all three models (MDL-001: PSI on feature distributions, daily; MDL-002: KS test on demand distribution, hourly; MDL-003: accuracy monitoring on labeled feedback, continuous). Model Health Metrics table tracks accuracy, latency, throughput, and error rates with warning and critical thresholds per model. Monitoring Diagram shows the flow from inference services through monitoring agents to alerting and retraining triggers. Retraining Pipeline section documents scheduled (monthly) and drift-triggered retraining with human approval gates.

- [x] **9.2** Description of logging and audit capabilities
  - **RAD-AI Section:** Operational AI View (incident response) + Responsible AI Concepts (transparency)
  - **Evidence:** See `arc42-extensions/12-operational-ai-view.md`, Incident Response table — documents severity levels P1 through P3 with response procedures (P1: auto-rollback for MDL-003 serving errors > 1%, page on-call safety officer; P2: alert ML team for accuracy degradation; P3: log and schedule retrain for minor drift). Rollback Procedures table specifies recovery time targets (< 5 min for MDL-003). See `arc42-extensions/08-responsible-ai-concepts.md`, Transparency section, Disclosure Requirements table — documents AI disclosure labels for end-user route recommendations and public-facing anomaly alerts per EU AI Act Art. 52. Privacy section documents data retention (2 years for predictions, 5 years for safety-critical MDL-003 decisions) and audit trail immutability.

---

## Summary

| Section | Items | Completed | Coverage |
|---------|-------|-----------|----------|
| 1. General Description | 4 | 4/4 | 100% |
| 2. System Elements | 3 | 3/3 | 100% |
| 3. Data Governance | 4 | 4/4 | 100% |
| 4. Training | 2 | 2/2 | 100% |
| 5. Risk Assessment | 2 | 2/2 | 100% |
| 6. Lifecycle Changes | 1 | 1/1 | 100% |
| 7. Performance | 2 | 2/2 | 100% |
| 8. Human Oversight | 2 | 2/2 | 100% |
| 9. Post-Market Monitoring | 2 | 2/2 | 100% |
| **Total** | **22** | **22/22** | **100%** |
