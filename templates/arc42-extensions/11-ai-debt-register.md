# AI Debt Register

> **Extends:** arc42 §11 — Risks & Technical Debt

## Purpose

Structured tracking of ML-specific technical debt categories, based on Sculley et al.'s taxonomy. AI systems accumulate debt in ways that traditional technical debt tracking misses — through data dependencies, feedback loops, and configuration complexity.

## Debt Categories

### Boundary Erosion

*When the boundaries between ML components and the rest of the system become unclear, making changes risky.*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[BE-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

### Entanglement

*When changing one model's input features or behavior unexpectedly affects other models (CACE: Changing Anything Changes Everything).*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[EN-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

### Hidden Feedback Loops

*When model outputs indirectly influence future training data, creating self-reinforcing biases.*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[FL-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

### Data Dependency Debt

*Unstable, underutilized, or poorly understood data dependencies.*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[DD-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

### Pipeline Debt

*Complexity and fragility in data pipelines, feature engineering, and preprocessing logic.*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[PD-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

### Configuration Debt

*Sprawling configuration (hyperparameters, feature flags, thresholds) that is hard to manage, test, and audit.*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[CD-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

### Model Staleness

*Models that have not been retrained despite data distribution changes.*

| ID | Description | Severity | Affected Components | Mitigation Plan | Owner | Status |
|----|-------------|----------|--------------------|--------------------|-------|--------|
| *[MS-001]* | *[Description]* | *[High/Medium/Low]* | *[Components]* | *[Plan]* | *[Owner]* | *[Open/Mitigated/Accepted]* |

## Debt Summary Dashboard

| Category | Open | Mitigated | Accepted | Total |
|----------|------|-----------|----------|-------|
| Boundary Erosion | | | | |
| Entanglement | | | | |
| Hidden Feedback Loops | | | | |
| Data Dependency Debt | | | | |
| Pipeline Debt | | | | |
| Configuration Debt | | | | |
| Model Staleness | | | | |
| **Total** | | | | |

## Review Cadence

- **Debt review frequency:** *[e.g., monthly, per sprint]*
- **Debt owner:** *[Team/role responsible for tracking]*
- **Escalation threshold:** *[e.g., any High severity open > 30 days]*
