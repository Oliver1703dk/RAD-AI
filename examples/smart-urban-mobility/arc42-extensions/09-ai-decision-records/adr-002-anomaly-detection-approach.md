# AI Architecture Decision Record

## AI-ADR-002: Anomaly Detection Approach for Safety-Critical Transport Monitoring

**Date:** 2025-09-15
**Status:** Accepted
**Deciders:** Lead ML Engineer, Safety Systems Architect, City Transport Authority Safety Officer, Data Protection Officer

### Context

The SUM platform requires real-time detection of safety-relevant anomalies in urban transport infrastructure, including infrastructure failures (sensor outages, signal malfunctions), crowd surges at transit stations, and hazardous traffic conditions. This component (MDL-003) is classified as **high-risk under EU AI Act Annex III** because it serves as a safety component of regulated transport infrastructure.

The high-risk classification imposes specific requirements:

- Per-decision explainability (Annex IV, Article 13): each anomaly flag must be accompanied by a human-understandable explanation of why it was triggered.
- Human oversight (Article 14): operators must be able to understand, interpret, and override any anomaly decision.
- Technical documentation (Annex IV): the detection methodology must be documented in sufficient detail for conformity assessment.
- Accuracy, robustness, and cybersecurity (Article 15): the system must maintain documented performance levels under adversarial conditions.

The system processes approximately 80,000 sensor events per minute from 2,400 intersections, with a maximum acceptable detection latency of 500ms per event. The city transport authority requires that safety officers -- who are domain experts but not ML practitioners -- can understand why any given anomaly was flagged.

### Decision Drivers

1. **Interpretability (critical):** EU AI Act Annex III high-risk classification requires per-decision explanations understandable by non-technical city authority safety officers. This is a hard regulatory constraint, not a preference.
2. **Low false negative rate (critical):** Missed safety anomalies can endanger lives. Recall >= 0.90 is a hard requirement from the city transport authority.
3. **Explainability to non-technical stakeholders:** Safety officers must be able to review flagged anomalies and understand, in domain terms, why the system flagged them (e.g., "speed on segment X dropped 80% while adjacent segments are normal" rather than "feature 37 exceeded isolation threshold").
4. **Real-time latency:** Detection must occur within 500ms of sensor event ingestion. Streaming inference is required; batch-only approaches are not viable.
5. **Robustness to novel anomalies:** The system must detect previously unseen anomaly types, not only patterns present in historical data.
6. **Operational maintainability:** The safety team needs to add, modify, and audit detection rules without retraining the ML model.
7. **Graceful degradation:** The system must have a viable fallback when the ML component is unavailable.

### Considered Alternatives

| Alternative | Type | Pros | Cons |
|------------|------|------|------|
| **Isolation Forest + domain rules ensemble (chosen)** | Unsupervised anomaly detection + rule-based post-processing | Inherently interpretable (isolation depth maps to anomaly score); detects novel anomaly types without labeled data; rules layer adds domain-specific precision and auditability; fast inference (<50ms); rules can be modified independently of model | Lower precision than supervised approaches on known anomaly types; requires careful threshold tuning; ensemble logic adds architectural complexity |
| **Autoencoder-based anomaly detection** | Deep learning (reconstruction error) | Learns complex multivariate patterns; strong performance on high-dimensional sensor data; can capture temporal dependencies with LSTM-autoencoder variant | Black-box: reconstruction error is not directly interpretable to non-technical users; explaining *why* reconstruction failed requires additional post-hoc methods (SHAP/LIME) that add latency; EU AI Act conformity assessment is harder for neural approaches; retraining is expensive |
| **Supervised classification** | Gradient boosted classifier on labeled anomaly types | Highest precision and recall on known anomaly types; well-understood ML approach; SHAP explanations available | Requires extensive labeled anomaly data (scarce -- only ~200 confirmed anomalies in 18 months of operation); cannot detect novel anomaly types not in training data; label imbalance (>99.9% normal) causes training instability; model must be retrained when new anomaly types emerge |

### Decision

**Chosen: Isolation Forest + domain rules ensemble.**

The decision combines a statistical unsupervised model (Isolation Forest) with a curated rule layer to achieve interpretability, novel anomaly detection, and operational flexibility:

**Isolation Forest layer (ML):**
- Trained on normal operating patterns (no labeled anomalies required).
- Produces an anomaly score (0-1) based on isolation depth -- data points that are easier to isolate are more anomalous.
- The anomaly score directly maps to an intuitive explanation: "this combination of sensor readings is unusual because it required only N splits to isolate, compared to the typical M splits."
- Runs on streaming features with <50ms inference latency.

**Domain rules layer (deterministic):**
- 23 hand-crafted rules encoding known safety patterns (e.g., "speed on highway segment < 10 km/h for > 5 min while upstream segments are flowing normally" -> potential incident).
- Rules are maintained by the safety team in a version-controlled YAML configuration, independently of the ML model.
- Each rule has a human-readable description, severity level, and recommended action.
- Rules can override the ML model: a triggered rule always produces an alert regardless of the Isolation Forest score.

**Ensemble logic:**
- An event is flagged as anomalous if: (a) Isolation Forest score > threshold (0.65), OR (b) any domain rule fires.
- Severity is determined by: max(IF-score-based severity, rule-based severity).
- The explanation combines both sources: "Anomaly detected: [IF explanation] + [rule explanation if applicable]."
- This ensures that novel anomalies (detected by IF but not matching any rule) are still caught, while known critical patterns (rules) are never missed.

**Why not the autoencoder:**
The autoencoder was the strongest alternative on pure detection performance (tested offline: recall 0.94 vs. IF's 0.91). However, explaining reconstruction errors to non-technical safety officers proved impractical in a user study with 5 city authority reviewers. Statements like "reconstruction error on dimensions 12, 37, 89 exceeded threshold" were not actionable. Adding SHAP post-hoc explanations increased inference latency to ~400ms (close to the 500ms budget) and still produced feature-importance explanations rather than domain-language explanations. Given the EU AI Act requirement for understandable explanations (Article 13(1)), the interpretability gap was a disqualifying factor.

**Why not supervised classification:**
The labeled anomaly dataset contains only 207 confirmed events over 18 months of operation. This is insufficient for training a robust classifier, especially given the extreme class imbalance (0.003% positive rate). More critically, a supervised approach cannot detect novel anomaly types -- it can only classify patterns seen during training. The city transport authority explicitly requires detection of "unknown unknowns" based on their experience with a previous rule-only system that missed a novel crowd surge pattern in 2024.

### AI-Specific Considerations

#### Dataset Characteristics

| Property | Value |
|----------|-------|
| Training dataset size | 14.2M normal sensor event windows (6 months of operation) |
| Feature count | 51 features per event window |
| Class balance | Unsupervised -- trained on normal data only. 207 labeled anomalies used for threshold calibration only |
| Data freshness | Model trained on data up to 30 days old; streaming features are real-time |
| Known biases | Underrepresentation of weekend/holiday patterns in initial training window; mitigated by ensuring training data spans full calendar cycles. Sensor coverage varies by neighborhood (central: 95%, peripheral: 72%) |

#### Fairness and Bias Trade-offs

- **Protected attributes considered:** Geographic equity -- anomaly detection sensitivity must not systematically vary by neighborhood socioeconomic status.
- **Fairness metric chosen:** Equal detection rate across city zones (within 5% relative difference).
- **Accepted trade-off:** Zone-stratified threshold calibration reduces overall precision by ~1% but ensures equitable monitoring coverage. Peripheral neighborhoods with lower sensor density use adjusted thresholds to compensate for sparser data.
- **Monitoring:** Monthly fairness audit comparing per-zone detection rates and false positive rates, reported to city transport authority.

#### Model Lifecycle

| Property | Value |
|----------|-------|
| Expected model lifetime | 6 months before scheduled retrain |
| Retraining trigger | (1) Scheduled: every 6 months. (2) Drift: PSI > 0.25 on input features for > 48 hours. (3) Performance: precision drops below 0.90 or recall drops below 0.88 on rolling 30-day labeled evaluation |
| Retraining strategy | Full retrain on most recent 6 months of normal data. Rules layer is updated independently via YAML configuration |
| Rollback plan | Previous model version retained in MLflow registry; rollback via feature flag in <5 min. If rollback also fails, Manual Safety Protocol (FB3) activates: all anomaly detection reverts to rule-only mode |
| A/B testing | Shadow mode for new model version: runs in parallel with production model for 2 weeks, outputs compared but not acted upon, before promotion |

#### Explainability

- **Explainability requirement:** Per-decision for all HIGH and CRITICAL severity anomalies (EU AI Act Article 13 for high-risk systems). Aggregate explanations for LOW/MEDIUM severity.
- **Chosen method:** Native Isolation Forest path-based explanation + domain rule descriptions.
  - For Isolation Forest: the features that cause early isolation are identified and translated to domain language via a feature-to-description mapping (e.g., feature `segment_speed_zscore` -> "speed on [segment name] was [X] standard deviations below normal").
  - For domain rules: each rule has a pre-written human-readable explanation template with parameter slots.
- **Trade-offs:** Native IF explanations are less precise than SHAP (they identify contributing features but not exact contribution magnitudes). This was accepted because domain-language explanations ("speed dropped unusually on Main Street") are more actionable for safety officers than numerical feature importances.
- **Explanation latency:** <10ms additional (feature-to-description lookup), well within the 500ms budget.

#### Regulatory Compliance

- **Applicable regulations:** EU AI Act Annex III (high-risk: safety component of transport infrastructure), GDPR Article 22 (automated decision-making).
- **Documentation requirements met:**
  - Annex IV(2)(b): Description of the elements of the AI system and of the process for its development -- covered by this ADR and the Model Registry View.
  - Annex IV(2)(c): Detailed description of the elements and process of monitoring, functioning and control -- covered by Drift Monitor documentation and this ADR's lifecycle section.
  - Annex IV(2)(d): Description of the logic involved -- Isolation Forest algorithm documented, rule set maintained in version control, ensemble logic specified above.
  - Annex IV(2)(e): Description of the validation and testing procedures -- A/B testing protocol, threshold calibration on labeled data, fairness audit.
  - Annex IV(2)(g): Detailed description of the human oversight measures -- Safety Review Queue (HITL), confidence gate CG-3, fallback to Manual Safety Protocol.
  - Article 13 (Transparency): Per-decision explanations in domain language provided for all HIGH+ severity anomalies.
  - Article 14 (Human oversight): Safety officers can override any anomaly decision; all overrides are logged.

### Consequences

- **Positive:**
  - Regulatory compliance is achievable: native interpretability of Isolation Forest + rule descriptions satisfies EU AI Act Article 13 without post-hoc explanation methods.
  - Novel anomaly detection: unsupervised approach catches anomaly types not in historical data.
  - Operational independence: safety team can add/modify rules without ML team involvement or model retraining.
  - Fast inference: <50ms for IF + <10ms for rules + <10ms for explanation = well within 500ms budget.
  - Clean fallback: rule layer alone serves as a functional (if noisier) safety net when the ML component is unavailable.

- **Negative:**
  - Lower precision on known anomaly types compared to a supervised classifier (~0.96 vs. estimated ~0.98 for supervised, based on offline evaluation with limited labels).
  - Ensemble logic adds architectural complexity: two detection paths must be maintained, tested, and monitored.
  - Threshold tuning requires domain expertise and periodic recalibration (quarterly review with safety officers).
  - Rule maintenance is manual -- the 23-rule set must grow as new anomaly patterns are discovered, creating a maintenance burden.

- **Neutral:**
  - The team must maintain expertise in both unsupervised anomaly detection (ML team) and domain rule authoring (safety team). This dual-competency requirement is appropriate given the safety-critical nature of the component.
  - The 207 labeled anomalies, while insufficient for supervised training, remain valuable for threshold calibration and evaluation. The team continues to label confirmed anomalies to build this dataset for potential future approaches.
