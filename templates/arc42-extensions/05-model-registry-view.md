# Model Registry View

> **Extends:** arc42 §5 — Building Block View

## Purpose

Document AI models as first-class architectural building blocks, with versioning metadata, lineage information, and deployment history. Models evolve through retraining, not code commits — this view captures that distinct lifecycle.

## Model Inventory

<!-- List all ML models in the system as architectural building blocks -->

| Model ID | Name | Task | Framework | Current Version | Status |
|----------|------|------|-----------|----------------|--------|
| *[e.g., MDL-001]* | *[Name]* | *[e.g., Classification, Regression, NER]* | *[e.g., PyTorch, TensorFlow, scikit-learn]* | *[e.g., v2.3.1]* | *[Production / Staging / Deprecated]* |

## Model Details

<!-- Repeat this section for each model -->

### [Model Name] (MDL-XXX)

**Purpose:** *[What architectural role does this model serve?]*

**Versioning:**

| Version | Training Date | Dataset Version | Key Metrics | Deployment Date | Notes |
|---------|--------------|-----------------|-------------|----------------|-------|
| *[v1.0]* | *[YYYY-MM-DD]* | *[DS-XXX v1]* | *[e.g., F1: 0.92, AUC: 0.95]* | *[YYYY-MM-DD]* | *[Initial release]* |

**Hyperparameters:**

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| *[e.g., learning_rate]* | *[e.g., 0.001]* | *[e.g., Grid search optimum]* |

**Training Data Lineage:**

| Dataset | Source | Records | Features | Refresh Cadence |
|---------|--------|---------|----------|----------------|
| *[Name]* | *[e.g., internal DB, vendor API]* | *[Count]* | *[Count]* | *[e.g., daily, weekly]* |

**Dependencies:**

- Upstream: *[What feeds into this model — data sources, feature stores, other models]*
- Downstream: *[What consumes this model's outputs — services, dashboards, other models]*

**Performance Baselines:**

| Metric | Baseline | Current | Alert Threshold |
|--------|----------|---------|----------------|
| *[e.g., Accuracy]* | *[e.g., 0.90]* | *[e.g., 0.93]* | *[e.g., < 0.85]* |

## Integration with Model Registry Tools

<!-- Describe how this documentation connects to your model registry (if any) -->

| Tool | Purpose | Link |
|------|---------|------|
| *[e.g., MLflow, Weights & Biases, DVC]* | *[Model tracking / experiment tracking / data versioning]* | *[URL]* |
