# arc42 Extensions Reference

**Detailed specification of RAD-AI's eight backward-compatible extensions to the arc42 template**

---

## Table of Contents

1. [Introduction](#introduction)
2. [E1: AI Boundary Delineation](#e1-ai-boundary-delineation)
3. [E2: Model Registry View](#e2-model-registry-view)
4. [E3: Data Pipeline View](#e3-data-pipeline-view)
5. [E4: Responsible AI Concepts](#e4-responsible-ai-concepts)
6. [E5: AI Decision Records (AI-ADR)](#e5-ai-decision-records-ai-adr)
7. [E6: AI Quality Scenarios](#e6-ai-quality-scenarios)
8. [E7: AI Debt Register](#e7-ai-debt-register)
9. [E8: Operational AI View](#e8-operational-ai-view)
10. [Summary Table](#summary-table)
11. [Cross-Extension Dependencies](#cross-extension-dependencies)

---

## Introduction

The arc42 template provides a pragmatic twelve-section structure for documenting software architectures. Its twelve sections are: (1) Introduction and Goals, (2) Constraints, (3) Context and Scope, (4) Solution Strategy, (5) Building Block View, (6) Runtime View, (7) Deployment View, (8) Cross-Cutting Concepts, (9) Architecture Decisions, (10) Quality Requirements, (11) Risks and Technical Debt, and (12) Glossary.

RAD-AI extends this template to address five identified documentation gaps (G1 through G5) that arise when arc42 is applied to AI-augmented ecosystems. These gaps are structural rather than domain-specific: comparative analysis on two unrelated production AI platforms (Uber Michelangelo and Netflix Metaflow/Maestro) reveals identical deficiency patterns.

RAD-AI provides eight extensions:

- **Seven extensions augment existing arc42 sections** (E1 through E7), adding AI-specific subsections, artifacts, and metadata to sections 3, 5, 6, 8, 9, 10, and 11.
- **One extension adds a new section** (E8: Operational AI View), covering post-deployment operational concerns unique to AI systems that have no equivalent in the standard arc42 template.

### Design Principles

All extensions follow three principles:

1. **Backward compatibility.** Every extension augments rather than replaces the standard arc42 content. Existing documentation remains valid; the extensions add AI-specific subsections alongside existing material.
2. **Incremental adoption.** Practitioners can adopt extensions individually, starting with those most relevant to their context. There is no requirement to adopt all eight at once.
3. **Traceability.** Each extension traces to one or more identified gaps (G1 through G5), providing clear rationale for its inclusion.

### Documentation Gaps Addressed

| Gap | Description |
|-----|-------------|
| G1 | No AI-specific arc42 sections for model lifecycle, data pipelines, or drift |
| G2 | No C4 diagram types for ML components or non-determinism boundaries |
| G3 | No mapping between EU AI Act Annex IV and arc42/C4 sections |
| G4 | No AI-specific Architecture Decision Record templates |
| G5 | No integration of Model/Data Cards into architecture documentation |

### Suggested Adoption Path

For teams new to RAD-AI, a three-stage adoption path is recommended:

- **Stage 1:** E1 (AI Boundary Delineation) and E2 (Model Registry View). Establishes visibility over which components are AI-driven and what models are deployed.
- **Stage 2:** E5 (AI-ADR) and E6 (AI Quality Scenarios). Captures decision rationale and measurable quality commitments.
- **Stage 3:** E8 (Operational AI View) and E7 (AI Debt Register). Adds operational maturity and long-term maintainability tracking.

Extensions E3 (Data Pipeline View) and E4 (Responsible AI Concepts) can be introduced at any stage, depending on organizational priorities around data governance and responsible AI.

---

## E1: AI Boundary Delineation

**Extends:** arc42 Section 3 (Context and Scope)
**Addresses Gaps:** G1, G2
**Key Artifact:** Annotated context diagram with AI boundaries

### Purpose

Standard arc42 Section 3 defines the system's external context (users, external systems, interfaces) and internal scope (subsystem boundaries). It assumes all components are deterministic: given the same inputs, they produce the same outputs. AI-augmented systems violate this assumption. A recommendation engine, a fraud detection model, or a demand forecasting service produces probabilistic outputs that vary with training data, input distributions, and model versions.

E1 requires practitioners to explicitly mark deterministic versus non-deterministic system boundaries in the context diagram. This visual distinction gives all stakeholders (developers, architects, product managers, regulators) an immediate indication of where probabilistic behavior enters the system and what guarantees each AI interface provides.

### What It Documents

#### AI Components Inventory

A structured table enumerating all AI components in the system. Each row captures:

| Column | Description | Example |
|--------|-------------|---------|
| Component | Name of the AI component | Route Optimization Service |
| Type | Classification of the AI approach | ML Model (supervised), Rule-based AI, LLM, Ensemble |
| Input Domain | What the component consumes | GPS traces, weather data, historical trip records |
| Output Domain | What the component produces | Predicted ETA (continuous), Optimal route (structured) |
| Confidence Range | Expected output quality bounds | Precision >= 0.92 at P95 latency < 50ms |
| Fallback Strategy | Behavior when the AI component fails or degrades | Rule-based default, cached last-known-good, human escalation |

#### System Boundary Diagram

An annotated version of the standard arc42 context diagram that partitions the system into two regions:

- **Deterministic region:** Components with stable, fully specified behavior (e.g., API gateways, databases, rule engines, static configuration services).
- **Non-deterministic region:** Components whose outputs are probabilistic or data-dependent (e.g., ML models, ensemble pipelines, generative AI services).

The boundary between these regions is drawn as a clearly marked dividing line. Each crossing point from the deterministic into the non-deterministic region (and vice versa) is annotated with a boundary contract.

#### Four-Part Boundary Contract

Every boundary crossing between deterministic and non-deterministic regions must be annotated with the following four properties:

1. **Output type.** The nature of the AI component's output:
   - *Categorical:* Discrete class labels (e.g., fraud/not-fraud, sentiment categories).
   - *Continuous:* Numerical predictions (e.g., predicted ETA in minutes, demand forecast in units).
   - *Generative:* Open-ended content (e.g., text, images, code). Generative outputs require additional documentation of guardrails and content policies.

2. **Confidence specification.** A quantitative statement of expected output quality under normal operating conditions. This should include the primary metric, its threshold, and any latency or throughput constraints. Examples:
   - "Precision >= 0.92 at P95 latency < 50ms"
   - "RMSE < 3.2 minutes for 95% of predictions"
   - "Toxicity score < 0.05 for 99.5% of generated outputs"

3. **Update frequency.** How often the underlying model is refreshed or retrained:
   - *Scheduled:* Regular cadence (e.g., daily, weekly, monthly).
   - *Event-triggered:* Retrained when specific conditions are met (e.g., performance degradation, data drift detection).
   - *Continuous:* Online learning with incremental updates.
   - *Static:* No retraining planned (document the rationale).

4. **Fallback behavior.** What happens when the AI component is unavailable, produces low-confidence outputs, or is detected as operating outside its valid input distribution:
   - *Rule-based default:* A deterministic fallback logic takes over (e.g., static routing rules, average historical values).
   - *Cached last-known-good:* The system serves the most recent valid prediction.
   - *Human escalation:* The request is routed to a human operator for manual handling.
   - *Graceful degradation:* The system continues with reduced functionality (document what functionality is lost).
   - *Circuit breaker:* The AI component is taken offline entirely until recovery conditions are met.

#### Failure Modes

For each AI component, document the known failure modes and their consequences:

| Failure Mode | Description | Impact | Mitigation |
|-------------|-------------|--------|------------|
| Model staleness | Model not retrained despite distribution shift | Degraded prediction quality | Drift monitoring (E8), retraining triggers |
| Input out of distribution | Inputs outside the training data distribution | Unpredictable outputs | Input validation, confidence thresholds |
| Latency spike | Inference time exceeds SLA | Downstream timeout | Timeout with fallback, model optimization |
| Catastrophic forgetting | Retraining on new data degrades performance on older patterns | Loss of previously learned capabilities | Validation on held-out historical data |

#### External AI Dependencies

If the system consumes third-party AI services (hosted models, AI APIs, cloud ML endpoints), document each dependency with:

- Provider and service name
- API contract and versioning policy
- SLA guarantees (availability, latency, throughput)
- Data handling and privacy implications
- Fallback strategy if the external service is unavailable

### Relationship to C4-E3

E1 operates at the arc42 documentation level (textual descriptions, tables, annotated diagrams). The complementary C4-E3 (Non-Determinism Boundary Diagram) applies the same boundary concept at the C4 diagram level, providing a visual overlay that partitions the C4 Container or Component diagram into deterministic and non-deterministic regions. The boundary contracts defined in E1 are the authoritative source; C4-E3 diagrams reference them.

### EU AI Act Relevance

E1 supports Annex IV Category 1 (General system description) by documenting AI component capabilities, limitations, and boundary conditions. It also supports Category 3 (Design specifications and architecture) by making the architectural distinction between deterministic and non-deterministic components explicit.

---

## E2: Model Registry View

**Extends:** arc42 Section 5 (Building Block View)
**Addresses Gaps:** G1, G5
**Key Artifact:** Model registry table and annotated building block diagram

### Purpose

Standard arc42 Section 5 decomposes the system into building blocks (modules, components, packages) with their responsibilities and interfaces. AI models are typically invisible in this view: a recommendation engine appears as a single box indistinguishable from a rules engine or a database adapter. Critical model metadata (version, framework, training data, performance baselines) has no structured home.

E2 elevates AI models to first-class building blocks. Each model receives its own entry in the building block hierarchy, with structured metadata that captures the information practitioners need for governance, debugging, and compliance.

### What It Documents

#### Model Inventory Table

A registry of all AI models in the system:

| Column | Description | Example |
|--------|-------------|---------|
| Model ID | Unique identifier for the model | route-opt-v3.2.1 |
| Name | Human-readable name | Route Optimization Model |
| Task | The ML task the model performs | Regression (ETA prediction) |
| Framework | ML framework used | XGBoost 1.7.6 |
| Version | Current deployed version | 3.2.1 |
| Status | Deployment status | Production / Canary / Shadow / Retired |
| Owner | Responsible team or individual | ML Platform Team |
| Last Retrained | Date of most recent training run | 2026-01-15 |

#### Per-Model Detail Sections

For each model in the inventory, the following subsections should be completed:

**Versioning History.** A log of model versions with dates, key changes, performance deltas, and deployment status transitions. This creates an auditable trail of how the model has evolved.

| Version | Date | Key Changes | Primary Metric (MAE) | Status |
|---------|------|-------------|----------------------|--------|
| 3.2.1 | 2026-01-15 | Added weather features | 4.1 min | Production |
| 3.1.0 | 2025-11-02 | Retrained on Q3 data | 4.3 min | Retired |
| 3.0.0 | 2025-08-20 | Architecture change to XGBoost | 4.5 min | Retired |
| 2.4.2 | 2025-06-10 | Last linear regression version | 5.8 min | Archived |

**Hyperparameters.** A snapshot of the current model's key hyperparameters. This is not a complete training configuration dump; rather, it captures the parameters most relevant to architectural decisions and performance characteristics.

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| max_depth | 8 | Balanced accuracy vs. inference latency |
| n_estimators | 500 | Plateau in validation MAE beyond 500 |
| learning_rate | 0.05 | Stability during incremental retraining |
| subsample | 0.8 | Regularization for GPS noise |

**Training Data Lineage.** Documentation of the data used to train the current model version:

- Dataset identifier and version (or hash)
- Source systems and collection period
- Size (rows, features)
- Known biases or limitations in the training data
- Preprocessing steps applied before training
- Link to the corresponding Data Card (if E3 is adopted)

**Upstream and Downstream Dependencies.** Each model's architectural dependencies:

- *Upstream:* What data sources, feature stores, or other models feed into this model.
- *Downstream:* What services, APIs, or other models consume this model's outputs.

This dependency mapping is essential for understanding the blast radius of model changes and for tracing cascading effects in ecosystem settings.

**Performance Baselines.** The acceptance thresholds that the model must meet to remain in production:

| Metric | Threshold | Current Value | Measurement Frequency |
|--------|-----------|---------------|----------------------|
| MAE | < 5.5 min | 4.1 min | Daily |
| P95 Latency | < 50 ms | 32 ms | Continuous |
| Throughput | > 10K req/s | 14K req/s | Continuous |

#### Integration with Model Cards

E2 integrates with Model Cards (Mitchell et al., 2019) by treating each Model Card as a linked sub-artifact of its corresponding model building block. The Model Card provides the detailed reporting (intended use, out-of-scope uses, ethical considerations, evaluation results across demographic groups), while E2 provides the architectural context (where the model sits in the system, what it depends on, what depends on it).

Practitioners using existing Model Card tooling (e.g., Hugging Face model cards, Google Model Card Toolkit) can link to those artifacts from the E2 registry entry rather than duplicating content.

#### Integration with Model Registry Tools

E2 is designed to complement, not replace, operational model registry tools:

- **MLflow Model Registry:** E2 entries can reference MLflow experiment IDs and run IDs. The architectural metadata (dependencies, fallback strategies, performance baselines) augments MLflow's experiment tracking.
- **Weights and Biases (W&B):** E2 entries can link to W&B project URLs for detailed experiment history.
- **DVC (Data Version Control):** Training data lineage in E2 can reference DVC file hashes for reproducibility.
- **Custom registries:** Organizations with internal model management systems can use E2 as the architectural overlay that connects model metadata to the broader system documentation.

### EU AI Act Relevance

E2 supports Annex IV Category 2 (System elements and development process) by providing a structured inventory of all AI components with their versioning history. It also supports Category 7 (Lifecycle changes) by making model version transitions auditable.

---

## E3: Data Pipeline View

**Extends:** arc42 Section 6 (Runtime View)
**Addresses Gaps:** G1, G5
**Key Artifact:** Data pipeline diagram with quality gates

### Purpose

Standard arc42 Section 6 documents runtime behavior through sequence diagrams, activity diagrams, or message flow diagrams. These capture request/response interactions between components but cannot represent the continuous data flows that define ML system behavior: data collection, preprocessing, feature engineering, training, inference, and feedback loops.

E3 documents the complete ML data flow from raw data collection through model inference and back through feedback loops. Each pipeline stage is annotated with quality gates that specify what checks are performed, what thresholds are applied, and what happens when checks fail.

### What It Documents

#### Pipeline Overview Diagram

A visual representation of the end-to-end data flow, typically structured as a directed acyclic graph (DAG) with the following stages:

```
Collection -> Preprocessing -> Feature Engineering -> Training -> Inference -> Feedback
```

Each stage is represented as a named node. Arrows between stages carry annotations describing the data format, volume, and freshness requirements. Quality gates are placed between stages as checkpoints.

The diagram should distinguish between:

- **Batch pipelines:** Scheduled data processing (e.g., daily retraining, weekly feature computation).
- **Streaming pipelines:** Real-time data processing (e.g., event-driven feature updates, online inference).
- **Hybrid pipelines:** Combinations of batch and streaming components.

#### Pipeline Inventory Table

| Column | Description | Example |
|--------|-------------|---------|
| Pipeline ID | Unique identifier | pipe-feature-eng-001 |
| Name | Human-readable name | Trip Feature Engineering Pipeline |
| Type | Batch, Streaming, or Hybrid | Hybrid (batch features + streaming events) |
| Schedule | Execution cadence | Daily at 02:00 UTC (batch); continuous (streaming) |
| SLA | Maximum acceptable end-to-end latency | Batch: 4 hours; Streaming: < 500ms |
| Owner | Responsible team | Data Engineering Team |

#### Quality Gates

Quality gates are the central artifact of E3. Each gate is placed at a specific point in the pipeline and specifies three properties:

1. **Check type.** The category of validation performed:
   - *Schema conformance:* Validates that the data matches the expected schema (column names, data types, value ranges).
   - *Distribution test:* Statistical tests comparing current data distributions to reference distributions (e.g., Kolmogorov-Smirnov test, Population Stability Index).
   - *Completeness check:* Verifies that required fields are present and non-null at acceptable rates.
   - *Freshness check:* Confirms that the data is sufficiently recent for the downstream consumer.
   - *Volume check:* Verifies that the data volume is within expected bounds (guards against both missing data and data explosions).
   - *Cross-feature consistency:* Validates relationships between features that should hold (e.g., departure time < arrival time).

2. **Threshold.** A quantitative criterion that determines pass or fail:
   - KS-statistic < 0.1 (distribution stability)
   - Null rate < 1% for required fields
   - Data freshness < 24 hours
   - Row count within 20% of historical average
   - Schema match: 100% of required columns present

3. **Action on failure.** What happens when the check fails:
   - *Halt pipeline:* Stop processing and alert the pipeline owner. No downstream data is produced. Use for critical quality gates where bad data could corrupt models.
   - *Alert and continue:* Log the failure, send an alert, and continue processing. Use for informational checks where the risk of bad data is acceptable in the short term.
   - *Activate fallback:* Switch to an alternative data source or cached data. Use when downstream consumers cannot tolerate missing data.

#### Quality Gate Table Example

| Gate ID | Location | Check Type | Threshold | Action on Failure |
|---------|----------|------------|-----------|-------------------|
| QG-001 | After collection | Schema conformance | 100% required columns | Halt pipeline |
| QG-002 | After collection | Volume check | Row count within 20% of 7-day avg | Alert and continue |
| QG-003 | After preprocessing | Completeness check | Null rate < 1% for required fields | Halt pipeline |
| QG-004 | Before training | Distribution test | KS-statistic < 0.1 vs. reference | Halt pipeline |
| QG-005 | Before training | Freshness check | Data < 24 hours old | Activate fallback (cached features) |
| QG-006 | After inference | Confidence check | Mean confidence > 0.7 | Alert and continue |

#### Feature Store Documentation

If the system uses a feature store (centralized repository of curated features shared across models), document:

- Feature store technology (e.g., Feast, Tecton, custom)
- Feature catalog: list of available features with descriptions, data types, and refresh cadences
- Feature ownership: which team owns each feature group
- Serving modes: online (low-latency lookup) vs. offline (batch retrieval)
- Feature versioning: how feature definitions are versioned and how consumers handle version transitions

#### Feedback Loops

Feedback loops occur when model outputs influence future training data. These are architecturally significant because they can create self-reinforcing biases or instabilities. For each feedback loop, document:

| Property | Description |
|----------|-------------|
| Loop description | How the model's output feeds back into training data |
| Delay | Time between prediction and feedback signal (minutes, hours, days) |
| Risk | Potential negative consequences (e.g., popularity bias, filter bubbles, self-fulfilling prophecies) |
| Mitigation | Countermeasures in place (e.g., exploration/exploitation balance, randomized holdouts, feedback delay monitoring) |
| Monitoring | How the feedback loop's health is tracked |

#### Integration with Data Cards

E3 integrates with Data Cards (Google, 2022) by attaching Data Card artifacts at data source nodes in the pipeline diagram. Each Data Card documents the provenance, composition, collection methodology, intended uses, and known limitations of a dataset. The Data Card provides the governance and transparency layer; E3 provides the architectural context showing how that data flows through the system.

### EU AI Act Relevance

E3 directly supports Annex IV Category 4 (Data and data governance), which requires documentation of data provenance, preparation procedures, and data quality measures. It also supports Category 5 (Training methodologies) by documenting the data processing pipeline that precedes model training.

---

## E4: Responsible AI Concepts

**Extends:** arc42 Section 8 (Cross-Cutting Concepts)
**Addresses Gap:** G1
**Key Artifact:** Responsible AI concern matrix

### Purpose

Standard arc42 Section 8 documents cross-cutting concerns that affect multiple building blocks: persistence strategies, UI patterns, error handling, logging, security, and similar technical concerns. For AI-augmented systems, five additional cross-cutting concerns require structured documentation: fairness, explainability, human oversight, privacy, and safety. These concerns cut across all AI components and often involve trade-offs that must be explicitly recorded.

E4 provides a structured concern matrix and detailed subsections for each responsible AI category, drawing on responsible AI pattern catalogues (Lu et al., 2024) and the documentation requirements of the EU AI Act.

### What It Documents

#### Responsible AI Concern Matrix

The central artifact is a matrix where rows represent AI components and columns represent the five responsible AI concern categories. Each cell documents the applicable metric or method, the acceptance threshold, the monitoring frequency, and the responsible party.

| Component | Fairness | Explainability | Human Oversight | Privacy | Safety |
|-----------|----------|----------------|-----------------|---------|--------|
| Route Optimization | Demographic parity ratio > 0.8 across districts; monitored weekly; ML Team | SHAP values per route for regulatory audits; on-demand; ML Team | Operator override via dashboard; always available; Ops Team | GPS data anonymized (k=5); continuous; Privacy Team | Fallback to static routes on model failure; continuous; SRE Team |
| Anomaly Detection | Equal false positive rate across vehicle types; monitored daily; Safety Team | Feature importance ranking for each alert; per-alert; Safety Team | Human review required for high-severity alerts; per-alert; Safety Team | Sensor data retention < 30 days; daily audit; Privacy Team | Circuit breaker at confidence < 0.6; continuous; SRE Team |

#### Fairness

For each AI component, document:

- **Protected attributes:** Which attributes are considered protected in the operational context (e.g., geographic district, demographic group, vehicle type).
- **Fairness metric:** The quantitative metric used to measure fairness (e.g., demographic parity, equalized odds, predictive parity).
- **Acceptance threshold:** The minimum acceptable value for the fairness metric.
- **Bias detection architecture:** Where in the pipeline bias is detected:
  - *Pre-training:* Dataset analysis for representation imbalances.
  - *In-training:* Fairness constraints or adversarial debiasing during model training.
  - *Post-training:* Evaluation of trained models on fairness benchmarks before deployment.
  - *In-production:* Continuous monitoring of fairness metrics on live predictions.
- **Remediation procedures:** What actions are taken when fairness thresholds are breached.

#### Explainability

For each AI component, document:

- **Explainability requirement level:** The degree of explanation required, which may vary by stakeholder:
  - *Global explanations:* Overall model behavior (e.g., feature importance rankings, partial dependence plots).
  - *Local explanations:* Per-prediction explanations (e.g., SHAP values for individual predictions, counterfactual examples).
  - *No explanation required:* Document the rationale for this decision.
- **Methods:** The explainability techniques used (e.g., SHAP, LIME, attention visualization, counterfactual explanations, decision tree surrogates).
- **Integration points:** Where explanations are surfaced in the system (e.g., user-facing interfaces, audit logs, operator dashboards, regulatory reports).
- **Audience:** Who consumes the explanations (end users, operators, auditors, regulators) and what level of technical detail is appropriate for each audience.

#### Human Oversight

For each AI component, document:

- **Oversight mechanism:** How humans can monitor and intervene in the AI component's operation:
  - *Human-in-the-loop:* A human must approve every AI output before it takes effect.
  - *Human-on-the-loop:* The AI operates autonomously, but a human can intervene when alerted.
  - *Human-in-command:* A human can override, disable, or roll back the AI component at any time.
- **Trigger conditions:** Under what circumstances human intervention is expected or required (e.g., low confidence scores, anomalous outputs, regulatory mandate).
- **Authority:** Who has the authority to override the AI component's decisions, and what organizational approval is needed.
- **Override procedures:** Step-by-step procedures for overriding or disabling the AI component, including downstream effects.
- **Escalation procedures:** How issues that cannot be resolved at the operator level are escalated (e.g., P1 incidents to on-call ML engineer, regulatory concerns to legal team).

#### Privacy

For each AI component, document:

- **Data classification:** The sensitivity level of data processed by the component (e.g., public, internal, confidential, restricted).
- **Privacy-preserving techniques:** Methods used to protect sensitive data (e.g., differential privacy, k-anonymity, federated learning, data minimization).
- **Data retention policies:** How long input data, predictions, and training data are retained.
- **Access controls:** Who can access the model's training data, predictions, and internal state.
- **Cross-border data handling:** If the system operates across jurisdictions, how data sovereignty requirements are addressed.

#### Safety

For each AI component, document:

- **Risk classification:** The component's risk level under the EU AI Act (prohibited, high-risk, limited risk, minimal risk) or other applicable regulatory frameworks.
- **Safety-critical failure modes:** Failure modes with safety implications (cross-reference with E1 failure mode table).
- **Safety mechanisms:** Technical controls that prevent or mitigate unsafe outputs (e.g., output clamping, confidence thresholds, circuit breakers).
- **Incident history:** A log of past safety-relevant incidents and the actions taken.

### EU AI Act Relevance

E4 directly supports Annex IV Category 9 (Human oversight measures) and contributes to Category 6 (Risk assessment and management). The fairness and transparency documentation supports the Act's general transparency requirements (Article 13) and the specific documentation obligations for high-risk systems (Article 11).

---

## E5: AI Decision Records (AI-ADR)

**Extends:** arc42 Section 9 (Architecture Decisions)
**Addresses Gap:** G4
**Key Artifact:** AI-ADR template with seven AI-specific fields

### Purpose

Standard arc42 Section 9 captures architecture decisions using Architecture Decision Records (ADRs), commonly following the Markdown Any Decision Records (MADR) format. MADR provides fields for title, status, context, decision drivers, alternatives considered, decision outcome, and consequences. These fields are sufficient for traditional architectural choices (e.g., selecting a database, choosing a communication protocol, deciding on a deployment topology) but cannot capture the unique considerations of AI-related decisions.

AI architectural decisions involve additional dimensions: the choice between model architectures, the characteristics of training data, fairness and bias trade-offs, expected model lifetimes, retraining strategies, explainability requirements, and regulatory compliance. E5 extends MADR with seven AI-specific fields that capture these dimensions.

### Standard MADR Fields (Retained)

The AI-ADR retains all standard MADR fields:

| Field | Description |
|-------|-------------|
| Title | Short noun phrase describing the decision |
| Status | Proposed, Accepted, Deprecated, Superseded |
| Context | The issue motivating the decision, including relevant constraints and forces |
| Decision Drivers | The factors that influenced the decision (e.g., performance, cost, timeline, regulatory) |
| Considered Alternatives | The options that were evaluated |
| Decision Outcome | The chosen option, with brief justification |
| Consequences | Positive, negative, and neutral consequences of the decision |

### Seven AI-Specific Fields

The following fields are added to the standard MADR template:

#### 1. Model Alternatives Considered

A structured comparison of the ML model architectures or approaches that were evaluated:

| Alternative | Type | Pros | Cons |
|-------------|------|------|------|
| XGBoost | Gradient-boosted trees | High explainability, fast inference, robust to missing data | Limited sequence modeling, manual feature engineering |
| LSTM | Recurrent neural network | Strong sequence modeling, automatic feature learning | Low explainability, higher inference latency, more training data needed |
| Linear Regression | Linear model | Maximum explainability, trivial deployment | Insufficient accuracy for non-linear patterns |

This field ensures that the rationale for choosing one model architecture over another is recorded alongside the traditional architectural alternatives.

#### 2. Dataset Characteristics

Structured description of the training data:

| Property | Value |
|----------|-------|
| Size | 2.1 million trip records |
| Feature count | 47 features (32 numeric, 15 categorical) |
| Collection period | 18 months (July 2024 through December 2025) |
| Class balance | N/A (regression task); target distribution: mean 12.3 min, std 4.7 min |
| Known biases | Under-representation of suburban districts (< 15% of records); weekend coverage sparse before 2025 |
| Data freshness | Updated daily; oldest training sample is 18 months |

This field creates a traceable link between the model decision and the data it was trained on, which is critical for debugging, auditing, and compliance.

#### 3. Fairness and Bias Trade-offs

Explicit documentation of how fairness concerns influenced the decision:

| Property | Value |
|----------|-------|
| Protected attributes | Geographic district, time of day |
| Fairness metric | Maximum prediction gap across districts |
| Accepted trade-off | 15% maximum prediction gap across districts (higher gap accepted in exchange for better overall accuracy) |
| Monitoring | Weekly fairness report; breach triggers rebalancing of training data |

This field ensures that fairness considerations are part of the architectural decision record rather than an afterthought.

#### 4. Expected Model Lifetime

The anticipated duration before the model requires a full refit (as opposed to incremental retraining):

- **Lifetime estimate:** Approximately 12 months before full architecture refit.
- **Rationale:** Traffic patterns in the deployment city are relatively stable year-over-year; seasonal variations are captured by features rather than model structure changes.
- **Trigger for reassessment:** Significant infrastructure changes (new transit lines, major road construction) or sustained performance degradation beyond retraining-addressable levels.

#### 5. Retraining Trigger

The conditions under which the model should be retrained:

| Trigger Type | Condition | Action |
|-------------|-----------|--------|
| Performance-based | MAE > 5.5 minutes for 3 consecutive days | Automated retraining with human approval |
| Drift-based | Feature distribution shift > 2 sigma on >= 3 features | Alert ML team; manual investigation before retraining |
| Scheduled | Every 30 days | Automated retraining with automated validation |
| Event-based | New data source integrated (e.g., bike-sharing data) | Manual retraining with architecture review |

#### 6. Explainability Requirements

The explainability obligations and methods chosen for this model:

| Property | Value |
|----------|-------|
| Method | SHAP (TreeExplainer for XGBoost) |
| Scope | Per-route explanations for regulatory audits; global feature importance for quarterly model reviews |
| Audience | City transport authority (regulatory audits), ML team (model review), operations team (anomaly investigation) |
| Integration | SHAP values stored in prediction logs; accessible via audit API endpoint |

#### 7. Regulatory Compliance

How this decision relates to applicable regulations:

| Property | Value |
|----------|-------|
| Applicable regulations | EU AI Act (Regulation 2024/1689) |
| Risk classification | Not high-risk (outside Article 6 scope); transparency per Article 50 |
| Annex IV items addressed | Category 5 (training methodology), Category 3 (design specification) |
| Documentation artifacts produced | Model Card (linked), training data lineage (E2), quality scenarios (E6) |

### Illustrative AI-ADR Example

The following condensed example shows an AI-ADR for a route optimization service:

| Field | Value |
|-------|-------|
| **Title** | Use gradient-boosted trees for route optimization |
| **Status** | Accepted |
| **Context** | Route optimization requires explainable predictions for public-service accountability |
| **Decision Drivers** | Explainability (regulatory), inference latency (< 50ms), accuracy (MAE < 5.5 min) |
| **Decision Outcome** | XGBoost (gradient-boosted trees) |
| **Consequences** | +Explainability, +Fast inference; -Lower sequence modeling capacity |
| **Model alternatives** | XGBoost, LSTM, linear regression |
| **Dataset** | 18 months, 2.1M trips, GPS + weather |
| **Fairness/bias** | Under-served districts monitored; max 15% prediction gap across districts |
| **Model lifetime** | Approximately 12 months before full refit |
| **Retraining trigger** | MAE > 5.5 min for 3 consecutive days |
| **Explainability** | SHAP values for per-route regulatory audits |
| **Regulatory** | Not high-risk (outside Art. 6 scope); transparency per Art. 50 |

### EU AI Act Relevance

E5 supports Annex IV Category 5 (Training methodologies and techniques) by recording the rationale for model selection, training data characteristics, and the trade-offs accepted. The regulatory compliance field directly maps decisions to Annex IV categories.

---

## E6: AI Quality Scenarios

**Extends:** arc42 Section 10 (Quality Requirements)
**Addresses Gap:** G1
**Key Artifact:** AI quality scenario table

### Purpose

Standard arc42 Section 10 captures quality requirements using quality scenarios following the established source-stimulus-artifact-environment-response-response measure format (Bass et al., 2021). This format works well for traditional quality attributes (performance, availability, security, modifiability) but cannot express AI-specific quality concerns: model freshness, drift tolerance, explainability timeliness, fairness constraints, and robustness to adversarial inputs.

E6 extends the quality scenario format with AI-specific sources, stimuli, environments, and response measures. It also introduces cross-component scenarios that capture quality concerns spanning multiple AI components in an ecosystem.

### AI-Specific Quality Attributes

E6 covers five AI-specific quality attributes that complement the standard software quality attributes:

#### Model Freshness

How recent the model's parameters are relative to the current data distribution. A stale model (trained on data that no longer represents current conditions) produces degraded predictions even if no code has changed.

- **Source:** Time elapsed since last retraining.
- **Stimulus:** Model age exceeds freshness SLA.
- **Response:** Automated retraining triggered; model validated before promotion.
- **Response measure:** Time from freshness breach to new model in production.

#### Drift Tolerance

The acceptable degree of data or concept drift before intervention is required. Data drift occurs when input feature distributions change; concept drift occurs when the relationship between inputs and outputs changes.

- **Source:** Monitoring system detecting distribution shift.
- **Stimulus:** Drift metric exceeds threshold (e.g., PSI > 0.2, KS-statistic > 0.1).
- **Response:** Alert generated; retraining initiated or fallback activated.
- **Response measure:** Time from drift detection to corrective action; impact on downstream consumers during the drift period.

#### Explainability

The ability to produce understandable explanations of model outputs within an acceptable time frame.

- **Source:** User, auditor, or regulatory request for explanation.
- **Stimulus:** Explanation requested for a specific prediction.
- **Response:** Explanation generated and delivered to the requester.
- **Response measure:** Explanation latency; explanation fidelity (how accurately the explanation reflects the model's actual reasoning).

#### Fairness

The absence of systematic bias in model outputs with respect to protected attributes.

- **Source:** Fairness monitoring system or external audit.
- **Stimulus:** Fairness metric falls below threshold.
- **Response:** Alert generated; model placed under review; remediation initiated.
- **Response measure:** Time to detection; time to remediation; affected population size during the fairness breach.

#### Robustness

Resilience of the model to adversarial inputs, edge cases, and out-of-distribution inputs.

- **Source:** Adversarial input, corrupted data, or out-of-distribution input.
- **Stimulus:** Input that is designed to mislead the model or falls outside training distribution.
- **Response:** Input flagged; fallback activated; incident logged.
- **Response measure:** Percentage of adversarial inputs correctly identified; impact on legitimate users during the attack.

### Scenario Format

Each AI quality scenario follows the standard six-part format:

| Field | Description |
|-------|-------------|
| Source | What triggers the scenario (e.g., data drift, model staleness, adversarial input, fairness monitoring) |
| Stimulus | The specific event or condition, with quantitative parameters |
| Artifact | The AI component or pipeline affected |
| Environment | The operational context (training, serving, monitoring, A/B testing) |
| Response | The system's expected behavior |
| Response Measure | Quantitative criteria for acceptable response |

### Illustrative Scenario: Single-Component Drift

| Field | Value |
|-------|-------|
| Source | Drift monitoring system |
| Stimulus | Feature distribution shift > 2 sigma on >= 3 features, sustained > 4 hours |
| Artifact | Demand Prediction Model |
| Environment | Production serving |
| Response | Confidence threshold tightened from 0.80 to 0.90; alert sent to ML team; retraining initiated |
| Response Measure | Drift detected within 1 hour; retraining completed within 6 hours; prediction quality degradation < 10% during drift period |

### Cross-Component Scenarios

A distinctive feature of E6 is support for cross-component quality scenarios. In AI-augmented ecosystems, quality degradation in one component can cascade to others. Standard quality scenarios address individual components; cross-component scenarios address the systemic effects.

#### Illustrative Scenario: Cascading Drift

| Field | Value |
|-------|-------|
| Source | Data drift in anomaly detection input features |
| Stimulus | Feature distribution shift > 2 sigma on >= 3 features, sustained > 12 hours |
| Artifact | Anomaly Detection System and Route Optimization Service (cross-component) |
| Environment | Production serving, all ecosystem components live |
| Response | Anomaly detection confidence threshold tightens from 0.85 to 0.95; route optimization activates last-known-good fallback. Drift detected within 2 hours, false safety alerts < 5%, downstream route MAE < 7 min |

This scenario illustrates a concern that is invisible in standard arc42: drift in the anomaly detection component causes false safety alerts, which trigger unnecessary rerouting in the route optimization component, degrading user experience. Only by documenting the cross-component dependency and the cascading effect can architects design appropriate mitigations.

### EU AI Act Relevance

E6 supports Annex IV Category 8 (Performance metrics and accuracy) by defining measurable quality criteria for each AI component. The cross-component scenarios are particularly relevant for ecosystem-level compliance, where the interaction between components may create risks not present in any individual component.

---

## E7: AI Debt Register

**Extends:** arc42 Section 11 (Risks and Technical Debt)
**Addresses Gap:** G1
**Key Artifact:** AI debt register with remediation plan

### Purpose

Standard arc42 Section 11 tracks risks and technical debt. For traditional software, technical debt categories are well understood: code duplication, outdated dependencies, missing tests, documentation gaps. ML systems introduce additional debt categories that are distinct from (and often more severe than) traditional technical debt. Sculley et al. (2015) identify ML-specific debt as a particularly insidious form of technical debt because it is often invisible, accumulates silently, and can have severe consequences.

E7 provides a structured register for tracking ML-specific technical debt alongside traditional architectural debt. Each entry includes severity assessment, affected components, a remediation plan, ownership, and current status.

### Debt Categories

E7 recognizes seven categories of ML-specific technical debt, drawn from Sculley et al. (2015) and Bogner et al. (2021):

#### 1. Boundary Erosion

Unclear or shifting boundaries between the ML component and the rest of the system. Signs include:
- ML logic leaking into application code (e.g., feature preprocessing duplicated in the serving layer).
- Application logic embedded in ML pipelines (e.g., business rules hardcoded in training scripts).
- No clear API contract between the ML model and its consumers.

**Architectural impact:** Makes it difficult to replace, update, or test the ML component independently.

#### 2. Entanglement (CACE: Changing Anything Changes Everything)

ML models create implicit dependencies between features. Changing one input feature can affect the learned weights of all other features, producing unpredictable changes in model behavior. Signs include:
- Adding a new feature improves accuracy on one segment but degrades it on another.
- Removing an unused feature changes model predictions.
- Feature correlations that are not explicitly documented.

**Architectural impact:** Makes incremental model improvement risky; changes require full regression testing.

#### 3. Hidden Feedback Loops

Model outputs influence future training data without explicit documentation or monitoring. Signs include:
- A recommendation model trained on click data, where the model's recommendations influence which items users see and click on.
- A fraud detection model whose alerts change the behavior of the system being monitored.
- No measurement of the feedback loop's effect on model behavior over time.

**Architectural impact:** Can create self-reinforcing biases, filter bubbles, or instabilities that are difficult to diagnose.

#### 4. Data Dependency Debt

Unstable, underutilized, or poorly governed data dependencies. Signs include:
- Input features derived from external systems with no SLA or change notification.
- Features that contribute minimally to model accuracy but add pipeline complexity.
- No ownership or documentation for upstream data sources.
- Legacy features retained "just in case" without evidence of their value.

**Architectural impact:** Increases pipeline fragility and maintenance burden; can cause silent model degradation when upstream data changes.

#### 5. Pipeline Debt

Complexity and fragility in the data and ML pipelines. Signs include:
- Pipeline code that is difficult to test, debug, or modify.
- Glue code connecting incompatible data formats or systems.
- No pipeline-level integration tests.
- Manual steps required for pipeline execution.

**Architectural impact:** Increases time to retrain, time to deploy, and risk of pipeline failures.

#### 6. Configuration Debt

Sprawling hyperparameters, thresholds, feature flags, and configuration values that are difficult to manage. Signs include:
- Hyperparameters scattered across multiple configuration files.
- Thresholds (e.g., confidence cutoffs, drift detection thresholds) hardcoded in application code.
- No version control for configuration changes.
- Configuration values that interact in undocumented ways.

**Architectural impact:** Makes the system difficult to reproduce, audit, or debug.

#### 7. Model Staleness

Models that are not retrained despite changes in the underlying data distribution. Signs include:
- No monitoring of model performance over time.
- No defined retraining schedule or triggers.
- Models deployed months or years ago without re-evaluation.
- Performance degradation attributed to "data issues" rather than model staleness.

**Architectural impact:** Gradual accuracy degradation; potential for sudden performance cliffs when distribution shifts accumulate.

### Register Entry Format

Each debt entry in the register follows a structured format:

| Field | Description |
|-------|-------------|
| ID | Unique identifier (e.g., AIDB-001) |
| Category | One of the seven categories above |
| Description | Clear description of the debt item and why it is problematic |
| Severity | Low, Medium, or High (based on blast radius and likelihood of impact) |
| Affected Components | Which models, pipelines, or services are affected |
| Mitigation Plan | Concrete steps to remediate the debt |
| Estimated Effort | T-shirt size (S/M/L/XL) or story points |
| Owner | Responsible team or individual |
| Status | Open, In Progress, Mitigated, Accepted (with rationale) |

### Illustrative Register Entries

| ID | Category | Description | Severity | Affected Components | Mitigation | Owner | Status |
|----|----------|-------------|----------|---------------------|------------|-------|--------|
| AIDB-001 | Hidden feedback loop | Route optimization predictions influence user route choices, which feed back into training data. No measurement of loop amplification. | High | Route Optimization | Implement randomized holdout (5% of users); measure prediction distribution shift with/without feedback | ML Team | Open |
| AIDB-002 | Data dependency | Weather data source changed provider without notification; new provider uses different units for wind speed | Medium | Route Optimization, Demand Prediction | Implement schema validation gate (E3 QG-001); negotiate SLA with weather provider | Data Eng | In Progress |
| AIDB-003 | Configuration debt | Confidence thresholds for 4 models hardcoded in 7 different service configurations | Low | All ML services | Centralize thresholds in config service; version-control changes | Platform Team | Open |
| AIDB-004 | Model staleness | Anomaly detection model last retrained 8 months ago; no retraining trigger configured | High | Anomaly Detection | Configure drift-based retraining trigger (E8); retrain immediately | Safety Team | Open |

### Debt Summary Dashboard

Teams should maintain a summary view of the AI debt register for management reporting:

| Category | Open | In Progress | Mitigated | Accepted |
|----------|------|-------------|-----------|----------|
| Boundary erosion | 1 | 0 | 2 | 0 |
| Entanglement | 0 | 1 | 0 | 0 |
| Hidden feedback loops | 1 | 0 | 0 | 0 |
| Data dependency | 2 | 1 | 1 | 0 |
| Pipeline debt | 0 | 0 | 1 | 1 |
| Configuration debt | 1 | 0 | 0 | 0 |
| Model staleness | 1 | 0 | 0 | 0 |
| **Total** | **6** | **2** | **4** | **1** |

### EU AI Act Relevance

E7 supports Annex IV Category 6 (Risk assessment and management) by providing a structured inventory of ML-specific risks with severity assessments and remediation plans. The debt register also contributes to Category 7 (Lifecycle changes) by documenting how technical debt evolves over the system's lifetime.

---

## E8: Operational AI View

**Extends:** None (new section; no arc42 equivalent)
**Addresses Gap:** G1
**Key Artifact:** Operational AI architecture diagram

### Purpose

Standard arc42 has no section dedicated to post-deployment operational concerns. Section 7 (Deployment View) documents how software is mapped to infrastructure, but it captures a static snapshot rather than the continuous operational processes. For traditional software, this is often sufficient: once deployed, the software runs as built until the next release.

AI systems are fundamentally different. They require continuous monitoring (for drift, performance degradation, fairness violations), periodic or triggered retraining, careful deployment strategies (to validate new model versions before full rollout), and robust rollback capabilities (because a retrained model can be worse than its predecessor). These operational concerns are architecturally significant: they shape infrastructure requirements, team structures, and organizational processes.

E8 adds a new arc42 section with four required subsections covering the operational lifecycle of AI components.

### What It Documents

#### Subsection 1: Monitoring

For each AI model or pipeline, document the monitoring configuration:

**Metrics tracked per model:**

| Metric | Type | Description | Example |
|--------|------|-------------|---------|
| Prediction quality | Performance | Primary evaluation metric on live data | MAE, RMSE, accuracy, F1 |
| Inference latency | Performance | Time to produce a prediction | P50, P95, P99 latency |
| Throughput | Performance | Predictions served per unit time | Requests per second |
| Feature drift | Data quality | Distribution shift in input features | PSI, KS-statistic per feature |
| Prediction drift | Data quality | Distribution shift in model outputs | Output distribution compared to reference |
| Data freshness | Data quality | Age of the most recent training data | Hours since last data ingestion |
| Fairness metrics | Responsible AI | Fairness indicators on live predictions | Demographic parity ratio, equalized odds |
| Confidence distribution | Model health | Distribution of prediction confidence scores | Mean confidence, percentage below threshold |

**Dashboard specifications:**

- Dashboard tool (e.g., Grafana, Datadog, custom)
- Dashboard layout and key panels
- Access controls (who can view which dashboards)
- Retention period for monitoring data

**Alerting thresholds:**

| Alert | Condition | Severity | Notification Channel | Escalation |
|-------|-----------|----------|---------------------|------------|
| Performance degradation | MAE > 5.5 min for 1 hour | P2 | Slack + PagerDuty | ML on-call |
| Severe performance degradation | MAE > 7.0 min for 30 min | P1 | PagerDuty | ML lead + SRE lead |
| Feature drift detected | PSI > 0.2 on >= 2 features | P3 | Slack | ML team |
| Inference latency spike | P95 > 100ms for 5 min | P2 | PagerDuty | SRE on-call |
| Fairness violation | Demographic parity < 0.75 | P2 | Slack + Email to compliance | ML lead + Legal |

**Drift detection configuration:**

- Detection method (e.g., Uber D3, custom PSI monitoring, Evidently AI)
- Reference distribution (training data distribution, recent production distribution, or fixed baseline)
- Detection window (how much data is needed to compute drift statistics)
- Detection frequency (how often drift checks are run)

#### Subsection 2: Retraining Policy

For each model, document the retraining policy:

**Triggers:**

| Trigger Type | Description | Example |
|-------------|-------------|---------|
| Scheduled | Regular cadence retraining | Every 30 days |
| Performance-based | Retraining when performance degrades | MAE > 5.5 min for 3 consecutive days |
| Drift-based | Retraining when data drift is detected | PSI > 0.2 on >= 2 features for 24 hours |
| Event-based | Retraining due to external events | New data source integrated, regulatory change |

**Automation level:**

- *Fully automated:* Retraining triggered, executed, validated, and deployed without human intervention. Document the validation criteria that must pass before automatic deployment.
- *Semi-automated:* Retraining triggered and executed automatically, but a human must approve deployment. Document who approves and what they review.
- *Manual:* Retraining is initiated by a human. Document the process for requesting and executing a retraining run.

**Approval workflow:**

A step-by-step description of the approval process for deploying a retrained model:

1. Retraining run completes; validation suite executes automatically.
2. If all validation checks pass, the retrained model is promoted to the staging environment.
3. ML engineer reviews validation results and staging performance (< 24 hours SLA).
4. If approved, the model enters the deployment pipeline (see Subsection 3).
5. If rejected, the issue is logged and the current production model remains active.

#### Subsection 3: Deployment Strategy

For each model, document the deployment approach:

**Deployment method:**

| Method | Description | Use Case |
|--------|-------------|----------|
| Canary deployment | New model serves a small percentage of traffic; gradually increased if metrics are acceptable | Default for production models |
| Blue-green deployment | Two identical environments; traffic is switched from old to new atomically | Models requiring zero-downtime cutover |
| Shadow deployment | New model runs alongside the production model but does not serve live traffic; outputs are logged for comparison | First deployment of a new model type; high-risk models |
| A/B testing | Two model versions serve different user segments; statistical comparison determines the winner | Evaluating model architecture changes |

**Promotion criteria:**

Quantitative criteria that must be met before a new model version is promoted from canary/shadow to full production:

| Criterion | Threshold | Measurement Period |
|-----------|-----------|-------------------|
| Primary metric (MAE) | <= production model + 5% | 24 hours of canary traffic |
| P95 latency | <= production model | 24 hours of canary traffic |
| Error rate | < 0.1% | 24 hours of canary traffic |
| Fairness metrics | >= production model | 24 hours of canary traffic |
| No P1/P2 alerts | 0 alerts | During entire canary period |

**Traffic split ratios:**

For canary deployments, document the traffic ramp schedule:

| Stage | Traffic to New Model | Duration | Criteria to Advance |
|-------|---------------------|----------|-------------------|
| Stage 1 | 1% | 6 hours | All promotion criteria met |
| Stage 2 | 10% | 12 hours | All promotion criteria met |
| Stage 3 | 50% | 24 hours | All promotion criteria met |
| Stage 4 | 100% | Permanent | N/A |

#### Subsection 4: Rollback Policy

For each model, document the rollback procedures:

**Rollback triggers:**

| Trigger | Condition | Action |
|---------|-----------|--------|
| Automatic rollback | P1 alert within first 6 hours of deployment | Immediate revert to previous model version |
| Manual rollback | P2 alert sustained > 2 hours | On-call ML engineer initiates rollback |
| Emergency rollback | Safety-critical failure detected | Any team member can trigger; no approval required |

**Model version retention:**

- How many previous model versions are retained in the deployment system.
- Retention policy: time-based (e.g., last 90 days) or count-based (e.g., last 5 versions).
- Storage location and access controls for archived model versions.

**Downstream implications:**

When a model is rolled back, document the effects on downstream consumers:

- Which services consume this model's output.
- Whether downstream services need to be notified of the rollback.
- Whether cached predictions from the rolled-back model need to be invalidated.
- Whether the rollback triggers any data pipeline reruns.

### Additional Operational Concerns

Beyond the four required subsections, E8 may include:

**A/B testing infrastructure:**

- Platform used for A/B testing (e.g., internal platform, Optimizely, LaunchDarkly).
- Statistical methodology (frequentist, Bayesian, multi-armed bandit).
- Minimum sample size and test duration requirements.
- Guardrail metrics (metrics that must not degrade, even if the primary metric improves).

**Incident response procedures:**

| Severity | Definition | Response Time | Escalation |
|----------|------------|---------------|------------|
| P1 | Safety-critical failure; data breach; system-wide outage | 15 minutes | ML lead + Engineering director + Legal |
| P2 | Significant performance degradation; fairness violation; drift affecting > 10% of predictions | 1 hour | ML on-call + ML lead |
| P3 | Minor performance degradation; drift detected but within acceptable bounds | Next business day | ML team |

### EU AI Act Relevance

E8 is the primary extension supporting Annex IV Category 10 (Post-market monitoring), which requires documentation of monitoring procedures, performance tracking, and incident response for deployed AI systems. It also supports Category 7 (Lifecycle changes) by documenting how model versions transition through deployment stages.

---

## Summary Table

| Extension | Name | Extends arc42 Section | Key Artifact | Gaps Addressed |
|-----------|------|-----------------------|--------------|----------------|
| E1 | AI Boundary Delineation | Section 3: Context and Scope | Annotated context diagram with AI boundaries | G1, G2 |
| E2 | Model Registry View | Section 5: Building Block View | Model registry table and annotated diagram | G1, G5 |
| E3 | Data Pipeline View | Section 6: Runtime View | Data pipeline diagram with quality gates | G1, G5 |
| E4 | Responsible AI Concepts | Section 8: Cross-Cutting Concepts | Responsible AI concern matrix | G1 |
| E5 | AI Decision Records (AI-ADR) | Section 9: Architecture Decisions | AI-ADR template with 7 AI-specific fields | G4 |
| E6 | AI Quality Scenarios | Section 10: Quality Requirements | AI quality scenario table | G1 |
| E7 | AI Debt Register | Section 11: Risks and Technical Debt | AI debt register with remediation plan | G1 |
| E8 | Operational AI View | New section (no arc42 equivalent) | Operational AI architecture diagram | G1 |

---

## Cross-Extension Dependencies

While each extension can be adopted independently, several extensions reference or complement each other. The following table documents the primary relationships:

| Extension | References | Nature of Dependency |
|-----------|------------|---------------------|
| E1 | E8 | E1 failure modes reference E8 monitoring and retraining triggers |
| E1 | C4-E3 | E1 boundary contracts are the authoritative source for C4-E3 diagram annotations |
| E2 | E3 | E2 training data lineage references E3 pipeline documentation |
| E2 | E5 | E2 model entries may link to AI-ADRs documenting the decision to adopt that model |
| E3 | E2 | E3 pipeline outputs feed into E2 model training; Data Cards attach at E3 source nodes |
| E4 | E6 | E4 concern matrix thresholds correspond to E6 quality scenarios |
| E4 | E8 | E4 monitoring frequencies are implemented in E8 monitoring configurations |
| E5 | E2 | E5 dataset characteristics reference E2 training data lineage |
| E5 | E6 | E5 retraining triggers correspond to E6 drift tolerance scenarios |
| E6 | E1 | E6 scenarios reference E1 boundary contracts (confidence specifications, fallback behaviors) |
| E6 | E7 | E6 quality degradation may be tracked as E7 debt entries when unresolved |
| E7 | E8 | E7 model staleness entries are mitigated by E8 retraining policies |
| E8 | E1 | E8 rollback policies implement E1 fallback strategies |
| E8 | E2 | E8 deployment strategies manage the model versions tracked in E2 |

These dependencies are informational, not blocking. A team adopting E1 without E8 simply documents fallback strategies at a higher level of abstraction, deferring the operational detail to a later adoption stage.

---

## References

- Starke, G. "arc42: The pragmatic architecture documentation template." 2023. https://arc42.org/
- Bass, L., Clements, P., and Kazman, R. *Software Architecture in Practice*, 4th ed. Addison-Wesley, 2021.
- Sculley, D. et al. "Hidden technical debt in machine learning systems." Proc. NeurIPS, 2015.
- Bogner, J., Verdecchia, R., and Gerostathopoulos, I. "Characterizing technical debt and antipatterns in AI-based systems." Proc. TechDebt, 2021.
- Mitchell, M. et al. "Model cards for model reporting." Proc. FAccT, 2019.
- Google. "Data Cards Playbook." 2022. https://sites.research.google/datacardsplaybook/
- Lu, Q. et al. "Responsible AI pattern catalogue." ACM Comput. Surv., vol. 56, no. 7, 2024.
- MADR. "Markdown Any Decision Records." https://adr.github.io/madr/
- European Parliament and Council. "Regulation (EU) 2024/1689 (AI Act)." Official J. EU, 2024.
