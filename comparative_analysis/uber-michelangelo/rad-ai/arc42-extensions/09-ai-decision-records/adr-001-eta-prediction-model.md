# AI Architecture Decision Record

> **Extends:** arc42 S9 -- Architecture Decisions
>
> Based on [MADR](https://adr.github.io/madr/) with AI-specific extensions.

---

## AI-ADR-001: ETA Prediction Model Architecture (DeepETA)

**Date:** 2022 (inferred from Uber Engineering Blog publication)
**Status:** Accepted
**Deciders:** Uber ML Platform Team, Maps and ETA Team (specific names not publicly documented)

### Context

Uber requires accurate travel time predictions for every trip across all products (Rides, UberEATS delivery, Freight). ETAs are displayed to riders before trip confirmation, used for driver dispatch decisions, and feed into pricing and matching models. The ETA system handles millions of prediction requests per second, making both accuracy and latency critical constraints.

The existing system used XGBoost gradient boosted trees to predict ETAs directly from features. While XGBoost provided reasonable accuracy, Uber identified that deep learning could potentially capture more complex spatial-temporal patterns, particularly non-linear interactions between features that tree models handle through explicit feature engineering.

Key constraints:
- **Latency:** ETA predictions must complete within 10ms P95 (including feature lookup from Cassandra). The model inference budget is approximately 5ms.
- **Scale:** The system serves millions of predictions per second globally across all Uber products.
- **Accuracy:** Measured by Mean Absolute Error (MAE) on actual vs. predicted trip duration. Every minute of MAE improvement has significant business impact across billions of trips.
- **Reliability:** ETA predictions are on the critical path for every trip and delivery. The system must maintain 99.99% availability.
- **Infrastructure:** The model must integrate with Michelangelo's existing serving infrastructure, Feature Store (Palette), and monitoring stack.

### Decision Drivers

- **DD-1: Prediction accuracy (MAE)** -- Lower MAE translates directly to better user experience and more efficient marketplace operations. Target: meaningful MAE improvement over XGBoost baseline.
- **DD-2: Inference latency** -- P95 < 5ms for model inference (excluding feature lookup). Non-negotiable for user-facing real-time serving.
- **DD-3: Training scalability** -- Must train on billions of trip records across all markets efficiently.
- **DD-4: Serving infrastructure compatibility** -- Must deploy on Michelangelo's existing Triton/Kubernetes serving infrastructure.
- **DD-5: Feature engineering burden** -- Reduce manual feature engineering effort where possible.
- **DD-6: Interpretability for debugging** -- ML engineers must be able to diagnose systematic ETA errors for specific markets or trip types.

### Considered Alternatives

| Alternative | Type | Pros | Cons |
|------------|------|------|------|
| **XGBoost (status quo)** | Gradient boosted trees | Fast inference (~2ms); well-understood; mature tooling; low operational complexity; inherently explainable via SHAP TreeExplainer | Performance ceiling: limited ability to capture high-order feature interactions; significant manual feature engineering burden (47+ handcrafted features); accuracy plateaued after extensive tuning |
| **Full Transformer (standard self-attention)** | Deep learning (attention mechanism) | Superior accuracy: self-attention captures pairwise feature interactions automatically; reduces feature engineering burden; can model complex spatial-temporal patterns | Inference latency too high: quadratic attention matrix computation (O(K^2)) exceeds latency budget at production feature dimensionality; GPU cost prohibitive at serving scale |
| **Linear Transformer with residual prediction** | Deep learning (kernel-based approximate attention) | Near-identical accuracy to full Transformer; O(K) attention approximation fits within latency budget; residual prediction over routing engine preserves physical routing accuracy; segment bias layer handles trip-type heterogeneity | More complex than XGBoost to debug; requires GPU serving infrastructure (Triton); black-box relative to tree models; new operational patterns for the team |
| **MLP (multi-layer perceptron)** | Deep learning (feedforward) | Simple architecture; fast inference; easy to serve | Accuracy between XGBoost and Transformer; no attention mechanism to capture feature interactions; still requires manual feature engineering |

### Decision

**Chosen: Linear Transformer with Residual Prediction (DeepETA)**

The Linear Transformer architecture was selected because it is the only alternative that meaningfully improves accuracy over XGBoost while satisfying the latency constraint:

1. **Accuracy (DD-1):** DeepETA achieved a significant MAE improvement over the XGBoost baseline (documented as a meaningful reduction in the Uber blog; exact numbers are internal). The improvement came from two sources: the self-attention mechanism automatically discovering feature interactions that required manual engineering in XGBoost, and the segment bias adjustment layer capturing trip-type-specific calibration.

2. **Latency (DD-2):** The linear transformer's kernel-based attention approximation reduces the quadratic O(K^2) cost of standard self-attention to O(K), achieving inference within the 5ms budget. The full transformer's quadratic cost made it infeasible at production feature dimensionality.

3. **Residual prediction design:** Rather than predicting ETA end-to-end, DeepETA predicts the residual (correction) to the routing engine's deterministic segment-sum estimate. This preserves the physical accuracy of map-based routing while allowing the ML model to learn corrections for factors the routing engine cannot model (driver behavior, time-of-day effects, weather impact). If the ML model fails, the routing engine provides a reasonable fallback.

4. **Feature processing innovation:** Bucketizing continuous features and embedding categorical features improved accuracy compared to using raw continuous values, as documented in the Uber blog. This approach also provides implicit feature normalization.

5. **Infrastructure compatibility (DD-4):** DeepETA deploys on Triton Inference Server within Michelangelo's Kubernetes infrastructure, compatible with existing monitoring and canary deployment patterns.

The primary trade-off accepted is increased operational complexity compared to XGBoost. Deep learning models are harder to debug, require GPU serving infrastructure, and have less transparent decision-making. This is mitigated by the segment bias layer (which provides trip-type-level interpretability) and by retaining XGBoost as a comparison baseline.

### AI-Specific Considerations

#### Dataset Characteristics

| Property | Value |
|----------|-------|
| Training dataset size | Billions of completed trip records (exact number internal) |
| Feature dimensionality | Hundreds of features across continuous, categorical, and calibration types |
| Target variable | Residual: actual trip duration minus routing engine estimate (minutes) |
| Geographic scope | All Uber markets globally |
| Data freshness | Features computed in real-time (online) and with daily refresh (offline) |
| Known biases | Data-sparse regions (rural, newly launched markets) have fewer training examples, leading to higher MAE in those regions; night-time trips are underrepresented relative to daytime |
| Data sources | GPS traces, routing engine estimates, trip completion records, real-time traffic, weather, map data |

#### Fairness and Bias Trade-offs

- **Protected attributes considered:** Geographic region (proxy for urban/rural, market maturity), trip type (delivery vs. ride), time of day (day vs. night)
- **Fairness concern:** ETA accuracy disparities across regions. Data-sparse regions receive less accurate ETAs, which translates to worse user experience and potentially unfair marketplace dynamics (if ETAs are used for pricing or matching).
- **Accepted trade-off:** The model optimizes for overall MAE without explicit fairness constraints. Region-specific calibration (via the segment bias layer) partially addresses geographic disparities, but a systematic fairness evaluation framework would be needed to fully ensure equitable accuracy.
- **Monitoring:** Per-market MAE is tracked continuously. Markets where MAE degrades disproportionately are flagged for investigation.

#### Model Lifecycle

| Property | Value |
|----------|-------|
| Expected model lifetime | Weeks to months between retrains; varies by market |
| Retraining trigger | Scheduled (periodic); event-driven (road network changes, major infrastructure updates, accuracy degradation detected by monitoring) |
| Retraining strategy | Full retrain on recent data window with global model; market-specific fine-tuning where needed |
| Training duration | Hours on GPU cluster (distributed training via Horovod) |
| Validation gates | MAE improvement over production baseline; latency within budget; no market-level regression; canary analysis passed |
| Rollback plan | Previous model version retained in Gallery; automated rollback if canary metrics degrade; routing engine serves as ultimate fallback (no ML) |
| Model versioning | Managed through Gallery lifecycle system |

#### Explainability

- **Explainability requirement:** Debugging capability for ML engineers investigating systematic ETA errors in specific markets or trip types. Not required for user-facing explanations.
- **Chosen method:** Segment bias layer provides trip-type-level decomposition. Per-prediction attention weight analysis available for debugging but not computed in production (adds latency).
- **Trade-offs:** The linear transformer is less interpretable than XGBoost (where SHAP TreeExplainer provides exact per-feature attribution). This was accepted because the primary explainability need is debugging, not per-user explanation.
- **Explanation format:** Market-level MAE decomposition by trip type, time of day, and route characteristics. Per-prediction debugging via offline attention analysis when investigating specific errors.

#### Regulatory Compliance

- **Applicable regulations:** Consumer protection laws in various jurisdictions requiring accurate fare estimates (which depend on ETA accuracy). No specific AI regulation mandating ETA model documentation at time of decision, but proactive documentation aligns with emerging AI governance frameworks.
- **Documentation requirements addressed:**
  - Model architecture and training methodology documented in this ADR
  - Performance metrics tracked and logged via Michelangelo monitoring
  - Fallback behavior documented (routing engine as deterministic baseline)
  - Market-level accuracy monitoring provides evidence of equitable service

### Consequences

- **Positive:**
  - Meaningful MAE improvement over XGBoost baseline, improving user experience across billions of trips
  - Reduced feature engineering burden: model learns feature interactions that previously required manual engineering
  - Residual prediction design provides graceful fallback to routing engine
  - Segment bias layer provides trip-type-level calibration, improving accuracy for heterogeneous trip populations
  - Architecture is extensible: new feature types can be added without redesigning the interaction structure

- **Negative:**
  - Increased operational complexity: GPU serving infrastructure (Triton), more complex model debugging, new failure modes (GPU memory, model loading latency)
  - Reduced per-prediction interpretability compared to XGBoost SHAP
  - Higher serving infrastructure cost (GPU vs. CPU for XGBoost)
  - Training requires GPU cluster (Horovod), increasing training infrastructure dependency
  - Team needed to build expertise in transformer architectures and GPU serving

- **Neutral:**
  - XGBoost remains available as a comparison baseline and potential fallback model
  - The linear transformer approximation introduces a small accuracy gap vs. full transformer, but this is negligible compared to the latency savings
  - Market-specific fine-tuning may be needed for newly launched or data-sparse markets
