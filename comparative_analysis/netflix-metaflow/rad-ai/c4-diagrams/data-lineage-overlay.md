# Data Lineage Overlay -- Netflix ML Platform

> **Diagram type:** C4 Data Lineage Overlay (RAD-AI extension)
>
> **Purpose:** Trace data provenance from member interaction events through transformation to model consumption, annotated with freshness requirements, processing modes (streaming vs. batch), and feedback loops. This overlay makes visible the critical streaming/batch distinction that standard C4 diagrams cannot express.

## Data Source Inventory

| Source ID | Name | Type | Freshness | Volume | Processing Mode |
|-----------|------|------|-----------|--------|----------------|
| DS-001 | Member Viewing Events | Kafka Stream | Real-time (sub-second) | Hundreds of thousands of events/sec peak | Streaming (Flink) |
| DS-002 | Member UI Interactions | Kafka Stream | Real-time (sub-second) | Hundreds of thousands of events/sec peak | Streaming (Flink) |
| DS-003 | Search Queries | Kafka Stream | Real-time | Tens of thousands of queries/sec | Streaming (Flink) |
| DS-004 | Device Telemetry | Kafka Stream | Real-time (5s intervals) | Millions of reports/min | Streaming (Flink) |
| DS-005 | Content Catalog | Internal API | On catalog change | ~15,000 titles with metadata | Batch (Spark) + event-driven |
| DS-006 | Content Embeddings | Batch pipeline output | Quarterly recomputation | All titles (dense vectors, dim 256) | Batch (Metaflow) |
| DS-007 | Member Profiles | Aggregated from DS-001/DS-002 | Daily aggregation | 200M+ profiles | Batch (Spark) |

## Data Lineage Diagram

```mermaid
graph LR
    subgraph Sources ["Data Sources"]
        direction TB
        DS1["DS-001: Viewing Events<br/>[Kafka Stream]<br/>Real-time, sub-second<br/>100K+ events/sec"]
        DS2["DS-002: UI Interactions<br/>[Kafka Stream]<br/>Clicks, scrolls, hovers<br/>100K+ events/sec"]
        DS3["DS-003: Search Queries<br/>[Kafka Stream]<br/>Text + voice queries<br/>10K+ queries/sec"]
        DS4["DS-004: Device Telemetry<br/>[Kafka Stream]<br/>Bandwidth, latency<br/>Millions/min"]
        DS5["DS-005: Content Catalog<br/>[Internal API]<br/>Metadata, genres, cast<br/>~15K titles"]
        DS6["DS-006: Content Embeddings<br/>[Batch Pipeline]<br/>SemanticGNN vectors<br/>Quarterly refresh"]
    end

    subgraph StreamingPath ["Streaming Path (Flink)"]
        direction TB
        KafkaTopics["Kafka Topics<br/>views.raw, clicks.raw,<br/>search.raw, device.raw"]
        FlinkDedup["Flink: Dedup + Validate<br/>Schema validation,<br/>event-time watermarks"]
        FlinkWindow["Flink: Windowed Aggregation<br/>1-min, 5-min, 1-hr windows<br/>View counts, click rates,<br/>session features"]
        FlinkFeature["Flink: Feature Computation<br/>Behavioral features,<br/>recency-weighted prefs,<br/>engagement scores"]
    end

    subgraph BatchPath ["Batch Path (Spark)"]
        direction TB
        S3Raw["S3: Raw Event Archive<br/>Partitioned by date<br/>Petabytes"]
        SparkETL["Spark: Historical ETL<br/>Dedup, join, aggregate<br/>Nightly pipeline"]
        SparkFeature["Spark: Training Features<br/>Point-in-time correct<br/>No future leakage"]
        SparkDataset["Spark: Training Dataset<br/>Labeled examples<br/>Train/val/test split"]
    end

    subgraph FeatureStore ["Amber Feature Store [FS]"]
        direction TB
        AmberOnline["Online Store<br/>Sub-ms serving<br/>Real-time features<br/>Freshness: seconds"]
        AmberOffline["Offline Store<br/>Batch queries<br/>Historical features<br/>Freshness: daily"]
    end

    subgraph Consumption ["Model Consumption"]
        direction TB
        MDL_REC["Recommendation Models<br/>[ML]<br/>Consumes: behavioral +<br/>content + contextual features<br/>Freshness req: seconds"]
        MDL_PERS["Personalization Models<br/>[ML]<br/>Consumes: member +<br/>session + device features<br/>Freshness req: seconds"]
        MDL_SEARCH["Search Models<br/>[ML]<br/>Consumes: query +<br/>member + content features<br/>Freshness req: seconds"]
        MDL_QUAL["Content Quality Models<br/>[ML]<br/>Consumes: content +<br/>device features<br/>Freshness req: daily"]
    end

    subgraph Feedback ["Feedback Loops"]
        direction TB
        ImplicitFB["Implicit Feedback<br/>Rec shown + viewed/skipped<br/>Search result + clicked"]
        ExplicitFB["Explicit Feedback<br/>Thumbs up/down<br/>Ratings, preferences"]
        ExperimentFB["Experiment Outcomes<br/>A/B test metrics<br/>Treatment effects"]
    end

    DS1 --> KafkaTopics
    DS2 --> KafkaTopics
    DS3 --> KafkaTopics
    DS4 --> KafkaTopics

    KafkaTopics --> FlinkDedup
    FlinkDedup --> FlinkWindow
    FlinkWindow --> FlinkFeature

    FlinkFeature -->|"real-time features<br/>(freshness: seconds)"| AmberOnline
    FlinkDedup -->|"validated events<br/>(archived)"| S3Raw

    S3Raw --> SparkETL
    DS5 -->|"catalog snapshots"| SparkETL
    DS6 -->|"content embeddings"| SparkETL
    SparkETL --> SparkFeature
    SparkFeature --> AmberOffline
    SparkFeature --> SparkDataset

    AmberOnline -->|"online features<br/>(sync, sub-ms)"| MDL_REC
    AmberOnline -->|"online features"| MDL_PERS
    AmberOnline -->|"online features"| MDL_SEARCH
    AmberOffline -->|"offline features<br/>(batch, daily)"| SparkDataset
    AmberOffline -->|"offline features"| MDL_QUAL

    SparkDataset -->|"training data"| MDL_REC
    SparkDataset -->|"training data"| MDL_SEARCH

    MDL_REC -->|"recommendations<br/>shown to members"| ImplicitFB
    MDL_SEARCH -->|"search results<br/>shown to members"| ImplicitFB
    MDL_PERS -->|"UI layout<br/>shown to members"| ImplicitFB
    ImplicitFB -->|"becomes future<br/>training data"| DS1
    ImplicitFB -->|"becomes future<br/>training data"| DS2
    ExplicitFB -->|"member signals"| DS2
    ExperimentFB -->|"experiment metrics"| SparkETL
```

## Lineage Details

### DS-001 (Viewing Events) -> MDL-REC (Recommendation) Path

**Streaming path (real-time serving):**

| Stage | Input | Transformation | Output | Quality Gate |
|-------|-------|---------------|--------|-------------|
| Kafka ingestion | Raw view event (play, pause, stop, complete) from Netflix client | Schema validation (Avro); deduplication by event_id + timestamp | Validated viewing events in Kafka topic | Schema conformance 100%; dedup rate logged |
| Flink windowed aggregation | Validated viewing events | Tumbling windows (1-min, 5-min, 1-hour): compute view count, avg duration, genre distribution per member | Time-windowed behavioral aggregates | Window completeness > 99%; late events handled within 5-min allowed lateness |
| Flink feature computation | Windowed aggregates + cached member profile | Compute engagement score, recency-weighted genre preferences, session momentum, viewing velocity | Real-time behavioral feature vector | Feature null rate < 0.1%; value range validation |
| Amber online materialization | Feature vector | Write to online store; key: member_id; TTL: 5 minutes (auto-refresh by next window) | Online features serving real-time inference | Write latency < 10ms; freshness < 5 seconds from source event |

**Batch path (model training):**

| Stage | Input | Transformation | Output | Quality Gate |
|-------|-------|---------------|--------|-------------|
| S3 archival | Validated Kafka events (written by Flink) | Partitioned by date, stored in Parquet | Raw event archive in S3 | Partition completeness check; record count reconciliation |
| Spark ETL | S3 raw events + content catalog + member profiles | Deduplication, join with content metadata, compute historical behavioral features (30/60/90-day windows) | Historical behavioral features per member per time window | No future data leakage (point-in-time correctness enforced) |
| Training dataset construction | Historical features + labels (viewed/not-viewed) | Negative sampling, label construction (implicit feedback), train/val/test split by time | Training dataset in S3 (Parquet) | Label distribution logged; class balance within expected range |

### Streaming vs. Batch: Freshness Comparison

| Feature Type | Streaming Freshness | Batch Freshness | Impact of Staleness |
|-------------|--------------------|-----------------|--------------------|
| Recent viewing behavior (last 1 hour) | < 5 seconds | 24 hours (stale) | High: member who just finished a series expects immediate recommendation update |
| Genre preferences (last 7 days) | < 1 minute | 24 hours | Medium: gradual preference shifts are captured daily |
| Long-term taste profile (last 90 days) | Not applicable (too expensive to recompute in streaming) | 24 hours | Low: long-term preferences change slowly |
| Content popularity (trending) | < 1 minute | 24 hours (stale) | High: trending content changes rapidly; stale popularity = outdated "Trending Now" |
| Device context (bandwidth, resolution) | < 5 seconds | Not applicable (always real-time) | High: encoding optimization requires current network conditions |

This freshness analysis demonstrates why the streaming/batch distinction matters architecturally: real-time features make recommendations responsive to a member's current session, while batch features capture long-term patterns that inform the model's underlying understanding of member preferences.

## Feedback Loop Analysis

### Primary Feedback Loop: Recommendations -> Viewing -> Training Data

```mermaid
graph TD
    RecModel["Recommendation Model<br/>produces ranked content"]
    Homepage["Homepage Display<br/>shows recommendations"]
    MemberBehavior["Member Behavior<br/>views, skips, abandons"]
    KafkaEvents["Kafka Event Stream<br/>captures interactions"]
    FlinkProcess["Flink Processing<br/>computes features"]
    TrainingData["Training Dataset<br/>labels: viewed = positive,<br/>shown-but-not-viewed = negative"]
    Retrain["Model Retraining<br/>learns from new data"]

    RecModel --> Homepage
    Homepage --> MemberBehavior
    MemberBehavior --> KafkaEvents
    KafkaEvents --> FlinkProcess
    FlinkProcess --> TrainingData
    TrainingData --> Retrain
    Retrain --> RecModel
```

**Feedback loop risks:**

| Risk | Description | Severity |
|------|-------------|----------|
| Popularity amplification | Recommended content gets viewed more, generating positive training signal, causing more recommendation | High |
| Position bias | Higher-ranked items get more clicks regardless of true relevance; model learns to replicate its own ranking | Medium |
| Filter bubble convergence | Member sees narrow content; narrows future preferences; model narrows further | High |
| Cold-start content disadvantage | New titles have no viewing data; model cannot recommend them; they never accumulate viewing data | Medium |

**Mitigations documented:**

| Mitigation | Mechanism | Effectiveness |
|------------|-----------|---------------|
| Contextual bandit exploration | 5--10% of recommendations are exploratory (not based on model ranking) | Breaks popularity feedback loop; provides unbiased training signal |
| Position-debiased training | Inverse propensity weighting accounts for position effect in training | Reduces position bias in learned rankings |
| Diversity post-processing | Genre entropy constraints applied to final recommendation sets | Prevents filter bubble convergence at serving time |
| Editorial injection | Editorially curated "Discovery" rows bypass ML ranking | Ensures new and niche content gets exposure regardless of model |

## Schema Registry

| Schema ID | Version | Domain | Used By | Last Breaking Change |
|-----------|---------|--------|---------|---------------------|
| SCH-VIEW | v4.1 | Viewing events | PL-001 (ingestion), PL-002 (Flink), PL-004 (training) | v4.0: added `view_context` field (device, session_id) |
| SCH-CLICK | v3.0 | UI interaction events | PL-001, PL-002, PL-004 | v3.0: unified click/scroll/hover into interaction event |
| SCH-SEARCH | v2.2 | Search events | PL-001, PL-002, MDL-SEARCH training | v2.0: added `query_intent` classification field |
| SCH-FEATURE | v5.0 | Feature vectors (Amber) | All models (online + offline) | v5.0: initial Amber schema (managed via Amber Schema Registry) |
