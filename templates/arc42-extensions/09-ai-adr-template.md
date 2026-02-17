# AI Architecture Decision Record (AI-ADR) Template

> **Extends:** arc42 §9 — Architecture Decisions
>
> Based on [MADR](https://adr.github.io/madr/) with AI-specific extensions.

---

## AI-ADR-XXX: [Title]

**Date:** YYYY-MM-DD
**Status:** [Proposed | Accepted | Deprecated | Superseded by AI-ADR-YYY]
**Deciders:** [List of people involved in the decision]

### Context

*[What is the issue that motivates this decision? What are the system constraints?]*

### Decision Drivers

- *[Driver 1: e.g., accuracy requirements]*
- *[Driver 2: e.g., latency constraints]*
- *[Driver 3: e.g., regulatory compliance]*
- *[Driver 4: e.g., team expertise]*

### Considered Alternatives

| Alternative | Type | Pros | Cons |
|------------|------|------|------|
| *[Option A]* | *[e.g., Random Forest]* | *[Pros]* | *[Cons]* |
| *[Option B]* | *[e.g., Neural Network]* | *[Pros]* | *[Cons]* |
| *[Option C]* | *[e.g., Rule-based baseline]* | *[Pros]* | *[Cons]* |

### Decision

*[Which option was chosen and why?]*

### AI-Specific Considerations

#### Dataset Characteristics

| Property | Value |
|----------|-------|
| Training dataset size | *[e.g., 500K records]* |
| Feature count | *[e.g., 42]* |
| Class balance | *[e.g., 70/30 split]* |
| Data freshness | *[e.g., updated daily]* |
| Known biases | *[e.g., underrepresentation of X]* |

#### Fairness and Bias Trade-offs

- Protected attributes considered: *[List]*
- Fairness metric chosen: *[e.g., Equalized Odds]*
- Accepted fairness-accuracy trade-off: *[e.g., 2% accuracy reduction for demographic parity gap < 0.05]*

#### Model Lifecycle

| Property | Value |
|----------|-------|
| Expected model lifetime | *[e.g., 3–6 months before retrain]* |
| Retraining trigger | *[e.g., accuracy drops below 0.85, monthly schedule, data drift detected]* |
| Retraining strategy | *[e.g., full retrain, incremental, fine-tune]* |
| Rollback plan | *[e.g., previous version in model registry]* |

#### Explainability

- Explainability requirement: *[e.g., per-decision for high-risk, aggregate for low-risk]*
- Chosen method: *[e.g., SHAP values]*
- Trade-offs: *[e.g., SHAP adds 200ms latency vs. LIME's lower fidelity]*

#### Regulatory Compliance

- Applicable regulations: *[e.g., EU AI Act high-risk, GDPR Art. 22]*
- Documentation requirements met: *[List specific Annex IV items addressed]*

### Consequences

- **Positive:** *[What becomes easier?]*
- **Negative:** *[What becomes harder? What risks are accepted?]*
- **Neutral:** *[What changes but neither improves nor worsens?]*
