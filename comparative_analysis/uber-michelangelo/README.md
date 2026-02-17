# Uber Michelangelo -- RAD-AI Worked Example

A comprehensive worked example applying the RAD-AI framework to Uber's Michelangelo ML platform, demonstrating what standard arc42/C4 documentation captures versus what RAD-AI extensions reveal.

## System Overview

Uber Michelangelo is Uber's internal ML-as-a-service platform, covering the end-to-end machine learning workflow: data management, model training, evaluation, deployment, prediction serving, and monitoring. First publicly described in 2017, it has since evolved into one of the largest production ML platforms in industry.

### Key Metrics (documented facts)

| Metric | Value | Source |
|--------|-------|--------|
| Models in production | 5,000+ | Uber Engineering Blog (2024) |
| Real-time predictions/sec (peak) | 10--15 million | Uber Engineering Blog (2024--2025) |
| Active ML projects | ~400 | Uber Engineering Blog (2024) |
| Monthly training jobs | 20,000+ | Uber Engineering Blog (2024) |
| Feature Store features (Palette) | 20,000+ | Feature Store Summit 2022 |
| Tier 1 datasets monitored by D3 | 300+ | D3 Blog Post (2023) |
| Data quality monitors (D3) | 100,000+ | D3 Blog Post (2023) |
| GPUs managed | 5,000+ | Uber Engineering Blog (2024) |

### Major AI Components

| Component | Task | Model Architecture | Criticality |
|-----------|------|--------------------|-------------|
| **DeepETA** | Travel time prediction (pickup, trip, delivery) | Linear Transformer + routing engine residual | High -- affects every ride/delivery ETA displayed to users |
| **Dynamic Pricing** | Surge pricing and fare estimation | Gradient boosted trees + demand forecasting | High -- directly affects revenue and driver supply |
| **Fraud Detection** | Payment fraud, account abuse, trip anomalies | Ensemble (GBM + neural networks) | High -- financial loss prevention |
| **Marketplace Matching** | Driver-rider matching optimization | Deep learning + combinatorial optimization | High -- core business logic |
| **Demand Forecasting** | Spatiotemporal demand prediction | Time-series models (LSTM, DeepAR variants) | Medium -- drives driver positioning and incentives |
| **Feature Store (Palette)** | Centralized feature management and serving | N/A (infrastructure) | Critical -- upstream dependency for all models |
| **Gallery** | Model lifecycle management (versioning, staging, deployment) | N/A (infrastructure) | Critical -- governs all model promotions |
| **D3** | Automated data drift detection | Prophet-based anomaly detection | High -- data quality assurance for all pipelines |

### Platform Architecture Layers

| Layer | Components | Technology |
|-------|------------|------------|
| Data Ingestion | Kafka, HDFS Data Lake, Hive | Apache Kafka, HDFS, Hive, Spark |
| Feature Engineering | Palette Feature Store | Cassandra (online), Hive (offline), Samza (streaming) |
| Model Training | Distributed training infrastructure | Spark, Horovod, Ray, TensorFlow, PyTorch, XGBoost |
| Model Management | Gallery lifecycle system | Custom (4-stage lifecycle: exploration, training, evaluation, production) |
| Model Serving | Online prediction service, batch scoring | Triton Inference Server, Kubernetes |
| Monitoring | D3 drift detection, model health monitoring | Prophet, Grafana, PagerDuty |
| Deployment Safety | Canary analysis, shadow deployment | Custom safety scoring framework |

## What Standard Frameworks Miss vs. What RAD-AI Captures

| Concern | Standard arc42/C4 | RAD-AI Extension | Michelangelo Example |
|---------|-------------------|------------------|----------------------|
| Feature Store as first-class component | Appears as a generic "Database" container, indistinguishable from Cassandra | `<<Feature Store>>` stereotype with feature count, freshness SLA, online/offline topology | Palette's 20,000+ features with dual Cassandra (online) / Hive (offline) serving are invisible in standard C4 |
| Model lifecycle stages | Gallery's 4-stage lifecycle collapses into static building blocks | Model Registry View captures versioned lifecycle with promotion gates and rule-based automation | Gallery's `WHEN metrics[mae] <= 5` deployment rules have no home in standard arc42 S5 |
| Data drift monitoring | D3's 100,000+ monitors have no representation in Runtime View | Operational AI View documents drift detection architecture and alert thresholds | D3 reduced time-to-detect from 45+ days to ~2 days; this operational improvement is invisible in standard documentation |
| Non-deterministic boundaries | DeepETA, pricing, and matching all appear as generic "Service" containers | Non-Determinism Boundary Diagram separates deterministic routing engine from probabilistic DeepETA residual | The hybrid routing-engine + neural-network architecture of DeepETA requires explicit boundary documentation |
| ML-specific technical debt | Hidden feedback loops in dynamic pricing are not tracked | AI Debt Register captures feedback loops, feature entanglement, data dependency debt | Dynamic pricing creates feedback loops: surge pricing affects demand, which feeds back into pricing models |
| Deployment safety for ML | Standard deployment is binary (deployed / not deployed) | Operational AI View documents canary analysis, shadow deployment, safety scoring | Uber's Model Excellence Score and shadow testing on 75%+ of critical models have no standard arc42 home |
| Data lineage and provenance | Data flows are generic arrows between boxes | Data Lineage Overlay traces data from source through transformation to model consumption | The path from raw GPS data through Kafka, feature engineering, Palette, to DeepETA inference needs explicit quality gates |
| Responsible AI concerns | No dedicated section for fairness, bias, or explainability | Responsible AI Concepts documents fairness metrics, bias detection, and regulatory requirements | Pricing fairness across geographic regions and rider demographics has no home in standard arc42 S8 |
| AI-specific quality requirements | Quality scenarios cover latency and availability only | AI Quality Scenarios add model freshness SLAs, drift thresholds, fairness constraints | DeepETA must retrain when road network changes; this quality requirement is AI-specific |
| AI decision rationale | Architecture Decision Records lack ML-specific fields | AI-ADR adds dataset, fairness, model lifetime, retraining trigger, explainability, regulatory fields | The decision to use a Linear Transformer over XGBoost for DeepETA requires documenting dataset characteristics, latency-accuracy trade-offs, and retraining strategy |

## Contents

### Standard Documentation (showing limitations)

- [`standard/arc42/03-context-scope.md`](standard/arc42/03-context-scope.md) -- Standard arc42 S3 for Michelangelo
- [`standard/arc42/05-building-block-view.md`](standard/arc42/05-building-block-view.md) -- Standard arc42 S5
- [`standard/arc42/06-runtime-view.md`](standard/arc42/06-runtime-view.md) -- Standard arc42 S6
- [`standard/c4/system-context.md`](standard/c4/system-context.md) -- Standard C4 Level 1
- [`standard/c4/container-diagram.md`](standard/c4/container-diagram.md) -- Standard C4 Level 2

### RAD-AI Extended Documentation

- [`rad-ai/arc42-extensions/03-ai-boundary-delineation.md`](rad-ai/arc42-extensions/03-ai-boundary-delineation.md) -- E1: AI Boundary Delineation
- [`rad-ai/arc42-extensions/05-model-registry-view.md`](rad-ai/arc42-extensions/05-model-registry-view.md) -- E2: Model Registry View
- [`rad-ai/arc42-extensions/06-data-pipeline-view.md`](rad-ai/arc42-extensions/06-data-pipeline-view.md) -- E3: Data Pipeline View
- [`rad-ai/arc42-extensions/08-responsible-ai-concepts.md`](rad-ai/arc42-extensions/08-responsible-ai-concepts.md) -- E4: Responsible AI Concepts
- [`rad-ai/arc42-extensions/09-ai-decision-records/adr-001-eta-prediction-model.md`](rad-ai/arc42-extensions/09-ai-decision-records/adr-001-eta-prediction-model.md) -- E5: AI-ADR for ETA Prediction
- [`rad-ai/arc42-extensions/10-ai-quality-scenarios.md`](rad-ai/arc42-extensions/10-ai-quality-scenarios.md) -- E6: AI Quality Scenarios
- [`rad-ai/arc42-extensions/11-ai-debt-register.md`](rad-ai/arc42-extensions/11-ai-debt-register.md) -- E7: AI Debt Register
- [`rad-ai/arc42-extensions/12-operational-ai-view.md`](rad-ai/arc42-extensions/12-operational-ai-view.md) -- E8: Operational AI View
- [`rad-ai/c4-diagrams/container-diagram.md`](rad-ai/c4-diagrams/container-diagram.md) -- C4-E1: AI Component Stereotypes
- [`rad-ai/c4-diagrams/data-lineage-overlay.md`](rad-ai/c4-diagrams/data-lineage-overlay.md) -- C4-E2: Data Lineage Overlay
- [`rad-ai/c4-diagrams/non-determinism-boundary.md`](rad-ai/c4-diagrams/non-determinism-boundary.md) -- C4-E3: Non-Determinism Boundary

### Gap Analysis

- [`gap-analysis.md`](gap-analysis.md) -- Detailed comparison mapping AI-specific concerns to standard vs. RAD-AI coverage

## Data Sources and Evidence Basis

All architectural details are sourced from publicly available Uber Engineering Blog posts, conference papers, and technical talks. Where specific internal details are not publicly documented, reasonable inferences are marked as such. Key sources:

- Hermann, J. and Del Balso, M. "Meet Michelangelo: Uber's Machine Learning Platform." Uber Engineering Blog, September 2017.
- Sun, C. et al. "Gallery: A Machine Learning Model Management System at Uber." EDBT 2020.
- "D3: An Automated System to Detect Data Drifts." Uber Engineering Blog, February 2023.
- "Raising the Bar on ML Model Deployment Safety." Uber Engineering Blog, 2025.
- "From Predictive to Generative: How Michelangelo Accelerates Uber's AI Journey." Uber Engineering Blog, 2024.
- "DeepETA: How Uber Predicts Arrival Times Using Deep Learning." Uber Engineering Blog, 2022.
- "Scaling Machine Learning at Uber with Michelangelo." Uber Engineering Blog, 2019.
- "Palette Meta Store Journey." Uber Engineering Blog, 2022.
