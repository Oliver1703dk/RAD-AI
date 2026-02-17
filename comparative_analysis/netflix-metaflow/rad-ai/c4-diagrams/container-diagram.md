# C4 Container Diagram — Netflix ML Platform

> **Diagram type:** C4 Level 2 -- Container Diagram
>
> **RAD-AI extensions:** AI component stereotypes `[ML]`, `[FS]`, `[DP]`, `[MON]`

## Purpose

This diagram decomposes the Netflix ML Platform into its runtime containers, applying RAD-AI stereotypes to distinguish ML models, feature stores, data pipelines, and monitors from conventional software containers. Each AI container is annotated with its type, scale, and key characteristics. This view makes visible what the standard C4 container diagram hides: the 3,000+ ML projects, the feature store mediating between data and models, the streaming vs. batch distinction in data processing, and the experimentation infrastructure.

## Container Inventory

| Container | Stereotype | Technology | Description |
|-----------|-----------|------------|-------------|
| API Gateway (Zuul) | -- | Java/Zuul | Request routing, auth, rate limiting, A/B test allocation |
| Recommendation Models | `[ML]` | PyTorch, DGL | Ensemble: CF + DNN + GNN for content ranking |
| Personalization Models | `[ML]` | PyTorch | Multi-task NN for UI layout, artwork, notifications |
| Search Models | `[ML]` | PyTorch (Transformer) | Query understanding and result ranking |
| Content Quality Models | `[ML]` | PyTorch (SemanticGNN) | Video/audio quality prediction, encoding optimization |
| Amber Feature Store | `[FS]` | Custom (online + offline) | Feature serving: sync (sub-ms) and async (batch) |
| Flink Streaming Cluster | `[DP]` | Apache Flink | 15,000+ streaming jobs; real-time feature computation |
| Spark Batch Cluster | `[DP]` | Apache Spark | Batch ETL, training data preparation |
| Metaflow Metadata Service | -- | Python/Flask | Experiment tracking, artifact management, run lineage |
| Maestro Orchestrator | -- | Java/Spring | Workflow scheduling, signal-based coordination |
| ABlaze Experimentation | -- | React/Node.js | A/B test management, allocation, analysis |
| ML Observability | `[MON]` | Custom + Grafana | Drift detection, model health, feature freshness |
| Kayenta Canary Analysis | `[MON]` | Kayenta (open-source) | Automated canary deployment analysis |
| Apache Kafka | -- | Kafka | Event streaming backbone (trillions of events/day) |
| S3 Data Lake | -- | AWS S3 | Model artifacts, training data (petabytes) |
| Cassandra | -- | Apache Cassandra | Online storage, prediction cache |
| Spinnaker | -- | Spinnaker | Continuous delivery, canary orchestration |

## Container Diagram

```mermaid
graph TB
    DS["Data Scientist<br/>[Person]"]
    MLE["ML Engineer<br/>[Person]"]
    Member["Netflix Member<br/>[Person]"]

    StreamingApp["Netflix App<br/>[External System]<br/>Mobile, TV, Web"]

    subgraph NML ["Netflix ML Platform"]
        direction TB

        APIGw["API Gateway (Zuul)<br/>[Container]<br/>Auth, routing, A/B allocation"]

        subgraph AIModels ["AI Models"]
            direction TB
            RecModel["Recommendation Models<br/>[ML] Ensemble (CF+DNN+GNN)<br/>3,000+ projects<br/>Real-time + batch serving"]
            PersModel["Personalization Models<br/>[ML] Multi-task NN<br/>Layout, artwork, targeting<br/>Real-time serving"]
            SearchModel["Search Models<br/>[ML] Transformer<br/>Query understanding + ranking<br/>Real-time serving"]
            QualModel["Content Quality Models<br/>[ML] SemanticGNN<br/>Encoding optimization<br/>Batch processing"]
        end

        subgraph DataInfra ["Data Infrastructure"]
            direction TB
            Kafka["Apache Kafka<br/>[Container: Message Broker]<br/>Trillions of events/day"]
            FlinkCluster["Flink Streaming Cluster<br/>[DP] 15,000+ jobs<br/>60+ PB/day processed<br/>Real-time features"]
            SparkCluster["Spark Batch Cluster<br/>[DP] Nightly ETL<br/>Training data preparation"]
            AmberFS["Amber Feature Store<br/>[FS] Online: sub-ms sync<br/>Offline: batch async<br/>1,000+ features"]
        end

        subgraph MLOps ["ML Operations"]
            direction TB
            MFMeta["Metaflow Metadata<br/>[Container]<br/>Run tracking, artifacts,<br/>experiment lineage"]
            Maestro["Maestro Orchestrator<br/>[Container]<br/>Signal-based coordination,<br/>workflow scheduling"]
            ABlaze["ABlaze<br/>[Container]<br/>A/B test allocation,<br/>sequential analysis"]
        end

        subgraph Monitoring ["Operational AI"]
            direction TB
            MLObs["ML Observability<br/>[MON] Drift detection,<br/>feature freshness,<br/>model health tracking"]
            Kayenta["Kayenta<br/>[MON] Automated canary<br/>analysis, statistical<br/>comparison"]
        end

        subgraph Storage ["Storage"]
            direction TB
            S3["S3 Data Lake<br/>[Container]<br/>Petabytes of artifacts"]
            Cass["Cassandra<br/>[Container]<br/>Online predictions,<br/>member profiles"]
        end

        Spin["Spinnaker<br/>[Container: CD]<br/>Canary deployment,<br/>rollback orchestration"]
    end

    DS --> MFMeta
    MLE --> Maestro
    MLE --> MLObs
    Member --> StreamingApp

    StreamingApp -->|"rec requests"| APIGw
    APIGw -->|"routed by A/B<br/>allocation"| RecModel
    APIGw -->|"layout requests"| PersModel
    APIGw -->|"search queries"| SearchModel

    StreamingApp -->|"interaction events<br/>(views, clicks)"| Kafka

    Kafka -->|"event streams"| FlinkCluster
    FlinkCluster -->|"real-time features<br/>(sub-second freshness)"| AmberFS
    FlinkCluster -->|"processed events"| S3

    S3 --> SparkCluster
    SparkCluster -->|"historical features<br/>(daily refresh)"| AmberFS
    SparkCluster -->|"training datasets"| S3

    AmberFS -->|"online features<br/>(sync, sub-ms)"| RecModel
    AmberFS -->|"online features"| PersModel
    AmberFS -->|"online features"| SearchModel
    AmberFS -->|"offline features<br/>(batch)"| S3

    S3 -->|"training data +<br/>model artifacts"| MFMeta
    Maestro -->|"orchestrate<br/>training workflows"| MFMeta
    Maestro -->|"signal: model_trained"| Spin
    Spin -->|"canary deploy"| RecModel
    Spin -->|"canary metrics"| Kayenta

    RecModel --> MLObs
    PersModel --> MLObs
    SearchModel --> MLObs
    QualModel --> MLObs
    AmberFS -->|"feature freshness"| MLObs

    RecModel -->|"predictions"| Cass
    ABlaze -->|"allocation rules"| APIGw
    Kayenta -->|"canary pass/fail"| Spin
```

## Stereotype Annotations

### ML Components

| Model ID | Container | Serving Mode | Key Characteristic |
|----------|-----------|-------------|-------------------|
| MDL-REC | Recommendation Models | Real-time (synchronous) + batch (pre-compute) | Ensemble of CF + DNN + GNN; multi-task architecture serving multiple recommendation surfaces |
| MDL-PERS | Personalization Models | Real-time (synchronous) | Multi-task NN; personalizes UI layout, artwork selection, and notification targeting per member |
| MDL-SEARCH | Search Models | Real-time (synchronous) | Transformer-based; query understanding + content retrieval + personalized ranking |
| MDL-QUAL | Content Quality Models | Batch | SemanticGNN; predicts perceptual quality per encoding configuration |

### Feature Store [FS]

Amber serves as the central feature mediation layer between data infrastructure and ML models:

- **Online serving:** Sub-millisecond synchronous lookups for real-time model inference. Features keyed by member ID, content ID, or context.
- **Offline serving:** Batch feature retrieval for model training with point-in-time correctness (prevents data leakage).
- **Feature groups:** 1,000+ features organized by domain (member behavioral, content, contextual, device).
- **Freshness contract:** Online features refreshed in real-time by Flink (< 5s from source event); offline features refreshed daily by Spark.

### Data Pipeline [DP]

| Pipeline | Type | Scale | Freshness |
|----------|------|-------|-----------|
| Flink Streaming Cluster | Continuous streaming | 15,000+ concurrent jobs; 60+ PB/day | Sub-second to seconds |
| Spark Batch Cluster | Scheduled batch | Nightly runs; petabytes per run | Daily |

The streaming/batch distinction is architecturally significant: Flink provides real-time features that make recommendations responsive to current member behavior, while Spark provides historical features that capture long-term patterns for model training.

### Monitoring [MON]

| Monitor | Scope | Key Capabilities |
|---------|-------|-----------------|
| ML Observability | All production models and features | Drift detection (data, concept, prediction), feature freshness tracking, model performance degradation alerting |
| Kayenta | Model deployments | Automated canary analysis comparing candidate model metrics against baseline using configurable statistical tests |

## Data Flow Summary

1. **Ingestion path:** Member interactions -> Netflix App -> Kafka -> Flink (streaming) -> Amber (online features) + S3 (raw events).
2. **Feature computation (real-time):** Kafka -> Flink (15,000+ jobs) -> Amber online store. Sub-second freshness.
3. **Feature computation (batch):** S3 -> Spark -> Amber offline store. Daily refresh.
4. **Real-time inference:** API Gateway -> ML Model -> Amber (feature lookup) -> prediction with confidence.
5. **Training:** S3 (training data) + Amber (offline features) -> Metaflow training workflow -> S3 (model artifacts).
6. **Deployment:** Metaflow -> Maestro (signal: model_trained) -> Spinnaker (canary deploy) -> Kayenta (analysis) -> production.
7. **Experimentation:** ABlaze (allocation) -> API Gateway (routing) -> ML Model variants -> ABlaze (metric collection + analysis).
8. **Monitoring loop:** All ML containers -> ML Observability -> Grafana/alerts -> Maestro (retraining trigger if drift detected).
