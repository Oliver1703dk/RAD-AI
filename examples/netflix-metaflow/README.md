# Netflix Metaflow/Maestro ML Platform — RAD-AI Example

A comprehensive worked example applying the RAD-AI framework to Netflix's ML platform infrastructure, demonstrating how standard architecture documentation frameworks (arc42, C4) fall short for AI-augmented ecosystems and how RAD-AI extensions capture the missing concerns.

## System Description

Netflix operates one of the world's largest ML platforms, supporting over 3,000 AI/ML projects that power content recommendation, personalization, content quality assessment, and operational optimization across 200+ million subscribers. The platform processes petabytes of data daily and manages tens of petabytes of models and artifacts.

### Core Platform Components

| Component | Technology | Role | Scale |
|-----------|-----------|------|-------|
| **Metaflow** | Python framework (open-source) | Human-centric ML workflow definition, experiment tracking, artifact management | 3,000+ ML projects; hundreds of millions of compute jobs |
| **Maestro** | Java-based orchestrator (open-source) | Production workflow scheduling, event-driven orchestration, signal-based coordination | Hundreds of thousands of workflows; millions of jobs daily |
| **Apache Flink** | Stream processing | Real-time feature computation, data transformation, event processing | 15,000+ Flink jobs; 60+ PB data processed per day |
| **Apache Kafka** | Message broker | Event streaming backbone, inter-service communication | Trillions of events per day |
| **Amber** (Feature Store) | Custom internal platform | Feature storage and serving for online/offline model consumption | Synchronous and asynchronous feature queries |
| **Metaflow Hosting** | Model serving platform | RESTful model deployment with auto-scaling and scale-to-zero | Production-grade HTTP endpoints |
| **ABlaze** | A/B testing frontend | Experiment allocation, test scheduling, real-time analytics | Concurrent A/B tests across all members |
| **Kayenta** | Canary analysis (with Google) | Automated canary deployment analysis integrated with Spinnaker | Statistical comparison of canary vs. baseline metrics |
| **Spinnaker** | Continuous delivery | Deployment pipelines, canary orchestration, automated rollback | Multi-region deployment |

### AI Components Inventory

| Component | Model Type | Task | Serving Mode |
|-----------|-----------|------|-------------|
| **Recommendation Models** | Ensemble (collaborative filtering + deep learning + graph-based) | Content ranking, homepage personalization, "Because You Watched" rows | Real-time inference + batch pre-computation |
| **Personalization Models** | Multi-task neural networks | UI layout optimization, artwork selection, notification targeting | Real-time inference |
| **Content Quality Models** | Deep neural networks (SemanticGNN) | Video/audio quality prediction, encoding optimization | Batch processing |
| **Search Models** | Transformer-based retrieval | Query understanding, search ranking, intent classification | Real-time inference |
| **Experimentation Models** | Causal inference, contextual bandits | A/B test analysis, treatment effect estimation, interface optimization | Batch analysis |

### Stakeholders

| Stakeholder | Role | Key Concerns |
|-------------|------|-------------|
| ML Platform (MLP) team | Platform operators, Metaflow/Maestro maintainers | Platform reliability, developer experience, scale |
| Data scientists (hundreds) | Model developers, experiment designers | Rapid prototyping, notebook-to-production path, reproducibility |
| ML engineers | Production model owners, pipeline maintainers | Model performance, retraining automation, monitoring |
| Content and product teams | Business stakeholders | Recommendation quality, personalization effectiveness, engagement |
| Trust and safety team | Responsible AI oversight | Content fairness, filter bubble mitigation, geographic equity |
| Netflix members (200M+) | End users | Relevant recommendations, content discovery, streaming quality |

## What Standard Frameworks Miss

Based on the comparative analysis in the RAD-AI paper, documenting Netflix's ML platform under standard arc42/C4 leaves four critical concerns undocumented:

1. **Invisible DAG dependencies** -- Metaflow's Python-native DAGs with event-triggered chaining (via Maestro signals) are invisible in the standard Runtime View, which can only show request-response or message-passing patterns.

2. **Signal-based cross-workflow coordination** -- Maestro's signal service enables one workflow step's output to unblock dependent steps across entirely different workflows. This publisher-subscriber orchestration pattern has no documentation counterpart in standard frameworks.

3. **Streaming vs. batch distinction lost** -- The 15,000+ Flink streaming jobs and batch ETL pipelines serve fundamentally different purposes (real-time feature computation vs. model training data preparation), but standard Building Block Views represent them identically as generic processing components.

4. **A/B testing infrastructure undocumented** -- ABlaze's experiment allocation, Kayenta's canary analysis, and the feedback loop between experiments and model updates have no documentation home in standard arc42 or C4.

## RAD-AI Coverage

| Concern | Standard arc42/C4 | RAD-AI Extension | How It Captures |
|---------|-------------------|-----------------|-----------------|
| DAG dependencies with event triggers | Not expressible in Runtime View | E3: Data Pipeline View + E8: Operational AI View | Explicit pipeline stages with `@trigger_on_finish` chains; Maestro signal dependencies mapped |
| Signal-based cross-workflow coordination | No counterpart | E8: Operational AI View + C4-E2: Data Lineage Overlay | Workflow dependency graph with signal lineage; cross-workflow data provenance |
| Streaming vs. batch distinction | Both appear as generic "Process" | E3: Data Pipeline View + C4-E1: AI Component Stereotypes | `<<Data Pipeline>>` stereotype distinguishes streaming (Flink) from batch (Spark); explicit freshness SLAs per pipeline |
| A/B testing infrastructure | No documentation home | E8: Operational AI View + E5: AI-ADR | A/B testing as first-class operational concern; experiment tracking decisions recorded in AI-ADRs |
| Model versioning and lineage | Models are generic components | E2: Model Registry View | 3,000+ projects tracked with version history, hyperparameters, experiment lineage, Model Cards |
| Feature store interactions | Invisible | C4-E1 `<<Feature Store>>` stereotype + C4-E2 Data Lineage | Amber feature store as first-class building block with online/offline serving paths |
| Non-deterministic boundaries | All components appear equal | C4-E3: Non-Determinism Boundary | Clear partition between deterministic infrastructure (CDN, API gateway) and non-deterministic ML region (recommendations, personalization) |
| Recommendation fairness | Not addressable | E4: Responsible AI Concepts | Content diversity, filter bubble mitigation, geographic equity documented as cross-cutting concerns |
| ML-specific technical debt | Mixed with general tech debt | E7: AI Debt Register | Recommendation entanglement, feature pipeline coupling, hidden feedback loops tracked separately |
| Drift and retraining | Not addressable | E6: AI Quality Scenarios + E8: Operational AI View | Model freshness SLAs, drift thresholds, canary deployment procedures |

## Contents

### Standard Documentation (showing limitations)

- [`standard/arc42/03-context-scope.md`](standard/arc42/03-context-scope.md) -- Standard arc42 section 3
- [`standard/arc42/05-building-block-view.md`](standard/arc42/05-building-block-view.md) -- Standard arc42 section 5
- [`standard/arc42/06-runtime-view.md`](standard/arc42/06-runtime-view.md) -- Standard arc42 section 6
- [`standard/c4/system-context.md`](standard/c4/system-context.md) -- Standard C4 Level 1
- [`standard/c4/container-diagram.md`](standard/c4/container-diagram.md) -- Standard C4 Level 2

### RAD-AI Extended Documentation

- [`rad-ai/arc42-extensions/03-ai-boundary-delineation.md`](rad-ai/arc42-extensions/03-ai-boundary-delineation.md) -- E1: AI Boundary Delineation
- [`rad-ai/arc42-extensions/05-model-registry-view.md`](rad-ai/arc42-extensions/05-model-registry-view.md) -- E2: Model Registry View
- [`rad-ai/arc42-extensions/06-data-pipeline-view.md`](rad-ai/arc42-extensions/06-data-pipeline-view.md) -- E3: Data Pipeline View
- [`rad-ai/arc42-extensions/08-responsible-ai-concepts.md`](rad-ai/arc42-extensions/08-responsible-ai-concepts.md) -- E4: Responsible AI Concepts
- [`rad-ai/arc42-extensions/09-ai-decision-records/adr-001-recommendation-model.md`](rad-ai/arc42-extensions/09-ai-decision-records/adr-001-recommendation-model.md) -- E5: AI-ADR
- [`rad-ai/arc42-extensions/10-ai-quality-scenarios.md`](rad-ai/arc42-extensions/10-ai-quality-scenarios.md) -- E6: AI Quality Scenarios
- [`rad-ai/arc42-extensions/11-ai-debt-register.md`](rad-ai/arc42-extensions/11-ai-debt-register.md) -- E7: AI Debt Register
- [`rad-ai/arc42-extensions/12-operational-ai-view.md`](rad-ai/arc42-extensions/12-operational-ai-view.md) -- E8: Operational AI View
- [`rad-ai/c4-diagrams/container-diagram.md`](rad-ai/c4-diagrams/container-diagram.md) -- C4-E1: AI Component Stereotypes
- [`rad-ai/c4-diagrams/data-lineage-overlay.md`](rad-ai/c4-diagrams/data-lineage-overlay.md) -- C4-E2: Data Lineage Overlay
- [`rad-ai/c4-diagrams/non-determinism-boundary.md`](rad-ai/c4-diagrams/non-determinism-boundary.md) -- C4-E3: Non-Determinism Boundary

### Gap Analysis

- [`gap-analysis.md`](gap-analysis.md) -- Detailed comparison of standard vs. RAD-AI documentation coverage

## Source Attribution

This example is based on publicly available information from Netflix engineering blog posts, conference presentations, and open-source documentation. Where specific internal details are not publicly documented, reasonable inferences are marked as such. Key sources:

- "Supporting Diverse ML Systems at Netflix" (Netflix Tech Blog, March 2024)
- "Maestro: Netflix's Workflow Orchestrator" (Netflix Tech Blog, 2024)
- "Building a Scalable Flink Platform: A Tale of 15,000 Jobs at Netflix" (Confluent Current, 2024)
- "It's All A/Bout Testing: The Netflix Experimentation Platform" (Netflix Tech Blog)
- "Automated Canary Analysis at Netflix with Kayenta" (Netflix Tech Blog)
- Netflix Metaflow open-source documentation (metaflow.org)
- "ML Observability: Bringing Transparency to Payments and Beyond" (Netflix Tech Blog)
- "Lessons Learnt From Consolidating ML Models in a Large Scale Recommendation System" (Netflix Research, 2024)
