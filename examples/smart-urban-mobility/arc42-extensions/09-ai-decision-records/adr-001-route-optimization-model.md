# AI Architecture Decision Record

> **Extends:** arc42 S9 -- Architecture Decisions
>
> Based on [MADR](https://adr.github.io/madr/) with AI-specific extensions.

---

## AI-ADR-001: Route Optimization Model Selection

**Date:** 2025-06-14
**Status:** Accepted
**Deciders:** J. van der Berg (ML Platform Lead), M. Chen (Data Scientist), A. Kowalski (ML Engineer), Dr. R. Hendriks (City Transport Authority Chief Architect)

### Context

The Smart Urban Mobility (SUM) platform requires a model to recommend optimal routes across multiple transport modes (bus, metro, tram, bike-share, ride-share) for commuters in real time. The model must integrate real-time traffic conditions, weather data, transit schedules, and demand forecasts (from MDL-002) to produce route recommendations that minimize commuter travel time while balancing load across the transport network.

Key constraints:
- **Latency:** Route recommendations must be served within 200ms (P95) to meet the mobile app's UX requirements. The inference budget is approximately 150ms after accounting for network overhead and feature retrieval (~50ms).
- **Explainability:** The city transport authority requires the ability to understand and audit why specific routes are recommended, particularly when investigating complaints about service equity across zones.
- **Scale:** The system serves approximately 850 requests/second during peak morning commute (07:00--09:00) and must handle 2x capacity for planned growth over the next 18 months.
- **Regulatory:** The route optimizer is classified as medium-risk under the EU AI Act. While per-decision explanations are not legally mandated, the city authority contractually requires aggregate-level explainability for quarterly reviews.
- **Team expertise:** The ML team (4 engineers) has strong experience with gradient boosting frameworks (3+ years production experience with XGBoost/LightGBM) and moderate experience with deep learning (primarily for MDL-002 LSTM). No team member has production experience with reinforcement learning.

### Decision Drivers

- **DD-1: Inference latency** -- P95 < 200ms end-to-end, P95 < 150ms model inference. Non-negotiable for user experience.
- **DD-2: Route accuracy** -- Measured as the fraction of recommendations where actual travel time is within 15% of predicted optimal. Target: >= 0.92.
- **DD-3: Explainability for city authority** -- Ability to produce feature importance rankings and per-zone aggregate explanations for quarterly audits.
- **DD-4: Team expertise and maintainability** -- The team must be able to train, debug, monitor, and retrain the model independently without external consultants.
- **DD-5: Operational complexity** -- Retraining, deployment, and rollback should integrate with existing MLOps infrastructure (MLflow, Feast, Kubernetes).
- **DD-6: Fairness** -- Route quality must not systematically disadvantage specific city zones (disparity ratio < 1.15).

### Considered Alternatives

| Alternative | Type | Pros | Cons |
|------------|------|------|------|
| **Gradient Boosted Machine (LightGBM)** | Ensemble of decision trees | Fast inference (~8ms per prediction); inherently interpretable via SHAP TreeExplainer (exact, not approximate); team has 3+ years production experience; handles mixed feature types natively; low resource requirements; mature tooling (MLflow integration, ONNX export) | Limited ability to capture complex sequential dependencies; feature engineering burden (manual spatial-temporal features); performance ceiling on highly nonlinear route interactions |
| **Deep Reinforcement Learning (DRL)** | Policy gradient network (PPO/SAC) | Can learn complex multi-step routing policies; adapts to dynamic environments; potentially higher ceiling for multi-modal optimization; can jointly optimize individual and system-level objectives | Inference latency 300-800ms (exceeds budget); opaque decision-making (SHAP approximations unreliable for policy networks); requires simulation environment for training (6+ months to build); no team expertise; high variance in training; difficult to debug and monitor in production; regulatory risk for unexplainable decisions |
| **Linear Programming (LP) / Mixed Integer Programming** | Mathematical optimization | Guaranteed optimal solution for defined objective; fully transparent and auditable; deterministic (reproducible results); well-understood theoretical properties | Cannot incorporate nonlinear real-world factors (weather impact, demand elasticity) without linearization that degrades accuracy; solving time scales poorly with network size (>500ms for full city network); rigid -- requires manual reformulation when objectives change; poor accuracy on benchmark (0.78) due to linearization |

### Decision

**Chosen: Gradient Boosted Machine (LightGBM)**

LightGBM was selected because it is the only alternative that satisfies all decision drivers simultaneously:

1. **Latency (DD-1):** LightGBM inference averages 8ms with P95 at 12ms on the benchmark dataset (1,247 features after engineering), well within the 150ms inference budget. DRL exceeded the budget by 2-4x. LP exceeded it for the full city network.

2. **Accuracy (DD-2):** On the 6-month historical benchmark (2.3M route requests with ground-truth travel times), LightGBM achieved 0.93 accuracy. DRL achieved 0.91 after 4 weeks of training in a simplified simulation (full simulation not available). LP achieved 0.78 due to linearization compromises.

3. **Explainability (DD-3):** LightGBM supports exact SHAP TreeExplainer values, producing faithful per-prediction and aggregate feature importance without approximation. This directly supports the city authority's audit requirements. DRL's post-hoc explanations (gradient-based or LIME) were evaluated as insufficiently faithful (fidelity score 0.71 vs. 0.95 for SHAP TreeExplainer).

4. **Team expertise (DD-4):** The team has operated LightGBM models in production for 3 years across two prior projects. DRL would require hiring or significant upskilling (estimated 6-9 months ramp-up). LP is well-understood theoretically but the team lacks production optimization solver experience.

5. **Operational complexity (DD-5):** LightGBM integrates directly with the existing MLOps stack (MLflow tracking, Feast feature store, ONNX serving on Kubernetes). DRL would require a custom serving infrastructure and simulation environment.

The primary trade-off accepted is the manual feature engineering burden. LightGBM requires explicit spatial-temporal features (47 engineered features from raw data), whereas DRL could potentially learn these representations. This trade-off is mitigated by the Feature Store (Feast), which automates feature computation and versioning.

### AI-Specific Considerations

#### Dataset Characteristics

| Property | Value |
|----------|-------|
| Training dataset size | 2.3M route request-outcome pairs (6 months: Jan--Jun 2025) |
| Feature count | 47 engineered features (from 12 raw data sources) |
| Target variable | Optimal route index (multi-class: up to 8 candidate routes per request) |
| Class balance | Varies by origin-destination pair; median class imbalance ratio 1:3.2 |
| Data freshness | Features computed in real time (< 2 min staleness); training data refreshed monthly |
| Known biases | Underrepresentation of micro-mobility routes (bike-share, scooter) due to Operator C data quality issues (see DD-002 in AI Debt Register); suburban zones have 40% fewer historical route requests than central zones |
| Data sources | Traffic sensors (city), weather API (WeatherCorp Pro), transit schedules (3 operators), ridership data (3 operators), map data (OpenStreetMap), event calendar (city) |

#### Fairness and Bias Trade-offs

- **Protected attributes considered:** City zone (proxy for socioeconomic status), transport mode (equity of multi-modal access), time of day (night-shift commuters vs. peak-hour commuters)
- **Fairness metric chosen:** Zone service disparity ratio -- ratio of average route quality (travel time deviation from optimal) between worst-served and best-served zones
- **Accepted fairness-accuracy trade-off:** Zone-balanced sampling during training reduces overall accuracy by approximately 0.8 percentage points (from 0.938 to 0.930) but reduces zone disparity ratio from 1.24 to 1.11, below the 1.15 threshold. This trade-off was explicitly approved by the city transport authority.
- **Monitoring:** Weekly fairness monitoring (see QS-07 in AI Quality Scenarios); quarterly audit with city equity officer

#### Model Lifecycle

| Property | Value |
|----------|-------|
| Expected model lifetime | 1--3 months between retrains under normal conditions; shorter during infrastructure changes |
| Retraining trigger | Scheduled: every 30 days. Event-driven: accuracy < 0.90 for 3 consecutive daily checks, PSI > 0.2 for 3 consecutive hourly checks, or city infrastructure change notification |
| Retraining strategy | Full retrain on rolling 90-day window with 3x recency weighting on most recent 14 days; warm-start from previous model weights for infrastructure-triggered retrains |
| Training duration | ~3 hours on 8-core CPU (no GPU required) |
| Validation gates | Accuracy >= 0.92, P95 latency <= 180ms, zone disparity ratio < 1.15, all automated in CI pipeline |
| Rollback plan | Previous model version retained in MLflow registry; canary deployment with automatic rollback if accuracy < 0.88 or latency > 250ms during 24-hour canary window |
| Model versioning | Semantic versioning (major.minor); current: v2.1. Major version for architecture changes, minor for retrains |

#### Explainability

- **Explainability requirement:** Aggregate-level for quarterly city authority audits (global feature importance, per-zone breakdowns); per-decision available on request for complaint investigation
- **Chosen method:** SHAP TreeExplainer (exact Shapley values for tree ensembles)
- **Implementation:** Pre-computed global SHAP summary generated weekly; per-decision SHAP computed on-demand via explanation API (adds ~45ms latency, not on critical path)
- **Trade-offs:** SHAP TreeExplainer adds no latency to inference (post-hoc computation). The alternative (LIME) was considered but rejected due to lower fidelity (0.82 vs. 0.95) and higher computation time (~200ms vs. ~45ms)
- **Explanation format:** Top-5 feature contributions with direction and magnitude; zone-level aggregate reports as automated PDF

#### Regulatory Compliance

- **Applicable regulations:** EU AI Act (medium-risk classification); GDPR Art. 22 (automated decision-making -- route recommendations affect access to public services)
- **Documentation requirements met:**
  - Annex IV Art. 10 (Data governance): Training data documented in dataset card; bias analysis completed; zone-balanced sampling documented
  - Annex IV Art. 11 (Technical documentation): Model architecture, hyperparameters, evaluation metrics, and limitations documented in this ADR and model registry
  - Annex IV Art. 13 (Transparency): Explainability via SHAP; users informed that routes are AI-recommended via in-app disclosure
  - Annex IV Art. 15 (Accuracy, robustness): Accuracy monitoring, drift detection, degraded-mode operation, and rollback procedures documented in Operational AI View

### Consequences

- **Positive:**
  - Route recommendations served well within latency budget, enabling a responsive user experience
  - City authority can audit routing decisions via SHAP explanations, building institutional trust
  - Team can independently maintain, retrain, and debug the model using existing skills and infrastructure
  - Monthly retraining cadence is operationally feasible with existing compute budget (~$45/retrain on cloud CPU)
  - ONNX export enables serving on CPU-only Kubernetes pods, reducing infrastructure cost vs. GPU-dependent alternatives

- **Negative:**
  - Manual feature engineering creates ongoing maintenance burden (47 features across 6 pipeline stages); changes to raw data sources require feature pipeline updates
  - Performance ceiling: LightGBM may not capture complex multi-hop routing interactions as effectively as DRL could in theory; estimated 2-5% potential accuracy gap for complex multi-modal routes
  - The rejected DRL approach could have enabled joint optimization of individual routes and system-level load balancing, which now requires a separate heuristic layer on top of LightGBM predictions
  - Feature entanglement risk with MDL-002 through shared Feature Store features (see EN-001 in AI Debt Register)

- **Neutral:**
  - The choice of LightGBM over XGBoost was a close call (comparable accuracy, LightGBM ~15% faster training); switching between the two would be low-effort if needed
  - The 47-feature engineering pipeline, while a maintenance burden, also serves as institutional knowledge documentation of what factors drive routing quality in this city
