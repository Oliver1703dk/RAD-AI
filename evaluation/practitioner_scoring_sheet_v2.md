# RAD-AI: Practitioner Compliance Scoring Sheet

## Background

We are evaluating whether established architecture documentation frameworks (arc42 and the C4 model) can address the documentation requirements of the EU AI Act Annex IV, and whether proposed extensions (RAD-AI) improve coverage.

**Your task:** Score how well each framework configuration can address each EU AI Act Annex IV requirement category, using the scale below.

**Time estimate:** 15-20 minutes.

---

## Scoring Scale

| Score | Label | Definition |
|-------|-------|------------|
| **0** | Not addressable | No section or artifact in the framework covers this concern. A practitioner using the framework would have no designated place to document this. |
| **1** | Partial | A relevant section exists, but it lacks AI-specific structure or detail. A practitioner could improvise documentation here, but the framework provides no guidance for AI concerns. |
| **2** | Fully addressable | A dedicated section or artifact directly addresses the requirement. The framework provides explicit structure and guidance for documenting this concern. |

---

## What You Are Scoring

Four framework configurations:

1. **Standard arc42** — The twelve-section arc42 template as-is ([arc42.org](https://arc42.org/))
2. **Standard C4** — The C4 model's diagram system as-is (Context, Container, Component, Code + supplementary diagrams)
3. **RAD-AI arc42** — arc42 extended with eight AI-specific sections (E1-E8, described below)
4. **RAD-AI C4** — C4 extended with three AI-specific diagram types (C4-E1 to C4-E3, described below)

---

## RAD-AI Extensions Summary

### arc42 Extensions (E1-E8)

**E1: AI Boundary Delineation** (extends Section 3 — Context & Scope)
Requires explicitly marking which parts of the system are deterministic (traditional software) and which are non-deterministic (AI/ML). Each boundary crossing between these regions is annotated with a four-part contract: what type of output the AI produces (categorical, continuous, or generative), what confidence level is guaranteed (e.g., "precision >= 0.92"), how often the underlying model is refreshed, and what happens when the AI component fails or degrades (e.g., fall back to a rule-based default, use a cached prediction, or escalate to a human).

**E2: Model Registry View** (extends Section 5 — Building Block View)
Treats each ML model as a first-class architectural building block (rather than hiding it inside a generic container). Each model entry documents: model ID and version, ML framework used, a hash of the training dataset with lineage reference, hyperparameter snapshot, primary evaluation metric with acceptance threshold, current deployment status (shadow, canary, or production), the responsible owner, and when it was last retrained. Model Cards (standardized model documentation) attach as linked sub-artifacts.

**E3: Data Pipeline View** (extends Section 6 — Runtime View)
Documents the complete ML data flow from collection through preprocessing, feature engineering, training, inference, and feedback loops. Each pipeline stage is annotated with quality gates that specify: what is checked (schema conformance, distribution test, or completeness), what the threshold is (e.g., KS-statistic < 0.1, null rate < 1%), and what action is taken on failure (halt pipeline, alert and continue, or activate fallback). Data Cards attach at data source nodes.

**E4: Responsible AI Concepts** (extends Section 8 — Cross-Cutting Concepts)
Adds a structured concern matrix where rows are AI components and columns are five concern categories: fairness, explainability, human oversight, privacy, and safety. Each cell documents the applicable metric or method (e.g., demographic parity ratio, SHAP feature attributions), the acceptance threshold, how often it is monitored, and who is responsible. This provides a single place to document all responsible AI considerations across the system.

**E5: AI Decision Records — AI-ADR** (extends Section 9 — Architecture Decisions)
Extends the standard MADR (Markdown Any Decision Records) template with seven AI-specific fields: model alternatives considered, dataset description, fairness/bias considerations, expected model lifetime, retraining trigger conditions, explainability approach, and regulatory classification. For example, an AI-ADR might document why gradient-boosted trees were chosen over an LSTM for a route optimization service, with retraining triggered when prediction error exceeds a threshold for three consecutive days.

**E6: AI Quality Scenarios** (extends Section 10 — Quality Requirements)
Adds AI-specific quality scenarios following the established source-stimulus-response format from software architecture. Each scenario specifies: an AI-specific source of concern (data drift, model staleness, adversarial input), a quantitative trigger (e.g., "feature distribution shift > 2 standard deviations on 3+ features for > 12 hours"), the environment (training, serving, or monitoring), and a measurable response with deadline. Importantly, these scenarios can span multiple AI components (e.g., drift in one model causing degraded predictions in a downstream model).

**E7: AI Debt Register** (extends Section 11 — Risks & Technical Debt)
Adds ML-specific technical debt categories alongside conventional architectural debt. Each entry records: debt category (boundary erosion, entanglement, hidden feedback loop, data dependency, or pipeline debt), which components are affected, severity (low/medium/high based on blast radius), estimated remediation effort, owner, and status. This enables systematic tracking and prioritization of ML-specific debt that standard debt registers are not structured to capture.

**E8: Operational AI View** (new section — no arc42 equivalent)
An entirely new section covering the operational lifecycle of AI components, structured around four subsections: (a) monitoring — what metrics are tracked per model, dashboard specifications, and alerting thresholds; (b) retraining policy — what triggers retraining (scheduled, performance-based, or drift-based), automation level, and approval workflow; (c) deployment strategy — canary, blue-green, or shadow deployment with promotion criteria and traffic split ratios; (d) rollback policy — what triggers a rollback, how many model versions are retained, and what happens to downstream data when a rollback occurs.

### C4 Extensions (C4-E1 to C4-E3)

**C4-E1: AI Component Stereotypes**
Adds five new visual stereotypes to C4 diagrams so AI components are immediately distinguishable from traditional software containers: `<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, and `<<Human-in-the-Loop>>`. For example, a feature store is immediately recognizable as architecturally distinct from a standard database, even though both store data. Compatible with Structurizr DSL and PlantUML notation.

**C4-E2: Data Lineage Overlay**
A supplementary diagram layer that can be applied on top of existing C4 Container or Component diagrams. It traces data provenance from source through transformation to model consumption, annotated with schema expectations, data freshness constraints, and privacy classifications. This reveals data dependencies between components that are invisible in standard C4 diagrams (e.g., which models depend on which data sources, and how data flows across system boundaries).

**C4-E3: Non-Determinism Boundary Diagram**
A system-level overlay that visually partitions the entire architecture into deterministic regions (traditional software) and non-deterministic regions (AI/ML components). Each boundary is annotated with a three-property contract: confidence specification, fallback strategy, and degradation profile. This gives stakeholders an immediate visual understanding of where probabilistic behavior exists in the system and what safeguards are in place at each boundary.

---

## Annex IV Requirement Categories

The EU AI Act Annex IV specifies nine sections of required technical documentation for high-risk AI systems. We group these into ten scoring categories:

| # | Category | What Annex IV Requires |
|---|----------|----------------------|
| 1 | **General system description** | Description of the AI system's intended purpose, developer identity, version, and interaction with hardware/software |
| 2 | **System elements & development process** | Detailed description of system elements, development process, and tools used |
| 3 | **Design specifications & architecture** | "How software components build on or feed into each other and integrate into the overall processing" (Section 2(c)) |
| 4 | **Data & data governance** | Data provenance, labelling procedures, data cleaning, bias examination, data gaps (Section 2(d)) |
| 5 | **Training methodologies & techniques** | Training procedures, hyperparameters, optimization, data preparation |
| 6 | **Risk assessment & management** | Foreseeable risks, risk mitigation measures, residual risks |
| 7 | **Lifecycle change description** | Pre-determined changes, version history, update documentation |
| 8 | **Performance metrics & accuracy** | Relevant metrics, testing procedures, accuracy levels, known limitations |
| 9 | **Human oversight measures** | Technical measures for human oversight, human-machine interface (Section 2(e)) |
| 10 | **Post-market monitoring** | Logging capabilities, monitoring plan, post-deployment performance tracking |

---

## Scoring Table

Please fill in each cell with **0**, **1**, or **2**.

| # | Annex IV Requirement Category | Standard arc42 | Standard C4 | RAD-AI arc42 | RAD-AI C4 |
|---|-------------------------------|:--------------:|:-----------:|:------------:|:---------:|
| 1 | General system description | | | | |
| 2 | System elements & development process | | | | |
| 3 | Design specifications & architecture | | | | |
| 4 | Data & data governance | | | | |
| 5 | Training methodologies & techniques | | | | |
| 6 | Risk assessment & management | | | | |
| 7 | Lifecycle change description | | | | |
| 8 | Performance metrics & accuracy | | | | |
| 9 | Human oversight measures | | | | |
| 10 | Post-market monitoring | | | | |
| | **Total (out of 20)** | | | | |

---

## Participant Information (optional)

- **Role/title:**
- **Years of experience with software architecture:**
- **Familiarity with arc42 (1-5):**
- **Familiarity with C4 (1-5):**
- **Familiarity with AI/ML systems (1-5):**
- **Familiarity with EU AI Act (1-5):**

---

## Notes

If you have comments on any specific score (e.g., why you chose 1 instead of 0 or 2), please note them here:

| # | Notes |
|---|-------|
| 1 | |
| 2 | |
| 3 | |
| 4 | |
| 5 | |
| 6 | |
| 7 | |
| 8 | |
| 9 | |
| 10 | |
