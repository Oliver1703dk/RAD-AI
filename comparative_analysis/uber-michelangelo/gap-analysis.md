# Gap Analysis: Standard arc42/C4 vs. RAD-AI for Uber Michelangelo

## Purpose

This document provides a detailed comparison mapping each AI-specific concern to what standard arc42/C4 frameworks capture versus what RAD-AI extensions capture, with concrete Michelangelo examples demonstrating each gap. The analysis follows the 10 AI-specific concerns identified in the RAD-AI paper.

## Summary Table

| # | AI-Specific Concern | Standard arc42/C4 Coverage | RAD-AI Coverage | Michelangelo Impact |
|---|--------------------|-----------------------------|-----------------|---------------------|
| 1 | AI boundary delineation | None -- no distinction between deterministic and non-deterministic components | E1: Explicit boundary with confidence gates, fallback strategies, degradation profiles | DeepETA's hybrid routing-engine + transformer architecture requires explicit boundary documentation |
| 2 | Feature Store as first-class element | Generic "Database" container | C4-E1 `<<Feature Store>>` stereotype with feature count, freshness SLA, dual topology | Palette's 20,000+ features with Cassandra/Hive dual serving are invisible as generic databases |
| 3 | Model lifecycle management | Static building blocks with no lifecycle | E2: Model Registry View with Gallery's 4-stage lifecycle and rule-based automation | Gallery's `WHEN metrics[mae] <= 5` promotion rules have no standard arc42 home |
| 4 | Data drift monitoring | Not represented in any standard section | E8: Operational AI View documents D3 architecture, monitor types, and alert thresholds | D3's 100,000+ monitors and 20x improvement in time-to-detect are architecturally invisible |
| 5 | Data lineage and provenance | Generic data flow arrows | C4-E2: Data Lineage Overlay with quality gates, freshness constraints, privacy flow | GPS traces -> Samza -> Palette -> DeepETA requires explicit quality gates at each stage |
| 6 | Non-determinism boundaries | Not represented | C4-E3: Non-Determinism Boundary Diagram with confidence contracts and fallback strategies | Deterministic routing engine vs. probabilistic DeepETA residual requires explicit partitioning |
| 7 | ML-specific technical debt | Generic risk/debt register | E7: AI Debt Register with Sculley et al. categories (entanglement, feedback loops, data dependencies) | Pricing-demand feedback loops and Feature Store coupling are ML-specific debt patterns |
| 8 | Responsible AI concerns | No dedicated section | E4: Responsible AI Concepts (fairness, bias detection, explainability, human oversight) | Pricing fairness across neighborhoods and fraud detection explainability for user appeals |
| 9 | AI-specific quality requirements | Standard quality scenarios (latency, availability) | E6: AI Quality Scenarios (model freshness SLAs, drift thresholds, fairness constraints, deployment safety) | DeepETA must retrain on road network changes; D3 drift detection triggers are quality requirements |
| 10 | AI decision rationale | Standard ADR (no ML-specific fields) | E5: AI-ADR with 7 AI fields (dataset, fairness, model lifetime, retraining, explainability, regulatory) | DeepETA architecture decision requires documenting dataset, latency-accuracy trade-off, and retraining strategy |

## Detailed Analysis

### 1. AI Boundary Delineation

**Standard arc42/C4:** The Context and Scope (S3) section and C4 System Context diagram show Michelangelo as a single "Software System" box with generic data flows. There is no mechanism to indicate that some outputs are deterministic (routing engine segment-sum ETA: always the same for the same input) and some are probabilistic (DeepETA residual: varies by model version, may have low confidence on novel routes).

**RAD-AI (E1):** The AI Boundary Delineation section explicitly partitions Michelangelo into deterministic (API gateway, routing engine, payment processing, logging) and non-deterministic (DeepETA, Dynamic Pricing, Fraud Detection, Marketplace Matching, Demand Forecasting, Feature Store) zones. Each ML component is annotated with its confidence range, and each boundary crossing has a confidence gate with defined fallback behavior.

**Michelangelo Example:**
- DeepETA's architecture is inherently hybrid: the routing engine provides a deterministic baseline, and the Linear Transformer predicts a probabilistic residual. The boundary between these two components is architecturally critical (if the ML residual is unreliable, the system should use the routing engine alone), but this boundary is invisible in standard documentation.
- Palette (Feature Store) straddles the boundary: the Cassandra infrastructure is deterministic, but feature values are computed from streaming data and statistical aggregations, making them non-deterministic in the broader sense. Standard arc42 has no way to express this nuance.

---

### 2. Feature Store as First-Class Element

**Standard arc42/C4:** In the standard C4 container diagram, Palette appears as two database containers: "Feature Store (Online) [Container: Database - Cassandra]" and "Feature Store (Offline) [Container: Database - Hive]." These are visually and semantically identical to any other database in the architecture. There is no indication of feature count (20,000+), freshness SLAs (varying from real-time to daily), feature versioning, or the Feature DSL that ensures train-serve consistency.

**RAD-AI (C4-E1):** The `<<Feature Store>>` stereotype elevates Palette to a distinct architectural element with documented feature inventory, serving topology, freshness contracts per feature group, and downstream consumer mapping.

**Michelangelo Example:**
- Palette hosts 20,000+ features organized into serving groups with different SLAs (ETA Critical: P95 < 3ms, 99.99% availability; Batch Analytics: no latency SLA). This tiered architecture is invisible in standard C4.
- The Feature DSL (Scala-based) that ensures identical transformations in Spark (training) and Samza (serving) is an architectural invariant critical for preventing training-serving skew. Standard arc42 has no place to document this invariant.
- Feature changes cascade to all consuming models. When a feature computation changes (e.g., aggregation window from 15-min to 5-min), the impact propagates to every model consuming that feature. Standard documentation shows data flows but not semantic dependencies.

---

### 3. Model Lifecycle Management

**Standard arc42/C4:** The Building Block View (S5) presents Michelangelo's components as static building blocks: Training Service, Evaluation Service, Model Repository, Prediction Service. The lifecycle is implied by the flow between components but not explicitly documented. Gallery's 4-stage lifecycle (exploration, training, evaluation, production) with automated rule-based promotion collapses into a generic linear flow.

**RAD-AI (E2):** The Model Registry View treats models as first-class building blocks with their own lifecycle, versioning, hyperparameter history, training lineage, and performance baselines. Gallery's lifecycle stages, quality gates, and automation rules are explicitly documented.

**Michelangelo Example:**
- Gallery manages 5,000+ production models through 4 lifecycle stages with automated promotion rules (e.g., `WHEN metrics[mae] <= 5, promote to production`). This rule-based automation is architecturally significant (it determines which model version serves millions of predictions per second) but has no representation in standard arc42 S5.
- Model versioning with full training lineage (dataset version, feature set version, hyperparameters, evaluation metrics) is critical for reproducibility and debugging. Standard arc42 tracks software component versions through code commits; ML model versions evolve through data and hyperparameter changes, which is a fundamentally different versioning paradigm.
- Model interdependencies (MDL-DEMAND output feeds into MDL-PRICE and MDL-MATCH) create a dependency graph that is distinct from software component dependencies. Changes to MDL-DEMAND's output distribution cascade to downstream models. This model-to-model dependency graph is invisible in standard arc42.

---

### 4. Data Drift Monitoring

**Standard arc42/C4:** The Runtime View (S6) documents request-response flows and batch processing patterns. Continuous monitoring of data distributions for drift detection is not a runtime pattern that standard arc42 recognizes. D3's architecture (Compute Layer, Prophet Anomaly Detection, Orchestrator) and its 100,000+ monitors have no documentation home.

**RAD-AI (E8):** The Operational AI View documents D3's architecture, monitor types (null rate, percentile, statistical, categorical, foreign key, boolean), performance metrics (95.23% detection accuracy, 20x improvement in time-to-detect, ~$0.01 per dataset compute cost), and integration with the retraining pipeline.

**Michelangelo Example:**
- Before D3, data quality issues took an average of 45+ days to detect. D3 reduced this to ~2 days. At 10 million predictions per second at peak, a 45-day detection window means trillions of potentially affected predictions. This operational improvement is architecturally significant but invisible in standard documentation.
- D3 monitors 100,000+ data quality indicators across 300+ Tier 1 datasets. The monitoring architecture (one-time profiler for 90-day baseline, daily scheduled anomaly detection, Prophet-based dynamic thresholds) is a substantial system in its own right, with its own infrastructure, compute costs, and operational procedures.
- D3's alerting integrates with Michelangelo's retraining pipeline: critical drift on model input features can trigger automated retraining. This monitoring-to-retraining loop is a core operational pattern with no standard arc42 representation.

---

### 5. Data Lineage and Provenance

**Standard arc42/C4:** Data flows in standard C4 container diagrams are generic arrows with labels like "raw data" or "predictions." There is no mechanism to document the transformation chain from raw GPS traces through Kafka, Samza stream processing, Feature DSL application, Cassandra storage, to DeepETA inference. Quality gates at each stage (schema validation, null rate checks, distribution drift monitoring) are not represented.

**RAD-AI (C4-E2):** The Data Lineage Overlay traces data from source through every transformation stage to model consumption, annotated with freshness requirements, quality gates (D3-integrated), privacy classifications, and feedback loops.

**Michelangelo Example:**
- The path from raw GPS traces to a DeepETA prediction involves: (1) Kafka ingestion with Avro schema validation, (2) Samza stream processing with map-matching and speed computation, (3) Feature DSL transformation, (4) Anonymization (driver ID stripping, zone-level aggregation), (5) Cassandra online store write, (6) DeepETA feature lookup and inference. Each stage has specific quality gates monitored by D3. A failure at any stage cascades to prediction quality.
- Privacy-by-design is embedded in the data lineage: GPS traces are anonymized to zone-level aggregates before entering the Feature Store; transaction data is PCI-DSS tokenized at Kafka ingestion. These privacy transformations are architecturally significant but invisible in standard data flow diagrams.
- The feedback loop from pricing decisions to demand data to pricing model retraining is a data lineage concern that spans the entire pipeline. Standard data flow arrows cannot represent the circular causality.

---

### 6. Non-Determinism Boundaries

**Standard arc42/C4:** Standard C4 diagrams do not partition the architecture into deterministic and non-deterministic regions. All containers are treated uniformly: the API Gateway and the DeepETA model are both just "containers" with different technologies.

**RAD-AI (C4-E3):** The Non-Determinism Boundary Diagram explicitly separates the architecture into four regions: Deterministic (infrastructure), Confidence Gate Layer (boundary enforcement), Non-Deterministic (ML components), and Deterministic Fallback (degraded-mode alternatives). Each boundary crossing has a defined confidence contract.

**Michelangelo Example:**
- DeepETA's residual prediction architecture is the clearest example: the routing engine (deterministic) provides a baseline ETA, and the Linear Transformer (non-deterministic) predicts a correction. The confidence gate determines whether to use the ML correction or fall back to the routing engine alone. This boundary is architecturally critical but invisible in standard C4.
- The fraud detection pipeline has a three-way confidence gate: auto-approve (score < 0.4), human review (score 0.4-0.6), auto-block (score >= 0.6). This creates three distinct execution paths depending on model confidence, a pattern that standard sequence diagrams can show but without the architectural significance of the non-determinism boundary.
- Feature Store degradation triggers cascade fallback across all online-serving models simultaneously. The cascade behavior (Cassandra outage -> all models fall back) is a system-level degradation pattern that requires explicit architectural documentation.

---

### 7. ML-Specific Technical Debt

**Standard arc42/C4:** arc42 S11 (Risks & Technical Debt) provides a generic register for tracking risks and debt. Standard categories (code quality, testing gaps, documentation debt, infrastructure upgrade needs) do not capture ML-specific debt patterns identified by Sculley et al.: entanglement, hidden feedback loops, data dependency debt, and pipeline debt.

**RAD-AI (E7):** The AI Debt Register uses Sculley et al.'s taxonomy to track debt categories specific to ML systems, with concrete items, severity assessments, affected components, and mitigation plans.

**Michelangelo Example:**
- **Pricing-demand feedback loop (FL-001):** Surge pricing suppresses demand, which feeds into demand forecasting training data, which reduces predicted demand, which reduces surge, creating an oscillation. This is not a code quality issue or a testing gap; it is an inherent ML system dynamics problem that requires causal modeling and price experimentation to mitigate.
- **Feature entanglement (EN-001):** MDL-PRICE consumes MDL-DEMAND's output as a feature. When MDL-DEMAND is retrained and its output distribution shifts, MDL-PRICE receives different input signals without notification. This cross-model entanglement through shared features is a debt pattern unique to ML systems.
- **Dead feature accumulation (PD-001):** With 20,000+ features, an unknown fraction are no longer consumed by any active model but still consume compute. At Uber's scale, even 10% dead features represent significant wasted resources.

---

### 8. Responsible AI Concerns

**Standard arc42/C4:** arc42 S8 (Cross-Cutting Concepts) covers generic quality attributes like logging, error handling, and security. There is no dedicated section for fairness, bias detection, explainability, or human oversight as architectural concerns.

**RAD-AI (E4):** Responsible AI Concepts documents fairness metrics per model, bias detection architecture (pre-training, in-training, post-training, in-production), explainability requirements and methods, human oversight mechanisms, transparency disclosures, and privacy-by-design.

**Michelangelo Example:**
- **Pricing fairness:** Dynamic Pricing may systematically disadvantage riders in lower-income neighborhoods where supply is scarcer. This fairness concern requires architectural support: zone-stratified evaluation, fairness monitoring dashboards, and price experimentation infrastructure. None of this has a home in standard arc42 S8.
- **Fraud detection explainability:** When a user's transaction is blocked, they need an explanation. The GBM component provides SHAP-based risk factor decomposition; the neural network component provides contribution scores. This per-decision explainability requirement is an architectural concern that drives component design (SHAP computation must be co-located with inference for latency) but has no standard arc42 representation.
- **Human oversight for fraud:** The Fraud Review Queue (HITL) is an architecturally significant component where human analysts make final decisions on ambiguous cases. These human decisions feed back as training labels. This human-in-the-loop pattern is a responsible AI architectural concern.

---

### 9. AI-Specific Quality Requirements

**Standard arc42/C4:** arc42 S10 (Quality Requirements) defines quality scenarios for traditional attributes: response time, throughput, availability, maintainability. These are necessary but insufficient for ML systems, which have additional quality dimensions: model freshness, drift tolerance, fairness constraints, and deployment safety.

**RAD-AI (E6):** AI Quality Scenarios define measurable quality requirements for AI-specific attributes with concrete stimulus-response-measure formulations.

**Michelangelo Example:**
- **Model freshness after road network change (QS-01):** When a new road opens, DeepETA must retrain within 72 hours. This quality requirement is specific to ML systems (traditional software does not need to "retrain" after environment changes). Standard arc42 quality scenarios have no mechanism to express retraining SLAs.
- **D3 drift detection SLA (QS-03):** When D3 detects feature drift (PSI > 0.2), the affected models must be evaluated within 1 business day and retrained within 3 business days. This is a quality requirement on the monitoring-to-retraining loop, a concept that does not exist in standard quality scenarios.
- **Deployment safety via MES (QS-08):** Uber's Model Excellence Score framework requires offline evaluation, shadow deployment, test coverage, and monitoring coverage before any model reaches production. This deployment safety quality requirement is ML-specific and has no equivalent in standard arc42.

---

### 10. AI Decision Rationale

**Standard arc42/C4:** arc42 S9 (Architecture Decisions) uses standard ADR format with context, decision, and consequences. Standard ADR templates lack fields for dataset characteristics, fairness trade-offs, model lifetime, retraining triggers, explainability methods, and regulatory compliance.

**RAD-AI (E5):** AI-ADR extends MADR with 7 AI-specific fields, ensuring that model selection decisions document the full ML decision context.

**Michelangelo Example:**
- The decision to replace XGBoost with a Linear Transformer for ETA prediction involved trade-offs specific to ML systems: (1) accuracy-latency trade-off (full Transformer too slow, linear approximation fits budget), (2) feature engineering burden (Transformer learns interactions vs. XGBoost requires manual engineering), (3) explainability trade-off (XGBoost has exact SHAP vs. Transformer attention analysis), (4) residual prediction design (preserving routing engine as deterministic fallback), (5) training infrastructure change (CPU to GPU cluster). A standard ADR captures the decision but not the ML-specific context.
- The AI-ADR captures dataset characteristics (billions of trips, geographic scope, known biases in data-sparse regions), fairness considerations (ETA accuracy disparities across regions), model lifetime (weeks to months), retraining triggers (road network changes, accuracy degradation), and the team's expertise trade-off (extensive GBM experience vs. limited Transformer experience). These seven additional dimensions are essential for understanding and maintaining the decision over time.

## Cross-Cutting Observations

### Scale Amplifies Gaps

At Michelangelo's scale (5,000+ models, 20,000+ features, 10-15M predictions/sec, 100,000+ D3 monitors), every documentation gap is amplified:
- An undocumented model-to-model dependency affects millions of predictions.
- A dead feature in the Feature Store wastes compute across thousands of serving requests per second.
- A missing drift threshold means weeks of undetected degradation at billions-of-predictions scale.

Standard arc42/C4 was designed for systems where the architecture is stable and deterministic. Michelangelo's architecture is continuously evolving through retraining, feature engineering, and data distribution changes. RAD-AI's extensions address this fundamental mismatch.

### Ecosystem Implications

Michelangelo is not a standalone system; it is an ecosystem component serving hundreds of product teams. The documentation gaps identified above have ecosystem-level consequences:
- **Cascading drift:** When a feature in Palette drifts, all consuming models across all product teams are affected. Without Feature Store documentation (C4-E1) and drift monitoring documentation (E8), product teams cannot understand their risk exposure.
- **Differentiated compliance:** Different product teams may face different regulatory requirements (pricing regulation varies by jurisdiction; fraud detection has PCI-DSS requirements). Without Responsible AI documentation (E4) at the platform level, compliance is fragmented.
- **Federated governance:** With 400+ active ML projects, governance must be federated. Model Excellence Score (MES) provides a platform-level governance framework, but it requires documentation infrastructure (AI Quality Scenarios, Operational AI View) that standard arc42 does not provide.
