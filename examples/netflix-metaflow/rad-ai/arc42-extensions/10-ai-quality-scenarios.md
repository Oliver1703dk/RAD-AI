# AI Quality Scenarios

> **Extends:** arc42 S10 -- Quality Requirements
>
> **System:** Netflix ML Platform (Metaflow/Maestro)
>
> **Last Updated:** 2026-02-17

## Purpose

Define quality scenarios for AI-specific quality attributes that traditional scenarios (response time, throughput, availability) do not cover: model freshness, drift tolerance, explainability, recommendation diversity, and robustness. These scenarios complement standard quality requirements and provide measurable acceptance criteria for Netflix's ML platform. At Netflix's scale, even small degradations in model quality affect millions of members.

## Quality Attribute Definitions

| Quality Attribute | Definition | Why It Matters for Netflix |
|-------------------|-----------|---------------------------|
| Model Freshness | How recent the model's training data and parameters are relative to current member behavior and content catalog | New content launches, seasonal viewing shifts, and global events change member preferences rapidly; stale models recommend outdated content and miss new titles |
| Drift Tolerance | Acceptable degree of data or concept drift before intervention is required | Gradual shifts in viewing patterns (post-pandemic, new market launches, content catalog evolution) silently degrade recommendation quality if undetected |
| Recommendation Diversity | Variety of content types, genres, origins, and ages in a member's recommendations | Filter bubble convergence reduces content discovery, long-term engagement, and member retention; Netflix has publicly linked diversity to retention |
| Serving Freshness | How quickly changes in member behavior are reflected in recommendations | A member who just finished a series expects immediate recommendation updates, not stale suggestions based on yesterday's profile |
| Robustness | Resilience to unusual inputs, traffic spikes, and infrastructure failures | Global events (new season drops, live events) create extreme traffic spikes; model serving must gracefully degrade rather than fail |

## Quality Scenarios

### QS-01: Model Freshness -- Recommendation Model After Content Launch

| Element | Description |
|---------|-------------|
| **Source** | Content catalog event: major new title or season launch (e.g., new season of a flagship series) |
| **Stimulus** | Recommendation model (MDL-REC) has not incorporated viewing signal from the new title (model trained on data before launch) |
| **Artifact** | MDL-REC (recommendation ensemble) |
| **Environment** | Production; first 48 hours after major content launch |
| **Response** | Real-time features (via Flink streaming pipeline) capture member interactions with new content within seconds. Contextual bandit exploration ensures new title is recommended to relevant members even before engagement data accumulates. Model retraining with new content signal initiated within 24 hours via Maestro-triggered pipeline. |
| **Response Measure** | New title appears in relevant recommendation rows within 1 hour of launch (via popularity and editorial injection); personalized recommendations incorporating new title engagement signal available within 48 hours of launch |

### QS-02: Model Freshness -- Feature Store Staleness

| Element | Description |
|---------|-------------|
| **Source** | Feature freshness monitor (continuous) |
| **Stimulus** | Amber Feature Store online features are stale (> 1 hour since last update) due to Flink pipeline delay or failure |
| **Artifact** | All real-time serving models (MDL-REC, MDL-PERS, MDL-SEARCH) |
| **Environment** | Production; degraded data infrastructure |
| **Response** | Feature freshness metadata propagated to models; model confidence scores reduced proportionally to staleness. If staleness > 4 hours, models switch to batch-computed features (last nightly refresh) with "degraded freshness" flag. Alert to Data Engineering team for pipeline recovery. |
| **Response Measure** | Staleness detected within 5 minutes; confidence adjustment applied immediately; batch fallback activated within 10 minutes; no silent degradation (all stale predictions flagged in monitoring) |

### QS-03: Drift Tolerance -- Seasonal Viewing Pattern Shift

| Element | Description |
|---------|-------------|
| **Source** | Drift monitoring system (daily evaluation) |
| **Stimulus** | Recommendation engagement metrics (click-through rate, play-through rate) degrade by > 5% relative to 30-day rolling baseline during a seasonal shift (e.g., summer viewing patterns differ from winter) |
| **Artifact** | MDL-REC (recommendation ensemble) |
| **Environment** | Production; seasonal transition period |
| **Response** | Drift detected via engagement metric monitoring. Retraining pipeline triggered with augmented training data emphasizing recent seasonal patterns. Contextual bandit exploration rate temporarily increased from 5% to 10% to accelerate learning of new patterns. Canary deployment of retrained model via Spinnaker/Kayenta. |
| **Response Measure** | Seasonal drift detected within 3 days; retrained model deployed within 7 days of detection; engagement metrics recover to within 2% of pre-drift baseline within 14 days |

### QS-04: Drift Tolerance -- New Market Behavior

| Element | Description |
|---------|-------------|
| **Source** | Regional engagement monitoring |
| **Stimulus** | Netflix launches in a new market or significantly expands content catalog for an existing market, causing member behavior patterns to diverge from global training distribution |
| **Artifact** | MDL-REC, MDL-SEARCH |
| **Environment** | Production; market expansion event |
| **Response** | Region-specific evaluation slices detect performance gap. Multi-task model produces acceptable baseline recommendations via transfer learning from similar markets. Region-specific features weighted more heavily in model scoring. If gap > 10% vs. established markets, targeted retraining with region-augmented data. |
| **Response Measure** | Regional performance gap detected within 1 week; recommendation quality in new market reaches 90% of global average within 30 days |

### QS-05: Recommendation Diversity -- Filter Bubble Prevention

| Element | Description |
|---------|-------------|
| **Source** | Diversity monitoring system (weekly batch analysis) |
| **Stimulus** | Genre entropy of recommendation sets decreases by > 15% for any member cohort (e.g., members who primarily watch one genre see increasingly narrow recommendations) |
| **Artifact** | MDL-REC (recommendation ensemble) |
| **Environment** | Production; continuous monitoring |
| **Response** | Diversity guardrail alert triggered. Analysis determines whether diversity decrease is due to model behavior (requires intervention) or genuine member preference narrowing (acceptable). If model-driven: increase contextual bandit exploration for affected cohort; adjust multi-task diversity loss weight in next retraining; inject editorial "discovery" rows for affected members. |
| **Response Measure** | Diversity degradation detected within 1 week; root cause analysis completed within 3 days; if model-driven, corrective retraining initiated within 7 days; genre entropy restored to baseline within 21 days |

### QS-06: Serving Freshness -- Real-Time Preference Update

| Element | Description |
|---------|-------------|
| **Source** | Member interaction event (e.g., member binge-watches a new genre) |
| **Stimulus** | Member's real-time behavioral features have changed significantly (new genre affinity detected after watching 3+ titles in a genre they have never explored before) |
| **Artifact** | MDL-REC, MDL-PERS (via Amber Feature Store real-time features) |
| **Environment** | Production; normal operation |
| **Response** | Flink streaming pipeline computes updated member behavioral features within seconds. Updated features written to Amber online store. Next recommendation request from this member uses fresh features, reflecting new genre interest. No model retraining needed; real-time features capture behavioral shift. |
| **Response Measure** | Feature update latency < 5 seconds from interaction event; next recommendation request reflects updated preferences; member sees genre-relevant content in next homepage render |

### QS-07: Robustness -- Traffic Spike on Content Launch

| Element | Description |
|---------|-------------|
| **Source** | Major content launch event (new season of popular series) causing recommendation traffic spike |
| **Stimulus** | Recommendation request volume increases 3--5x within minutes as millions of members open the app simultaneously |
| **Artifact** | MDL-REC serving infrastructure (Metaflow Hosting) |
| **Environment** | Production; high-traffic event |
| **Response** | Auto-scaling provisions additional serving instances. If scaling cannot keep pace, graceful degradation: return pre-computed batch recommendations from Cassandra cache instead of real-time model inference. Load shedding prioritizes members currently browsing over pre-fetch requests. |
| **Response Measure** | P95 serving latency remains < 250ms for 95% of requests during spike; zero member-visible errors; graceful degradation activated if needed within 30 seconds; full real-time serving restored within 5 minutes of traffic normalization |

### QS-08: Robustness -- Flink Job Failure Cascade

| Element | Description |
|---------|-------------|
| **Source** | Infrastructure failure causing multiple Flink streaming jobs to fail simultaneously |
| **Stimulus** | > 10% of the 15,000+ Flink jobs fail within a 5-minute window (e.g., Kafka cluster issue, shared state backend failure) |
| **Artifact** | PL-002 (Real-Time Feature Computation), Amber Feature Store |
| **Environment** | Production; infrastructure failure |
| **Response** | Flink jobs checkpoint regularly (sub-minute intervals); failed jobs restart from last checkpoint. During recovery, affected features in Amber become stale. Feature freshness metadata propagated to consuming models. Models reduce confidence for predictions using stale features. If recovery takes > 30 minutes, batch-computed features activated as fallback. |
| **Response Measure** | Flink job recovery from checkpoint < 30 seconds per job; feature staleness detected within 5 minutes; batch fallback activated within 30 minutes if needed; zero data loss (Kafka replay provides at-least-once guarantees) |

### QS-09: Experiment Validity -- A/B Test Statistical Rigor

| Element | Description |
|---------|-------------|
| **Source** | Experiment analysis pipeline (continuous during experiment) |
| **Stimulus** | A/B test for a new recommendation model variant has been running for 7 days and approaches statistical significance |
| **Artifact** | MDL-EXP (experimentation engine), ABlaze |
| **Environment** | Production; active experiment |
| **Response** | Sequential testing framework (Netflix's published approach) computes always-valid confidence intervals. Test stopped when confidence interval excludes zero for primary metric AND all guardrail metrics (diversity, latency, error rate) are non-degraded. If primary metric is positive but a guardrail is violated, experiment flagged for manual review. |
| **Response Measure** | False positive rate controlled at < 5% via sequential testing; guardrail violations detected in real-time; experiment decisions made within 14 days for standard tests; no model deployed that violates guardrail metrics |

## Quality Scenario Summary

| ID | Attribute | Model(s) | Priority | Source |
|----|-----------|----------|----------|--------|
| QS-01 | Model Freshness | MDL-REC | High | Public (content launch patterns documented) |
| QS-02 | Model Freshness | All serving models | Critical | Inferred from Flink/Amber architecture |
| QS-03 | Drift Tolerance | MDL-REC | High | Inferred from seasonal patterns |
| QS-04 | Drift Tolerance | MDL-REC, MDL-SEARCH | Medium | Inferred from market expansion |
| QS-05 | Recommendation Diversity | MDL-REC | High | Public (Netflix research on diversity) |
| QS-06 | Serving Freshness | MDL-REC, MDL-PERS | High | Public (real-time feature architecture) |
| QS-07 | Robustness | MDL-REC serving | Critical | Public (Netflix traffic spike handling) |
| QS-08 | Robustness | PL-002, Amber | Critical | Inferred from Flink architecture |
| QS-09 | Experiment Validity | MDL-EXP | High | Public (Netflix sequential testing publications) |
