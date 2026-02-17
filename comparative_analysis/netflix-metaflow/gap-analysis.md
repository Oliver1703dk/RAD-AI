# Gap Analysis: Standard Frameworks vs. RAD-AI for Netflix ML Platform

## Purpose

This document provides a detailed comparison mapping each AI-specific architectural concern to what standard frameworks (arc42, C4) capture versus what RAD-AI extensions capture, with concrete Netflix examples. This gap analysis supports the RAD-AI paper's comparative claim that standard architecture documentation frameworks leave critical AI-augmented ecosystem concerns structurally undocumentable.

## Methodology

For each concern, we assess:
- **Standard coverage:** What can be documented using standard arc42 (sections 1--11) and C4 (system context, container, component, code diagrams) without any extensions?
- **RAD-AI coverage:** What additional information does the RAD-AI extension capture?
- **Netflix example:** A concrete instance from the Netflix ML platform that illustrates the gap.

## Detailed Gap Analysis

### Concern 1: Non-Deterministic Component Identification

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Models appear as generic software components ("Recommendation Service," "Model Serving") with no indication they produce probabilistic outputs. | E1 (AI Boundary Delineation) and C4-E3 (Non-Determinism Boundary) explicitly partition the architecture into deterministic and non-deterministic regions. |
| **Netflix example** | Standard arc42 section 5 lists "Model Serving" as a building block identical to "API Gateway" or "Billing Service." An architect reviewing the documentation cannot determine that recommendation model responses vary across model versions, member contexts, and retraining cycles. | RAD-AI marks recommendation models with confidence ranges (0.60--0.98), annotates confidence gates at each boundary crossing (CG-1 through CG-4), and documents fallback strategies when ML components produce low-confidence outputs or become unavailable. |
| **Impact of gap** | Architects and operators cannot reason about system behavior at the deterministic/non-deterministic boundary. Failure modes, degradation behavior, and confidence propagation are invisible. |

### Concern 2: Model Versioning and Lifecycle

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard building block views show a snapshot of the current system. There is no concept of model versioning, training lineage, or lifecycle states (development, canary, production, deprecated). | E2 (Model Registry View) treats models as first-class building blocks with version history, hyperparameters, training data lineage, performance baselines, and lifecycle state diagrams. |
| **Netflix example** | Standard arc42 section 5 documents "Recommendation Models" as a single building block. It cannot express that the recommendation system evolved from a multi-model ensemble (v10.x) to a consolidated multi-task architecture (v12.x), that each version has specific training data lineage, or that 3,000+ ML projects coexist on the platform. | RAD-AI's Model Registry View documents the version history, architecture changes per version, training data sources at each version, and integration with Metaflow's experiment tracking. The lifecycle state diagram shows how models move from development through canary deployment to production. |
| **Impact of gap** | Without model versioning in architecture documentation, teams cannot trace a production issue to a specific model version, understand why a model was retrained, or plan the impact of feature changes on downstream models. |

### Concern 3: Streaming vs. Batch Data Pipeline Distinction

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard runtime views (arc42 section 6) and C4 container diagrams show data flows as generic arrows between components. There is no mechanism to distinguish streaming from batch processing. | E3 (Data Pipeline View) and C4-E1 (`<<Data Pipeline>>` stereotype) explicitly document processing mode, freshness SLAs, scale, and quality gates per pipeline. |
| **Netflix example** | Standard arc42 shows "Flink" and "Spark" as building blocks with arrows to storage and models. An architect cannot tell that Flink's 15,000+ streaming jobs provide sub-second feature freshness for real-time serving, while Spark's nightly batch jobs prepare historical training datasets with daily freshness. The fundamentally different latency profiles, failure modes, and cost models are invisible. | RAD-AI's Data Pipeline View documents each pipeline with its processing mode (streaming vs. batch), freshness SLA (< 5s for streaming features, daily for batch features), scale (60+ PB/day for Flink, petabytes per nightly run for Spark), and quality gates. A side-by-side comparison table makes the architectural distinction explicit. |
| **Impact of gap** | The streaming/batch distinction is fundamental to understanding why recommendations respond in real-time to member behavior. Without it, architects cannot reason about feature freshness, plan infrastructure capacity, or understand degradation behavior when one processing mode fails while the other continues. |

### Concern 4: DAG Dependencies and Event-Triggered Chaining

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard runtime views show request-response or message-passing patterns. Workflow DAG topologies and event-triggered cross-workflow dependencies cannot be expressed. | E3 (Data Pipeline View) and E8 (Operational AI View) document Maestro's signal-based workflow orchestration, including `@trigger_on_finish` chains and signal metadata propagation. |
| **Netflix example** | In standard arc42, the relationship between a feature engineering workflow and a model training workflow is represented as a simple arrow ("Feature Engineering -> Model Training"). But the actual architecture is event-driven: Maestro's signal service publishes a `features_ready` signal when feature engineering completes, and model training subscribes to this signal. The signal carries metadata (feature version, row count, quality metrics) that configures the training workflow. This coordination pattern is invisible in standard documentation. | RAD-AI's Operational AI View documents the signal service explicitly, with a table of signal types (`data_landed`, `features_ready`, `model_trained`, `canary_passed`), their sources, subscribers, and metadata. A diagram shows how workflows chain through signals rather than direct invocation. |
| **Impact of gap** | Without signal-based coordination documentation, architects cannot understand how ML pipelines are triggered, what happens when an upstream pipeline fails (which downstream workflows are affected), or how to add new workflows to the orchestration graph. |

### Concern 5: Signal-Based Cross-Workflow Coordination

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | No standard arc42 or C4 concept maps to Maestro's publisher-subscriber signal pattern. Sequence diagrams show point-to-point interactions, not many-to-many signal-based coordination. | E8 (Operational AI View) and C4-E2 (Data Lineage Overlay) document signal lineage, showing which workflows publish signals and which subscribe, with metadata contracts. |
| **Netflix example** | When a batch feature pipeline completes, its output signal unblocks multiple independent workflows: recommendation model training, search model training, and a data quality audit workflow. This fan-out coordination pattern, where one event triggers many independent downstream workflows, cannot be represented in standard sequence diagrams (which show sequential point-to-point interactions). | RAD-AI documents this fan-out explicitly: the `features_ready` signal has multiple subscribers (recommendation training, search training, quality audit), each triggered independently when the signal is published. The signal service tracks lineage, enabling architects to trace which workflow triggered which downstream work. |
| **Impact of gap** | Cross-workflow coordination is the architectural mechanism that makes Netflix's ML platform self-orchestrating rather than manually scheduled. Without documenting it, architects cannot understand the platform's event-driven nature, plan for failure propagation, or reason about workflow dependencies. |

### Concern 6: Feature Store as Architectural Element

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Feature stores do not appear in standard C4 or arc42 because they have no counterpart in traditional software architecture. They are either invisible or misrepresented as a generic database. | C4-E1 (`<<Feature Store>>` stereotype) and E3 (Data Pipeline View) document the feature store as a distinct architectural element with online/offline serving modes, freshness contracts, and feature group inventory. |
| **Netflix example** | Netflix's Amber feature store mediates between data infrastructure (Flink streaming, Spark batch) and all ML models, with two fundamentally different serving interfaces: synchronous (sub-millisecond for real-time inference) and asynchronous (batch for training). In standard C4, Amber would appear as a generic "Database" container, indistinguishable from Cassandra or S3. | RAD-AI documents Amber with the `[FS]` stereotype, explicitly distinguishing online serving (sync, sub-ms) from offline serving (async, batch). Feature groups are inventoried with refresh rates, source pipelines, and consuming models. The Data Lineage Overlay traces data from sources through Amber to model consumption. |
| **Impact of gap** | The feature store is the single most important intermediary in an ML architecture. Without documenting it as a distinct element, architects cannot reason about feature freshness, online/offline consistency, feature versioning, or the impact of feature changes on downstream models. |

### Concern 7: A/B Testing and Experimentation Infrastructure

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard arc42 has no section for experimentation infrastructure. A/B testing platforms appear as generic application containers in C4, with no documentation of their integration with model serving, data collection, and model promotion. | E8 (Operational AI View) documents the complete A/B testing lifecycle: experiment design, traffic allocation, metric collection, sequential analysis, guardrail enforcement, and model promotion. E5 (AI-ADR) records experiment-driven architecture decisions. |
| **Netflix example** | ABlaze manages concurrent A/B tests across all Netflix members, with real-time metric collection via Spark Streaming + Kafka + ElasticSearch. Kayenta performs automated canary analysis. The connection between experiment results and model deployment (winning variant becomes new production model) is a first-class architectural flow. In standard arc42, none of this has a documentation home. | RAD-AI's Operational AI View documents ABlaze's experiment lifecycle with tables for each phase, concurrent experiment management, canary deployment via Spinnaker/Kayenta with step-by-step procedures, and the connection between experiment outcomes and model promotion. |
| **Impact of gap** | Experimentation is how Netflix makes model deployment decisions. Without documenting it, architects cannot understand how model updates are validated, how concurrent experiments are managed, or how guardrail metrics prevent deployment of models that degrade member experience. |

### Concern 8: ML-Specific Technical Debt

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard arc42 section 11 (Risks & Technical Debt) can track generic technical debt but has no categories for ML-specific debt (entanglement, hidden feedback loops, data dependency debt, training-serving skew). | E7 (AI Debt Register) uses Sculley et al.'s taxonomy to track ML-specific debt categories: boundary erosion, entanglement, hidden feedback loops, data dependency debt, pipeline debt, configuration debt, model staleness. |
| **Netflix example** | Netflix's recommendation models share behavioral features computed by Flink streaming jobs. When a Flink job's windowing logic changes (e.g., 5-minute to 1-minute windows), all downstream models experience an implicit distribution shift because the feature values change even though the feature schema is unchanged. This entanglement (EN-001) is invisible in standard tech debt tracking because the feature schema contract is technically unbroken. | RAD-AI's AI Debt Register documents this as feature entanglement debt, traces the dependency chain (Flink job -> Amber feature -> multiple models), and proposes mitigation (feature versioning in Amber). The debt dependency map shows how this entanglement interacts with other debt items. |
| **Impact of gap** | ML-specific technical debt (Sculley et al., 2015) accounts for a significant fraction of total system complexity in ML-augmented ecosystems. Without tracking it separately, teams address symptoms rather than root causes, and debt accumulates silently until it causes production incidents. |

### Concern 9: Drift Monitoring and Model Freshness

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard arc42 quality requirements (section 10) define traditional quality attributes (performance, availability, security). There is no concept of model freshness, drift tolerance, or prediction quality as quality attributes. | E6 (AI Quality Scenarios) defines measurable quality scenarios for model freshness, drift tolerance, recommendation diversity, serving freshness, and robustness. E8 (Operational AI View) documents drift detection methods, thresholds, and automated responses. |
| **Netflix example** | When Netflix launches a new market or significantly expands a content catalog, member behavior patterns diverge from the training distribution. A recommendation model trained on US viewing patterns produces lower-quality recommendations in a new Asian market. This concept drift must be detected (via engagement metric monitoring) and responded to (region-specific retraining with augmented data). Standard arc42 has no way to specify this requirement. | RAD-AI's quality scenario QS-04 specifies: stimulus (new market launch causing distribution divergence), detection method (regional engagement monitoring), response (multi-task transfer learning + region-specific retraining), and measurable acceptance criteria (regional quality reaches 90% of global average within 30 days). |
| **Impact of gap** | Without AI-specific quality scenarios, model degradation is detected ad hoc (often by member complaints) rather than systematically. Freshness SLAs and drift thresholds are not architectural requirements but informal team knowledge. |

### Concern 10: Responsible AI and Content Fairness

| Aspect | Standard arc42/C4 | RAD-AI |
|--------|-------------------|--------|
| **Coverage** | Standard arc42 section 8 (Cross-Cutting Concepts) covers technical concerns (security, logging, error handling) but has no structure for fairness, bias detection, explainability, or content diversity as architectural concerns. | E4 (Responsible AI Concepts) documents fairness dimensions, bias detection architecture, explainability mechanisms, human oversight, transparency, and privacy as cross-cutting architectural concerns. |
| **Netflix example** | Netflix's recommendation models must balance engagement optimization with content diversity (preventing filter bubbles), geographic equity (similar quality across markets), and creator fairness (exposure for new and niche content). These are not just ethical aspirations; they are measurable architectural quality attributes with specific mitigation mechanisms (contextual bandits for exploration, diversity constraints in post-processing, guardrail metrics in A/B tests). Standard arc42 has no place for these concerns. | RAD-AI documents fairness as a multi-dimensional architectural concern with specific metrics per dimension, bias detection at four lifecycle stages (pre-training, in-training, post-training, in-production), and explainability by design (named recommendation rows as natural-language explanations). |
| **Impact of gap** | Without documenting responsible AI concerns as architectural elements, they remain the responsibility of individual model teams rather than being system-level architectural quality attributes. This leads to inconsistent fairness practices across the 3,000+ ML projects on the platform. |

## Summary Table

| # | Concern | Standard arc42/C4 | RAD-AI Extension(s) | Gap Severity |
|---|---------|-------------------|---------------------|-------------|
| 1 | Non-deterministic component identification | Not expressible | E1 + C4-E3 | High |
| 2 | Model versioning and lifecycle | Not expressible | E2 | High |
| 3 | Streaming vs. batch pipeline distinction | Not expressible | E3 + C4-E1 | High |
| 4 | DAG dependencies and event-triggered chaining | Not expressible | E3 + E8 | High |
| 5 | Signal-based cross-workflow coordination | Not expressible | E8 + C4-E2 | High |
| 6 | Feature store as architectural element | Misrepresented as generic database | C4-E1 + E3 | High |
| 7 | A/B testing and experimentation infrastructure | No documentation home | E8 + E5 | Medium |
| 8 | ML-specific technical debt | Partially expressible (generic debt tracking) | E7 | High |
| 9 | Drift monitoring and model freshness | Not expressible | E6 + E8 | High |
| 10 | Responsible AI and content fairness | Not expressible | E4 | Medium |

**Gap severity legend:**
- **High:** The concern is structurally undocumentable in standard frameworks. There is no section, diagram type, or annotation mechanism that could capture it.
- **Medium:** The concern can be partially documented using standard frameworks but lacks the structure, vocabulary, or annotation types needed for complete documentation.

## Quantified Coverage Assessment

Across the 10 AI-specific concerns:
- **Standard arc42/C4:** 0 of 10 fully covered; 1 of 10 partially covered (ML debt via generic debt tracking); 9 of 10 structurally undocumentable.
- **RAD-AI extensions:** 10 of 10 fully covered through the 8 arc42 extensions and 3 C4 diagram types.

This assessment is consistent with the RAD-AI paper's finding that Netflix's ML platform has four concerns that are entirely undocumentable under standard frameworks, while the remaining concerns are partially addressable but lack the necessary structure and vocabulary.
