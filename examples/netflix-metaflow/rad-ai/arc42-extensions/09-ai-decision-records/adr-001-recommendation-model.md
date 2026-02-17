# AI Architecture Decision Record

> **Extends:** arc42 S9 -- Architecture Decisions
>
> Based on [MADR](https://adr.github.io/madr/) with AI-specific extensions.

---

## AI-ADR-001: Recommendation Model Architecture Consolidation

**Date:** 2025-01-15
**Status:** Accepted
**Deciders:** ML Platform leadership, Recommendation team leads, Product leadership (names not disclosed; decision documented in Netflix's public research)

### Context

Netflix's recommendation system historically consisted of many specialized models, each responsible for a specific recommendation surface (e.g., one model for "Because You Watched" rows, another for "Top Picks," another for genre rows). As the number of recommendation surfaces grew, this multi-model architecture created significant operational complexity: each model had its own training pipeline, serving infrastructure, monitoring, and retraining schedule. Changes to shared features required coordinated updates across many models.

Netflix publicly described this consolidation in their 2024 research paper "Lessons Learnt From Consolidating ML Models in a Large Scale Recommendation System," which motivates this ADR.

Key constraints:
- **Scale:** 200M+ subscribers across 190+ countries, with heterogeneous content preferences and viewing behaviors.
- **Latency:** Homepage rendering must complete within 250ms (P95), including multiple recommendation row computations.
- **Diversity:** Recommendations must balance relevance (engagement optimization) with content diversity (filter bubble mitigation) and creator fairness (exposure for new and niche content).
- **Operational complexity:** Multiple specialized models create an O(N) maintenance burden. Each model requires independent training data, training pipeline, evaluation, deployment, and monitoring.
- **Feature entanglement:** Shared features across models create implicit coupling. A change to one feature (e.g., how "recently viewed" is computed) silently affects all models consuming that feature.
- **Team expertise:** The ML team has deep experience with both traditional ensemble methods and deep learning. Multi-task learning is a proven technique at Netflix scale.

### Decision Drivers

- **DD-1: Operational simplicity** -- Reduce the number of independent models from dozens to a consolidated architecture, reducing training, deployment, and monitoring overhead.
- **DD-2: Feature consistency** -- A single model consumes features through a unified interface, eliminating feature version skew across models.
- **DD-3: Recommendation quality** -- Multi-task learning can share representations across tasks, improving quality on surfaces with sparse training signal.
- **DD-4: Diversity control** -- A single model can jointly optimize for engagement and diversity, rather than applying diversity heuristics post-hoc across independently optimized models.
- **DD-5: Latency** -- Consolidated inference reduces total serving latency by sharing computation across recommendation surfaces.
- **DD-6: Experimentation velocity** -- Fewer models means fewer coordination points for A/B tests; experiments can test changes across all surfaces simultaneously.

### Considered Alternatives

| Alternative | Type | Pros | Cons |
|------------|------|------|------|
| **Multi-model ensemble (status quo)** | Many specialized models per recommendation surface | Each model can be independently optimized; failure isolation (one model failing does not affect others); simpler individual models | O(N) operational complexity; feature entanglement across models; inconsistent recommendations across surfaces; difficult to jointly optimize diversity; slow experimentation |
| **Consolidated multi-task model** | Single model producing outputs for multiple recommendation tasks | Shared representations improve data efficiency; unified training and serving; consistent feature consumption; joint diversity optimization; faster experimentation | Single point of failure; more complex training; requires careful task weighting; larger model size; risk of task interference |
| **Two-stage architecture (retrieval + ranking)** | Separate retrieval model (broad candidate generation) and ranking model (fine-grained scoring) | Clean separation of concerns; retrieval can be approximate (fast); ranking can be sophisticated; scales well with catalog size | Does not address the core problem of multiple specialized ranking models per surface; adds complexity at the retrieval stage |

### Decision

**Chosen: Consolidated multi-task model**

The multi-model ensemble was replaced with a consolidated multi-task architecture where a single model generates recommendations for multiple surfaces simultaneously. This decision was driven primarily by the unsustainable O(N) operational complexity of the multi-model approach and the demonstrated quality improvements from shared representations across recommendation tasks.

Netflix's research found that the consolidated model not only simplified the system architecture but also improved model performance. Shared representations allowed the model to transfer learning from data-rich surfaces (e.g., homepage) to data-sparse surfaces (e.g., notification targeting), improving quality across the board.

The primary trade-off accepted is the single-point-of-failure risk: if the consolidated model fails, all recommendation surfaces are affected simultaneously. This is mitigated by robust fallback strategies (popularity-based recommendations) and the reduced deployment risk from having a single, well-tested deployment pipeline rather than many independent ones.

### AI-Specific Considerations

#### Dataset Characteristics

| Property | Value |
|----------|-------|
| Training data scale | Billions of interaction events across all recommendation surfaces |
| Feature count | ~1,000+ features (member, content, contextual, behavioral, temporal) |
| Task count | Multiple recommendation surfaces (homepage rows, search, notifications) treated as separate tasks sharing a backbone |
| Data freshness | Features computed in real-time (Flink streaming) and batch (daily Spark jobs); training on rolling historical windows |
| Known biases | Popularity bias in implicit feedback (popular content receives disproportionate positive signal); position bias (items shown first receive more engagement regardless of true relevance); geographic bias (more training signal from larger markets) |
| Data sources | Member interaction streams (Kafka), content catalog (CMS), member profiles (aggregated from interaction history), device telemetry |

#### Fairness and Bias Trade-offs

- **Protected attributes considered:** Geographic region (proxy for language and cultural preferences), member activity level (heavy vs. light users), content origin country (local vs. international content).
- **Fairness metric chosen:** Content diversity (genre entropy per recommendation set) and geographic equity (cross-region engagement variance).
- **Accepted fairness-accuracy trade-off:** Contextual bandit exploration (5--10% of recommendations) reduces short-term engagement metrics but maintains long-term content diversity and prevents filter bubble convergence. Netflix has publicly stated that diverse recommendations correlate with better long-term retention.
- **Monitoring:** Guardrail metrics in every A/B test ensure diversity metrics do not degrade when new model versions are evaluated.

#### Model Lifecycle

| Property | Value |
|----------|-------|
| Expected model lifetime | Weeks to months between retrains, depending on observed performance and distribution shifts |
| Retraining trigger | Scheduled (regular cadence); event-driven (significant catalog changes, seasonal shifts); drift-detected (engagement metric degradation beyond threshold) |
| Retraining strategy | Full retrain on updated historical window; multi-task loss rebalancing based on per-surface performance |
| Training infrastructure | Distributed training on GPU clusters via PyTorch + Horovod; managed by Metaflow workflows, orchestrated by Maestro |
| Validation gates | Offline metrics (recall, NDCG, diversity scores) must meet per-surface thresholds; online A/B test required before full deployment |
| Rollback plan | Previous model version always retained; Spinnaker enables rapid rollback; Kayenta canary analysis catches regressions before full rollout |
| Model versioning | Managed by Metaflow artifact tracking; every training run produces a versioned model with full lineage |

#### Explainability

- **Explainability requirement:** Row-level (each recommendation row has an interpretable title explaining why those items were selected); aggregate-level (A/B test analysis decomposes model impact by content type, region, and member segment).
- **Chosen method:** Explanation by design -- the recommendation UI structure (named rows like "Because You Watched X") serves as the explanation mechanism, rather than post-hoc explanation of opaque model outputs.
- **Trade-offs:** This approach sacrifices fine-grained per-item explanations (which would require SHAP or similar post-hoc methods at serving latency) in favor of interpretable row-level explanations that are computationally free.

#### Regulatory Compliance

- **Applicable regulations:** EU Digital Services Act (DSA) Art. 27 (recommender system transparency for very large online platforms); GDPR (data processing for personalization); California CCPA (opt-out rights for data usage in recommendations).
- **Documentation requirements met:**
  - DSA Art. 27: Netflix's Help Center provides information about recommendation parameters and options to modify recommendations. The "explanation by design" approach (named recommendation rows) provides the "main parameters" disclosure required by DSA.
  - GDPR Art. 22: Recommendation is not "solely automated decision-making" with legal or significant effects. Members can influence recommendations via explicit controls (thumbs up/down, hide, profile preferences).
  - Privacy: Training data is derived from member interactions under the service agreement. Members can delete viewing history, which propagates to future training datasets.

### Consequences

- **Positive:**
  - Dramatically reduced operational complexity: one model to train, deploy, and monitor instead of dozens.
  - Improved recommendation quality through shared representations, particularly for data-sparse surfaces.
  - Consistent feature consumption eliminates feature version skew.
  - Joint optimization of engagement and diversity in a single training objective.
  - Faster experimentation: A/B tests evaluate changes across all surfaces simultaneously.

- **Negative:**
  - Single point of failure: consolidated model outage affects all recommendation surfaces simultaneously (mitigated by robust fallbacks).
  - More complex training: multi-task loss balancing requires careful tuning to prevent task interference.
  - Larger model size increases training cost and serving resource requirements.
  - Debugging is harder: a quality regression on one surface may be caused by task interference from another surface's data.

- **Neutral:**
  - The consolidated architecture is consistent with industry trends (Google, Meta have published similar consolidation efforts), suggesting this is a mature architectural pattern for large-scale recommendation systems.
  - The decision does not preclude adding specialized models for specific surfaces if needed in the future; the consolidated model can coexist with specialized models for edge cases.
