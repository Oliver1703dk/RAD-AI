# Case Study: Netflix Metaflow/Maestro

> **Case study type:** Production AI platform (industry)
>
> **Framework applied:** RAD-AI (arc42 extensions E1 through E8, C4 extensions C4-E1 through C4-E3)
>
> **Purpose:** Demonstrate that RAD-AI's coverage improvements are not domain-specific but reflect structural properties of the framework extensions, by comparing the gap pattern here with the Uber Michelangelo case study

---

## 1. System Overview

Netflix operates one of the world's largest ML platforms, supporting over 3,000 AI/ML projects that power content recommendation, personalization, content quality assessment, search, and operational optimization across 200+ million subscribers. The platform processes petabytes of data daily and manages tens of petabytes of models and artifacts.

### Core Infrastructure Components

**Metaflow.** A Python-native framework for ML workflows, originally developed at Netflix and released as open source. Metaflow represents ML pipelines as DAGs (directed acyclic graphs) defined in Python code, with built-in support for experiment tracking, artifact management, and versioning. Metaflow provides the primary interface through which Netflix's data scientists define, run, and iterate on ML experiments.

**Maestro.** Netflix's workflow orchestrator, responsible for production scheduling and event-driven coordination of ML pipelines at scale. Maestro introduces a signal-based cross-workflow coordination mechanism: when one workflow step completes, it publishes a signal (e.g., `features_ready`) that can trigger dependent steps in entirely different workflows. This publisher-subscriber orchestration pattern enables Netflix's ML platform to be self-orchestrating rather than manually scheduled. Maestro manages hundreds of thousands of workflows executing millions of jobs daily.

**Apache Flink (Streaming).** Netflix operates 15,000+ Flink streaming jobs processing over 60 petabytes of data per day. Flink provides sub-second feature freshness for real-time model serving, computing streaming features from member behavior events as they occur.

**ABlaze.** Netflix's A/B testing platform, managing concurrent experiments across all Netflix members. ABlaze handles experiment allocation, traffic splitting, real-time metric collection (via Spark Streaming, Kafka, and ElasticSearch), and sequential statistical analysis. Combined with Kayenta (canary analysis, co-developed with Google) and Spinnaker (continuous delivery), ABlaze forms the experimentation infrastructure through which Netflix validates every model update before production deployment.

### AI Components

| Component | Model Type | Task | Serving Mode |
|-----------|-----------|------|-------------|
| Recommendation Models | Ensemble (collaborative filtering + deep learning + graph-based) | Content ranking, homepage personalization | Real-time inference + batch pre-computation |
| Personalization Models | Multi-task neural networks | UI layout, artwork selection, notifications | Real-time inference |
| Content Quality Models | Deep neural networks (SemanticGNN) | Video/audio quality prediction, encoding | Batch processing |
| Search Models | Transformer-based retrieval | Query understanding, search ranking | Real-time inference |
| Experimentation Models | Causal inference, contextual bandits | A/B test analysis, treatment effect estimation | Batch analysis |

---

## 2. Documentation Under Standard Frameworks

Documenting Netflix's ML platform using standard arc42 (sections 1 through 12) and C4 (system context, container, component, code diagrams) reveals four structural gaps.

### 2.1 DAG Invisibility

Metaflow's Python-native DAGs with event-triggered chaining are invisible in the Runtime View (arc42 Section 6). Standard arc42 shows process interactions as request-response or message-passing patterns. ML workflow orchestration, where a data ingestion step triggers a feature engineering step that triggers a model training step that triggers an evaluation step, is a fundamentally different runtime pattern. The DAG topology, step dependencies, artifact passing between steps, and event-triggered chaining (via Maestro's `@trigger_on_finish` decorator) cannot be expressed in standard runtime diagrams.

In standard arc42, the relationship between a feature engineering workflow and a model training workflow is represented as a simple arrow ("Feature Engineering -> Model Training"). But the actual architecture is event-driven: Maestro's signal service publishes a `features_ready` signal when feature engineering completes, and model training subscribes to this signal. The signal carries metadata (feature version, row count, quality metrics) that configures the training workflow. This coordination pattern is invisible in standard documentation.

### 2.2 Signal Coordination Gap

Maestro's signal-based cross-workflow coordination, where one step's output unblocks dependent steps across different workflows, has no documentation counterpart in arc42 or C4. When a batch feature pipeline completes, its output signal unblocks multiple independent workflows: recommendation model training, search model training, and a data quality audit workflow. This fan-out coordination pattern, where one event triggers many independent downstream workflows, cannot be represented in standard sequence diagrams, which show sequential point-to-point interactions.

The signal service tracks lineage: which workflow published a signal, which workflows subscribed, and what metadata was carried. This lineage is architecturally significant for understanding failure propagation (if an upstream pipeline fails, which downstream workflows are affected?) and for planning changes (if a feature pipeline is modified, which training workflows need to be updated?). Standard arc42 and C4 have no mechanism to capture signal-based coordination.

### 2.3 Processing Distinction Lost

The distinction between streaming (15,000+ Flink jobs, sub-second freshness) and batch processing (nightly Spark jobs, daily freshness) is lost in standard C4 container diagrams. Both Flink and Spark appear as generic processing containers with arrows to storage and models. An architect reviewing the standard documentation cannot determine that Flink's streaming jobs provide the sub-second feature freshness required for real-time recommendation serving, while Spark's nightly batch jobs prepare historical training datasets with daily freshness.

The fundamentally different latency profiles, failure modes, and cost models of streaming versus batch processing are invisible. This distinction matters for capacity planning, degradation behavior (what happens when Flink fails but Spark continues?), and feature freshness reasoning.

### 2.4 A/B Testing Undocumented

The ABlaze A/B testing platform, Kayenta canary analysis, and the connection between experiment results and model deployment have no documentation home in standard arc42. Section 8 (Cross-Cutting Concepts) covers generic quality attributes like logging and error handling, not experimentation infrastructure. Section 6 (Runtime View) covers request-response patterns, not experiment allocation and traffic splitting.

Experimentation is the mechanism through which Netflix makes model deployment decisions. Without documenting it, architects cannot understand how model updates are validated, how concurrent experiments are managed, or how guardrail metrics prevent deployment of models that degrade member experience.

---

## 3. Documentation Under RAD-AI

RAD-AI's extensions provide structured homes for each of the concerns identified in Section 2.

### 3.1 C4-E1: AI Component Stereotypes

The `<<ML Model>>` stereotype distinguishes recommendation, personalization, search, and content quality models from standard software services. The `<<Data Pipeline>>` stereotype distinguishes Metaflow/Flink ML workflows from standard ETL or message queues, with annotations for processing mode (streaming versus batch). The `<<Feature Store>>` stereotype identifies Netflix's Amber feature store as architecturally distinct from a generic database.

These stereotypes make the streaming/batch distinction visible in C4 diagrams. A container annotated `<<Data Pipeline>> [Streaming, Flink]` is immediately distinguishable from `<<Data Pipeline>> [Batch, Spark]`, enabling architects to reason about freshness profiles, failure modes, and capacity requirements.

### 3.2 C4-E2: Data Lineage Overlay

The Data Lineage Overlay traces data from ingestion through Flink streaming and Spark batch processing to model consumption. The overlay reveals dependencies that are invisible in standard C4: which models depend on which streaming features, which batch features are shared across model training pipelines, and where data quality gates are enforced.

For Netflix, the overlay documents how member behavior events flow through Kafka, are processed by Flink streaming jobs into real-time features stored in Amber, and are simultaneously aggregated by nightly Spark jobs into training datasets. The two paths converge at model consumption: real-time features for inference, batch features for training. This dual-path architecture is the foundation of Netflix's ML platform, yet it is entirely invisible in standard C4 container diagrams.

### 3.3 E5: AI Decision Records (AI-ADR)

The AI-ADR template records experiment-driven architecture decisions with the seven AI-specific fields: model alternatives considered, dataset characteristics, fairness/bias trade-offs, expected model lifetime, retraining triggers, explainability requirements, and regulatory rationale.

For Netflix, this is particularly important because model deployment decisions are experiment-driven. An AI-ADR for the recommendation model architecture records not only the decision to consolidate from a multi-model ensemble (v10.x) to a multi-task architecture (v12.x), but also the A/B test configurations that validated this decision, the metrics compared (engagement, diversity, member satisfaction), and the guardrail metrics that constrained the decision space.

### 3.4 E8: Operational AI View

The Operational AI View documents four subsections that capture Netflix's operational ML infrastructure:

- **Monitoring:** Documents drift detection methods, model performance dashboards, and alerting thresholds for each model category (recommendation, personalization, search, content quality)
- **A/B testing infrastructure:** Documents ABlaze's experiment lifecycle (design, allocation, execution, analysis, decision), concurrent experiment management, canary deployment via Spinnaker/Kayenta with step-by-step procedures, and the connection between experiment outcomes and model promotion
- **Deployment strategy:** Documents canary deployment procedures, shadow mode testing, and the progressive rollout strategy from 1% to 100% of traffic
- **Retraining policy:** Documents Maestro's signal-based orchestration of retraining workflows, including `@trigger_on_finish` chains, signal metadata propagation, and the fan-out coordination pattern

The signal-based coordination is documented explicitly within the Operational AI View. A table of signal types (`data_landed`, `features_ready`, `model_trained`, `canary_passed`) lists their sources, subscribers, and metadata contracts. A diagram shows how workflows chain through signals rather than direct invocation, making the self-orchestrating nature of Netflix's ML platform visible.

### 3.5 E3: Data Pipeline View

The Data Pipeline View captures Metaflow DAG structure with quality gates at each stage. Each pipeline stage is documented with its processing mode (streaming or batch), freshness SLA, scale, and quality gate (schema conformance, distribution tests, completeness checks, with thresholds and failure actions).

The view makes the streaming/batch distinction architecturally explicit. A side-by-side comparison documents the two pipeline families:

| Property | Streaming (Flink) | Batch (Spark) |
|----------|-------------------|---------------|
| Jobs | 15,000+ | Nightly runs |
| Freshness | Sub-second | Daily |
| Scale | 60+ PB/day | Petabytes per run |
| Failure mode | Feature staleness | Training data staleness |
| Quality gates | D3-integrated, continuous | Pre-training validation |

### 3.6 E2: Model Registry View

The Model Registry View documents model versioning across Netflix's 3,000+ ML projects. Each model is registered with its version history, architecture, training data lineage, evaluation metrics, lifecycle state, and experiment history. The registry integrates with Metaflow's artifact tracking, providing a single architectural view of which models are in production, which are in canary testing, and how each version relates to its training data and experiment context.

The registry captures a critical architectural transition: Netflix's recommendation system evolved from a multi-model ensemble (v10.x, with separate collaborative filtering, content-based, and contextual models) to a consolidated multi-task architecture (v12.x, where a single model handles multiple recommendation objectives). This architectural evolution, including the rationale, the A/B test evidence, and the performance trade-offs, is documented in the model version history. Under standard arc42, this evolution would be invisible; the Building Block View shows only the current state.

### 3.7 C4-E3: Non-Determinism Boundary

The Non-Determinism Boundary Diagram partitions Netflix's architecture into deterministic and non-deterministic regions:

- **Deterministic region:** CDN infrastructure, API Gateway, member authentication, content metadata service, billing and subscription management, device registration
- **Confidence Gate Layer:** Boundary enforcement at each point where ML inference results enter deterministic serving paths (e.g., the recommendation API endpoint applies confidence thresholds before returning personalized results)
- **Non-deterministic region:** Recommendation models, personalization models, search models, content quality models, experimentation models, Amber feature store (non-deterministic due to real-time streaming feature computation)
- **Deterministic Fallback:** Popularity-based content ranking (fallback for recommendation), default UI layout (fallback for personalization), keyword-based search (fallback for ML-powered search)

Each ML component has a defined degradation profile. When recommendation model confidence drops below threshold, the system falls back to popularity-based ranking; the member experience degrades gracefully rather than failing. This degradation behavior is architecturally significant (it affects how the CDN caches content, how the API gateway routes requests, and how the client application renders results) but is invisible in standard C4 diagrams.

### 3.8 E4: Responsible AI Concepts

The Responsible AI Concepts section documents fairness, explainability, and content diversity as cross-cutting architectural concerns:

- **Content diversity:** Recommendation models must balance engagement optimization with content diversity (preventing filter bubbles). Contextual bandits provide exploration; diversity constraints in post-processing ensure exposure for new and niche content. These are not ethical aspirations but measurable architectural quality attributes with specific implementation mechanisms.
- **Geographic equity:** Recommendation quality should be comparable across all markets (200+ countries). Models trained primarily on US viewing patterns may produce lower-quality recommendations in smaller markets. Per-market quality monitoring and region-specific fine-tuning are documented as architectural requirements.
- **Creator fairness:** Content from new creators and niche genres must receive fair exposure. The recommendation system includes fairness-aware re-ranking that ensures minimum exposure thresholds for underrepresented content categories.
- **Explainability by design:** Named recommendation rows ("Because You Watched," "Trending Now," "Top 10 in Your Country") serve as natural-language explanations for recommendations. This design decision is documented as an explainability mechanism in the Responsible AI Concepts section.

### 3.9 E7: AI Debt Register

The AI Debt Register tracks Netflix-specific ML technical debt using Sculley et al.'s taxonomy:

- **Feature entanglement (EN-001):** Recommendation models share behavioral features computed by Flink streaming jobs. When a Flink job's windowing logic changes (e.g., from 5-minute to 1-minute windows), all downstream models experience an implicit distribution shift because feature values change even though the feature schema is unchanged. This entanglement is invisible in standard tech debt tracking because the feature schema contract is technically unbroken.
- **Hidden feedback loop (FL-001):** Recommendations influence member viewing behavior, which generates the behavioral data used to retrain recommendation models. This creates a feedback loop where the model increasingly reinforces its own past recommendations, potentially narrowing content diversity over time.
- **Data dependency debt (DD-001):** The recommendation system depends on content metadata features (genre, cast, director, keywords) that are maintained by a separate content team. Changes to the metadata taxonomy (e.g., adding new genre categories or reclassifying existing content) silently affect model behavior without triggering any notification to the ML team.

### 3.10 E6: AI Quality Scenarios

AI Quality Scenarios document measurable quality requirements for AI-specific attributes:

- **Model freshness after market launch (QS-01):** When Netflix launches in a new market, recommendation models must be fine-tuned on regional viewing data within 30 days to achieve regional quality within 90% of global average
- **Concept drift from catalog expansion (QS-02):** When a major content catalog expansion occurs (e.g., 500+ new titles in one month), models must be retrained within 14 days; engagement metrics must not drop below 95% of pre-expansion baseline during the retraining period
- **A/B test guardrail violation (QS-03):** If any guardrail metric (member cancellation rate, playback error rate, customer support contact rate) degrades by more than 0.5% during an A/B test, the experiment must be automatically paused within 1 hour and the ML team notified

---

## 4. Concern Coverage

The following table maps ten AI-specific concerns to their coverage under standard arc42/C4 versus RAD-AI.

| # | AI-Specific Concern | Standard arc42/C4 | RAD-AI | Extension(s) |
|---|---------------------|-------------------|--------|--------------|
| 1 | Non-deterministic component identification | Not captured | **Fully captured** | E1, C4-E3 |
| 2 | Model versioning and lifecycle | Not captured | **Fully captured** | E2 |
| 3 | Streaming vs. batch pipeline distinction | Not captured | **Fully captured** | E3, C4-E1 |
| 4 | DAG dependencies and event-triggered chaining | Not captured | **Fully captured** | E3, E8 |
| 5 | Signal-based cross-workflow coordination | Not captured | **Fully captured** | E8, C4-E2 |
| 6 | Feature Store as architectural element | Partial (misrepresented as generic database) | **Fully captured** | C4-E1, E3 |
| 7 | A/B testing and experimentation infrastructure | Not captured | **Fully captured** | E8, E5 |
| 8 | ML-specific technical debt | Partial (generic debt tracking) | **Fully captured** | E7 |
| 9 | Drift monitoring and model freshness | Not captured | **Fully captured** | E6, E8 |
| 10 | Responsible AI and content fairness | Not captured | **Fully captured** | E4 |

**Summary:**

- **Standard arc42/C4:** 0 of 10 fully captured; 2 of 10 partially captured (Feature Store, ML debt); 8 of 10 structurally undocumentable
- **RAD-AI:** 8 of 10 fully captured; 2 of 10 improved from partial to full coverage

This result mirrors the Uber Michelangelo case study: the same gap pattern (0 fully captured, 2 partial, 8 missing under standard frameworks) appears in a fundamentally different domain.

---

## 5. Key Insight

Netflix and Uber operate in fundamentally different domains (content recommendation versus marketplace logistics), use different ML infrastructure stacks (Metaflow/Maestro versus Gallery/Palette), and face different business constraints (engagement optimization versus real-time prediction serving). Yet both exhibit the identical gap pattern under standard frameworks and the identical improvement under RAD-AI.

This convergence provides evidence that the documentation deficiencies are structural properties of the frameworks rather than domain-specific oversights. Standard arc42 and C4 were designed for deterministic software with stable interfaces and single-lifecycle evolution. When applied to AI-augmented systems, regardless of domain, the same categories of architectural concern (non-determinism, model lifecycle, data lineage, drift monitoring, experimentation, ML-specific debt, responsible AI) fall outside the frameworks' expressive capacity.

RAD-AI's extensions are not domain-specific patches. They address fundamental properties of AI-augmented systems (probabilistic outputs, data-dependent evolution, dual lifecycles, emergent quality attributes) that are present across domains. The cross-domain consistency of the gap pattern and the coverage improvement supports the claim that RAD-AI's extensions are general rather than case-specific.

The Netflix case study also reveals a concern that is particularly pronounced in content recommendation systems: the interplay between A/B testing infrastructure and architecture documentation. At Netflix, no model reaches production without passing through ABlaze's experimentation pipeline. The experimentation infrastructure is therefore not an operational convenience but a first-class architectural component that governs the system's evolution. Standard arc42 has no section for experimentation infrastructure; the Operational AI View (E8) provides this missing section and documents the complete experiment-to-deployment lifecycle that is central to Netflix's ML platform architecture.

---

## References

1. "Supporting Diverse ML Systems at Netflix." Netflix Technology Blog, March 2024.
2. "Maestro: Netflix's Workflow Orchestrator." Netflix Technology Blog, 2024.
3. "Building a Scalable Flink Platform: A Tale of 15,000 Jobs at Netflix." Confluent Current, 2024.
4. "It's All A/Bout Testing: The Netflix Experimentation Platform." Netflix Technology Blog.
5. "Automated Canary Analysis at Netflix with Kayenta." Netflix Technology Blog.
6. Netflix Metaflow open-source documentation. https://metaflow.org
7. "ML Observability: Bringing Transparency to Payments and Beyond." Netflix Technology Blog.
8. "Lessons Learnt From Consolidating ML Models in a Large Scale Recommendation System." Netflix Research, 2024.
