# Responsible AI Concepts

> **Extends:** arc42 §8 — Cross-Cutting Concepts

## Purpose

Document the cross-cutting responsible AI concerns that affect all AI components: fairness, explainability, transparency, human oversight, and privacy. These are first-class architectural quality attributes for AI systems that have no home in standard arc42.

## Fairness

### Fairness Metrics

| Model | Protected Attributes | Metric | Target | Current | Measurement Frequency |
|-------|---------------------|--------|--------|---------|----------------------|
| *[Model name]* | *[e.g., gender, age, ethnicity]* | *[e.g., Demographic Parity, Equalized Odds]* | *[e.g., gap < 0.05]* | *[e.g., 0.03]* | *[e.g., per retrain, weekly]* |

### Bias Detection Architecture

<!-- Describe how bias is detected and mitigated -->

- **Pre-training:** *[e.g., dataset bias audits, representation analysis]*
- **In-training:** *[e.g., fairness-constrained optimization, adversarial debiasing]*
- **Post-training:** *[e.g., threshold calibration, output auditing]*
- **In-production:** *[e.g., continuous fairness monitoring, slice-based analysis]*

## Explainability

### Explainability Requirements

| Model | Explanation Scope | Method | Audience | Trigger |
|-------|------------------|--------|----------|---------|
| *[Model name]* | *[Per-decision / Aggregate / On-demand]* | *[e.g., SHAP, LIME, attention weights, feature importance]* | *[e.g., End user, regulator, internal auditor]* | *[e.g., Every prediction, on appeal, quarterly report]* |

### Explainability Integration Points

<!-- Describe where explainability APIs/services are integrated in the architecture -->

| Component | Explainability Service | API Surface | Latency Budget |
|-----------|----------------------|-------------|---------------|
| *[Component name]* | *[e.g., SHAP server, LIME endpoint]* | *[e.g., REST /explain]* | *[e.g., < 500ms]* |

## Human Oversight

### Oversight Mechanisms

| Mechanism | Scope | Trigger | Authority | Response SLA |
|-----------|-------|---------|-----------|-------------|
| *[e.g., Human-in-the-loop review]* | *[e.g., low-confidence predictions]* | *[e.g., confidence < 0.7]* | *[e.g., domain expert]* | *[e.g., < 1 hour]* |

### Override and Escalation

<!-- Describe how human overrides are handled -->

- **Override mechanism:** *[How can a human override an AI decision?]*
- **Override logging:** *[How are overrides tracked?]*
- **Escalation path:** *[What happens when the AI system flags uncertainty?]*

## Transparency

### Disclosure Requirements

| Context | Disclosure | Format | Regulation |
|---------|-----------|--------|------------|
| *[e.g., End-user interaction]* | *[e.g., "This recommendation is AI-generated"]* | *[e.g., UI label, API header]* | *[e.g., EU AI Act Art. 52]* |

## Privacy

### Data Protection

| Data Category | Classification | Handling | Retention | Legal Basis |
|---------------|---------------|----------|-----------|-------------|
| *[e.g., Training data]* | *[e.g., Personal / Sensitive / Anonymized]* | *[e.g., encrypted at rest, anonymized before training]* | *[e.g., 2 years]* | *[e.g., GDPR Art. 6(1)(f)]* |
