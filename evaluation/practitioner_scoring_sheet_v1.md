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

| Ext. | Name | Extends | What It Adds |
|------|------|---------|--------------|
| E1 | AI Boundary Delineation | Section 3 (Context & Scope) | Marks deterministic vs. non-deterministic boundaries with contracts (output type, confidence, update frequency, fallback) |
| E2 | Model Registry View | Section 5 (Building Block View) | Elevates ML models to first-class building blocks with version, framework, dataset hash, metrics, deployment status |
| E3 | Data Pipeline View | Section 6 (Runtime View) | Documents ML data flow (collection through inference) with quality gates at each stage |
| E4 | Responsible AI Concepts | Section 8 (Cross-Cutting Concepts) | Concern matrix: fairness, explainability, human oversight, privacy, safety per AI component |
| E5 | AI Decision Records (AI-ADR) | Section 9 (Architecture Decisions) | Extended MADR template with 7 AI-specific fields (model alternatives, dataset, fairness, retraining trigger, etc.) |
| E6 | AI Quality Scenarios | Section 10 (Quality Requirements) | AI-specific quality scenarios (drift, staleness, adversarial input) in source-stimulus-response format |
| E7 | AI Debt Register | Section 11 (Risks & Technical Debt) | ML-specific debt categories (boundary erosion, entanglement, hidden feedback loops, pipeline debt) |
| E8 | Operational AI View | New section (no equivalent) | Monitoring, retraining policy, deployment strategy (canary/blue-green/shadow), rollback policy |

### C4 Extensions (C4-E1 to C4-E3)

| Ext. | Name | What It Adds |
|------|------|--------------|
| C4-E1 | AI Component Stereotypes | Five stereotypes: `<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, `<<Human-in-the-Loop>>` |
| C4-E2 | Data Lineage Overlay | Supplementary diagram layer tracing data provenance with schema expectations, freshness constraints, privacy classifications |
| C4-E3 | Non-Determinism Boundary Diagram | System-level overlay partitioning architecture into deterministic and non-deterministic regions with confidence/fallback annotations |

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
