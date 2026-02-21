# Case Study: Uber Michelangelo

> **Case study type:** Production AI platform (industry)
>
> **Framework applied:** RAD-AI (arc42 extensions E1 through E8, C4 extensions C4-E1 through C4-E3)
>
> **Purpose:** Demonstrate how RAD-AI captures AI-specific architectural concerns that standard arc42/C4 frameworks leave structurally undocumented

---

## 1. System Overview

Uber Michelangelo is Uber's internal ML-as-a-service platform, covering the full machine learning lifecycle: data management, feature engineering, model training, evaluation, deployment, prediction serving, and monitoring. First described publicly in 2017, Michelangelo has since grown into one of the largest production ML platforms in industry, managing over 5,000 production models that collectively serve approximately 10 million predictions per second at peak load.

### Key Metrics

| Metric | Value | Source |
|--------|-------|--------|
| Models in production | 5,000+ | Uber Engineering Blog (2024) |
| Real-time predictions/sec (peak) | 10 to 15 million | Uber Engineering Blog (2024, 2025) |
| Active ML projects | ~400 | Uber Engineering Blog (2024) |
| Monthly training jobs | 20,000+ | Uber Engineering Blog (2024) |
| Feature Store features (Palette) | 20,000+ | Feature Store Summit (2022) |
| D3 data quality monitors | 100,000+ | D3 Blog Post (2023) |
| Tier 1 datasets monitored by D3 | 300+ | D3 Blog Post (2023) |
| GPUs managed | 5,000+ | Uber Engineering Blog (2024) |

### Key Infrastructure Components

**Feature Store (Palette).** A centralized feature management and serving platform hosting over 20,000 features organized into serving groups with tiered SLAs. Palette provides dual-topology serving: Cassandra for online (real-time, sub-millisecond) queries and Hive for offline (batch training) queries. A Scala-based Feature DSL ensures identical feature transformations in Spark (training) and Samza (serving), preventing training-serving skew. Feature changes cascade to all consuming models.

**Gallery.** Uber's model lifecycle management system, organizing the ML workflow into four explicit stages: exploration, training, evaluation, and production. Gallery enforces rule-based deployment automation; for example, a rule such as `WHEN metrics[mae] <= 5` promotes a model version from evaluation to production automatically. Gallery tracks full training lineage per model version, including dataset version, feature set version, hyperparameters, and evaluation metrics.

**D3 (Drift Detection).** An automated system for detecting data quality issues and distribution drifts across Michelangelo's data infrastructure. D3 manages over 100,000 data quality monitors across 300+ Tier 1 datasets, using a one-time profiling step (90-day baseline) followed by daily scheduled anomaly detection with Prophet-based dynamic thresholds. D3 reduced the average time-to-detect data quality issues from over 45 days to approximately 2 days.

### Major AI Components

| Component | Task | Model Architecture | Criticality |
|-----------|------|--------------------|-------------|
| DeepETA | Travel time prediction | Linear Transformer + routing engine residual | High |
| Dynamic Pricing | Surge pricing and fare estimation | Gradient boosted trees + demand forecasting | High |
| Fraud Detection | Payment fraud and account abuse | Ensemble (GBM + neural networks) | High |
| Marketplace Matching | Driver-rider matching | Deep learning + combinatorial optimization | High |
| Demand Forecasting | Spatiotemporal demand prediction | Time-series models (LSTM, DeepAR variants) | Medium |

---

## 2. Documentation Under Standard Frameworks

Documenting Uber Michelangelo using standard arc42 (sections 1 through 12) and C4 (system context, container, component, code diagrams) reveals three structural gaps where the frameworks cannot capture architecturally significant concerns.

### 2.1 Feature Store Invisibility

In a standard C4 container diagram, Palette appears as two database containers: "Feature Store (Online) [Container: Database, Cassandra]" and "Feature Store (Offline) [Container: Database, Hive]." These are visually and semantically identical to any other database in the architecture. Standard C4 has no notation to distinguish a Feature Store from a Cassandra data store, a Redis cache, or a PostgreSQL database.

The following properties, all of which are architecturally significant, are invisible:

- Feature inventory: 20,000+ features organized into serving groups with tiered SLAs
- Dual serving topology: online (Cassandra, P95 < 3ms) and offline (Hive, batch)
- Feature DSL: the Scala-based transformation layer ensuring training-serving consistency
- Cascade dependencies: a change to any feature computation propagates to every consuming model
- Freshness contracts: different feature groups have different freshness SLAs (real-time through daily)

### 2.2 Lifecycle Collapse

Gallery's four-stage model lifecycle (exploration, training, evaluation, production) with rule-based deployment automation collapses into static building blocks under standard arc42 Section 5 (Building Block View). The Building Block View presents components as fixed modules with stable interfaces. It has no concept of lifecycle stages, promotion gates, versioned artifacts, or automation rules.

At any given moment, Gallery manages 5,000+ production models, each with its own version history, training lineage, and performance baselines. The rule-based promotion (e.g., `WHEN metrics[mae] <= 5`) determines which model version serves millions of predictions per second, yet this automation has no representation in standard arc42.

Model interdependencies compound this problem. The output of the Demand Forecasting model feeds as input features to both Dynamic Pricing and Marketplace Matching. When Demand Forecasting is retrained and its output distribution shifts, downstream models receive different input signals without notification. This model-to-model dependency graph is distinct from software component dependencies and is invisible in standard arc42.

### 2.3 Monitoring Gap

D3's drift detection architecture, which tracks 100,000+ data quality monitors and reduced time-to-detect from 45 days to 2 days, has no home in the Runtime View (arc42 Section 6). The Runtime View documents request-response flows and batch processing patterns. Continuous monitoring of data distributions for drift detection is not a runtime pattern that standard arc42 recognizes.

Before D3, data quality issues took an average of 45+ days to detect. At 10 million predictions per second at peak, a 45-day detection window means trillions of potentially affected predictions. D3's monitoring architecture (one-time profiler for 90-day baseline, daily scheduled anomaly detection, Prophet-based dynamic thresholds) is a substantial system in its own right, yet standard arc42 and C4 provide no section or diagram type to document it.

D3's alerting also integrates with Michelangelo's retraining pipeline: critical drift on model input features can trigger automated retraining. This monitoring-to-retraining feedback loop is a core operational pattern with no standard arc42 representation.

---

## 3. Documentation Under RAD-AI

RAD-AI's extensions capture each of the concerns identified in Section 2, making previously invisible architectural information explicit and structured.

### 3.1 C4-E1: AI Component Stereotypes

The `<<Feature Store>>` stereotype distinguishes Palette from regular databases in any C4 diagram. Architects immediately recognize that Palette is not a generic data store; it is a specialized infrastructure component with feature inventory, freshness SLAs, dual-topology serving, and downstream consumer dependencies. Similarly, `<<ML Model>>` stereotypes distinguish inference services (DeepETA, Dynamic Pricing, Fraud Detection) from standard microservices, and `<<Monitor>>` identifies D3 as a drift detection and data quality monitoring component rather than a standard logging or APM service.

The stereotypes serve a communication function: when a data scientist and a software architect review the same C4 diagram, the stereotypes provide a shared vocabulary that bridges their different mental models of the system.

### 3.2 E2: Model Registry View

The Model Registry View captures Gallery's versioned lifecycle as a first-class architectural artifact. Each model is documented with:

- Model ID, name, and architecture (e.g., MDL-ETA: DeepETA, Linear Transformer)
- Lifecycle state (exploration, training, evaluation, production)
- Version history with training dataset hash, hyperparameters, and evaluation metrics per version
- Promotion rules (e.g., `WHEN metrics[mae] <= 5, promote to production`)
- Downstream dependencies (which other models or services consume this model's output)
- Owner and last-retrained date

Gallery's rule engine for deployment automation is documented explicitly. The rule-based promotion logic determines which model version serves production traffic, making it one of the most architecturally significant mechanisms in the platform. Under standard arc42, this logic has no documentation home.

### 3.3 E8: Operational AI View

The Operational AI View provides a dedicated section for D3's monitoring architecture, including:

- **Monitoring subsection:** Documents D3's architecture (Compute Layer, Prophet Anomaly Detection, Orchestrator), the six monitor types (null rate, percentile, statistical, categorical, foreign key, boolean), and performance metrics (95.23% detection accuracy, approximately $0.01 per dataset compute cost)
- **Retraining policy subsection:** Documents the monitoring-to-retraining loop, where critical drift on model input features triggers automated retraining through Gallery
- **Deployment strategy subsection:** Documents Uber's Model Excellence Score (MES) framework, which requires offline evaluation, shadow deployment, test coverage, and monitoring coverage before any model reaches production
- **Rollback policy subsection:** Documents version retention, rollback triggers, and downstream implications of rolling back a model version

### 3.4 C4-E3: Non-Determinism Boundary

The Non-Determinism Boundary Diagram explicitly partitions Michelangelo into deterministic and non-deterministic regions. DeepETA's architecture is the clearest example: the routing engine provides a deterministic baseline ETA, and the Linear Transformer predicts a probabilistic residual. The confidence gate at the boundary determines whether to use the ML correction or fall back to the routing engine alone.

The diagram identifies four regions:

1. **Deterministic:** API Gateway, routing engine, payment processing, logging
2. **Confidence Gate Layer:** Boundary enforcement with defined confidence thresholds per model
3. **Non-Deterministic:** DeepETA, Dynamic Pricing, Fraud Detection, Marketplace Matching, Demand Forecasting, Feature Store (non-deterministic due to streaming feature computation)
4. **Deterministic Fallback:** Degraded-mode alternatives for each ML component

Each boundary crossing is annotated with a confidence contract specifying acceptable ranges, fallback triggers, and degradation behavior.

### 3.5 E3: Data Pipeline View

The Data Pipeline View documents the complete data flow from raw events through Feature Store to model consumption. For DeepETA, the pipeline traces:

1. Raw GPS traces ingested via Kafka with Avro schema validation
2. Samza stream processing with map-matching and speed computation
3. Feature DSL transformation (ensuring training-serving consistency)
4. Anonymization (driver ID stripping, zone-level aggregation)
5. Cassandra online store write (for real-time serving)
6. DeepETA feature lookup and inference

Quality gates monitored by D3 are documented at each stage: schema conformance, null rate checks, distribution drift monitoring, and completeness verification. A failure at any stage cascades to prediction quality; the Data Pipeline View makes this cascade chain explicit.

### 3.6 E5: AI Decision Records (AI-ADR)

The AI-ADR for DeepETA's model selection decision records the full ML decision context:

- **Model alternatives considered:** XGBoost (incumbent), full Transformer, Linear Transformer (selected)
- **Dataset characteristics:** Billions of trips, global geographic scope, known biases in data-sparse regions
- **Fairness/bias trade-offs:** ETA accuracy disparities across geographic regions
- **Expected model lifetime:** Weeks to months (depends on road network changes)
- **Retraining triggers:** Road network changes, accuracy degradation beyond threshold
- **Explainability requirements:** XGBoost provides exact SHAP values; Linear Transformer provides attention analysis
- **Regulatory rationale:** Not currently classified as high-risk, but ETA accuracy affects driver earnings

### 3.7 E7: AI Debt Register

The AI Debt Register tracks ML-specific technical debt using Sculley et al.'s taxonomy:

- **Feedback loop (FL-001):** Surge pricing suppresses demand, which feeds into demand forecasting training data, reducing predicted demand, which reduces surge. This oscillation is an inherent ML system dynamics problem.
- **Feature entanglement (EN-001):** Dynamic Pricing consumes Demand Forecasting's output as a feature. When Demand Forecasting is retrained and its output distribution shifts, Dynamic Pricing receives different input signals without notification.
- **Dead feature accumulation (PD-001):** With 20,000+ features, an unknown fraction are no longer consumed by any active model but still consume compute and storage resources.

These debt items are tracked with severity, affected components, remediation effort, owner, and status.

---

## 4. Concern Coverage

The following table maps ten AI-specific concerns to their coverage under standard arc42/C4 versus RAD-AI, with the corresponding RAD-AI extension(s) that address each concern.

| # | AI-Specific Concern | Standard arc42/C4 | RAD-AI | Extension(s) |
|---|---------------------|-------------------|--------|--------------|
| 1 | AI boundary delineation | Not captured | **Fully captured** | E1, C4-E3 |
| 2 | Feature Store as first-class element | Not captured | **Fully captured** | C4-E1 |
| 3 | Model lifecycle management | Not captured | **Fully captured** | E2 |
| 4 | Data drift monitoring | Not captured | **Fully captured** | E8 |
| 5 | Data lineage and provenance | Partial (generic arrows) | **Fully captured** | C4-E2, E3 |
| 6 | Non-determinism boundaries | Not captured | **Fully captured** | C4-E3 |
| 7 | ML-specific technical debt | Partial (generic debt register) | **Fully captured** | E7 |
| 8 | Responsible AI concerns | Not captured | **Fully captured** | E4 |
| 9 | AI-specific quality requirements | Not captured | **Fully captured** | E6 |
| 10 | AI decision rationale | Not captured | **Fully captured** | E5 |

**Summary:**

- **Standard arc42/C4:** 0 of 10 fully captured; 2 of 10 partially captured (data lineage, ML debt); 8 of 10 structurally undocumentable
- **RAD-AI:** 8 of 10 fully captured; 2 of 10 improved from partial to full coverage

The two partial-to-full improvements (concerns 5 and 7) reflect cases where standard frameworks can capture some surface-level information (generic data flow arrows, generic debt items) but lack the vocabulary and structure to capture AI-specific properties (quality gates in data lineage, ML-specific debt categories).

---

## 5. Key Insight

Despite Uber's heavy investment in model management (Gallery), drift detection (D3), and deployment governance (Model Excellence Score), standard arc42/C4 provides no place to record any of it. The Feature Store, which mediates between all data infrastructure and all ML models, appears as a generic database. The model lifecycle, which determines which version serves millions of predictions per second, collapses into a static building block. The drift detection system, which manages 100,000+ monitors and reduced detection latency by a factor of 20, has no documentation home.

These gaps are structural, not due to lack of information. Uber's engineering teams have extensively documented Michelangelo's capabilities in blog posts, conference papers, and internal documentation. The problem is that the frameworks they would use for architecture documentation (arc42, C4) have no sections, diagram types, or annotation mechanisms that can accommodate this information. RAD-AI closes this gap through targeted, backward-compatible extensions that provide structured homes for each concern.

---

## References

1. Hermann, J. and Del Balso, M. "Meet Michelangelo: Uber's Machine Learning Platform." Uber Engineering Blog, September 2017.
2. Sun, C. et al. "Gallery: A Machine Learning Model Management System at Uber." In Proceedings of the 23rd International Conference on Extending Database Technology (EDBT), 2020.
3. "D3: An Automated System to Detect Data Drifts." Uber Engineering Blog, February 2023.
4. "Raising the Bar on ML Model Deployment Safety." Uber Engineering Blog, 2025.
5. "From Predictive to Generative: How Michelangelo Accelerates Uber's AI Journey." Uber Engineering Blog, 2024.
6. "DeepETA: How Uber Predicts Arrival Times Using Deep Learning." Uber Engineering Blog, 2022.
7. "Palette Meta Store Journey." Uber Engineering Blog, 2022.
8. Sculley, D. et al. "Hidden Technical Debt in Machine Learning Systems." In Advances in Neural Information Processing Systems (NeurIPS), 2015.
