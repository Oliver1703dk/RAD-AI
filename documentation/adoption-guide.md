# RAD-AI Adoption Guide

**A practical guide to incrementally adopting RAD-AI extensions**

> RAD-AI extends arc42 and C4 with AI-specific documentation capabilities. This guide provides a structured path for incremental adoption, from initial visibility through operational maturity.

---

## Table of Contents

1. [Adoption Philosophy](#1-adoption-philosophy)
2. [Prerequisites](#2-prerequisites)
3. [Three-Stage Adoption Path](#3-three-stage-adoption-path)
4. [Compliance-Driven Adoption](#4-compliance-driven-adoption)
5. [Team Roles](#5-team-roles)
6. [Integration with Existing Tools](#6-integration-with-existing-tools)
7. [Templates](#7-templates)
8. [Common Pitfalls](#8-common-pitfalls)

---

## 1. Adoption Philosophy

RAD-AI is built on three design principles that directly inform how you should adopt it:

**Backward-compatible.** Every RAD-AI extension augments an existing arc42 section or adds a supplementary C4 diagram type. Your existing arc42 documentation and C4 diagrams remain valid without modification. You are adding new sections and diagrams alongside what you already have, not rewriting anything.

**Incremental.** You do not need to adopt all eleven extensions at once. Each extension is self-contained and provides value independently. Start with the extensions most relevant to your context and add more as needs arise.

**Prioritized by context.** The right adoption order depends on your situation. The three-stage path in Section 3 provides a general recommendation. If regulatory compliance is your primary driver, Section 4 provides an alternative prioritization based on EU AI Act requirements. If you are unsure where to start, begin with Stage 1 (E1 and E2) to establish basic visibility over AI components.

### When to Adopt RAD-AI

RAD-AI is relevant when your system includes one or more AI components (ML models, data pipelines feeding models, feature stores, or inference services) and you are using arc42 or C4 for architecture documentation. Specific triggers for adoption include:

- A new AI component is being added to an existing system
- A regulatory assessment identifies documentation gaps for AI components
- Architecture reviews reveal that AI components are poorly documented relative to traditional components
- Incident post-mortems surface issues related to undocumented model behavior, data dependencies, or drift

---

## 2. Prerequisites

### For arc42 Extensions (E1 through E8)

You should have existing arc42 documentation with at least the following sections populated:

- **S1 (Introduction and Goals):** System purpose, key quality goals, stakeholders
- **S3 (Context and Scope):** System context diagram showing external interfaces
- **S5 (Building Block View):** At least the top-level decomposition of your system
- **S6 (Runtime View):** Key runtime scenarios documented

If you are starting arc42 documentation from scratch, fill in these four sections first using the standard arc42 template before adding RAD-AI extensions. The extensions build on the structure these sections provide.

### For C4 Extensions (C4-E1 through C4-E3)

You should have existing C4 diagrams with at least:

- **System Context diagram:** Showing the system boundary and external actors
- **Container diagram:** Showing the major deployable units within your system

If you are using Structurizr DSL or PlantUML, the C4 extensions integrate directly into your existing diagram definitions.

### Prerequisite Knowledge

- Understanding of which system components are AI-driven (if this is unclear, that is precisely what Stage 1 helps establish)
- Familiarity with your system's ML model inventory (names, purposes, approximate versions)
- Access to ML engineers or data scientists who can provide model and pipeline details

---

## 3. Three-Stage Adoption Path

The recommended adoption path progresses through three stages of increasing depth. Each stage is self-contained: you gain value immediately and can pause at any stage.

### Stage 1: Visibility

**Goal:** Establish a clear picture of where AI lives in your architecture.

| Extension | Description | Key Outcome |
|---|---|---|
| E1: AI Boundary Delineation | Mark deterministic vs. non-deterministic boundaries in your context diagram | Stakeholders can see at a glance which parts of the system exhibit probabilistic behavior |
| E2: Model Registry View | Create a model inventory table as part of the building block view | A complete list of deployed models with version, framework, status, and owner |
| C4-E1: AI Component Stereotypes | Apply `<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, and `<<Human-in-the-Loop>>` stereotypes to C4 diagrams | AI components are visually distinct from traditional software containers |

**Estimated effort:** 1 to 2 days for a medium-complexity system (5 to 15 AI components).

**How to execute:**

1. Start with E1. Take your existing arc42 Section 3 context diagram and annotate each external interface that involves an AI component. For each AI boundary, document the four-part contract: output type, confidence specification, update frequency, and fallback behavior.
2. Create the E2 model registry table. For each model in production, record: model ID and version, ML framework, training dataset reference, primary evaluation metric with threshold, deployment status, owner, and last-retrained date.
3. Update C4 Container and Component diagrams with the five stereotypes. This is often the fastest step if you are already using Structurizr DSL, as it requires only adding stereotype annotations to existing container definitions.

**Outcome:** After Stage 1, anyone reading your architecture documentation can immediately identify which components are AI-driven, what models are deployed, and where non-deterministic behavior enters the system.

### Stage 2: Decision Governance

**Goal:** Establish an auditable decision trail and measurable quality commitments for AI components.

| Extension | Description | Key Outcome |
|---|---|---|
| E5: AI Decision Records (AI-ADR) | Capture decision rationale for model selections and AI design choices | Each AI decision is documented with model alternatives, dataset characteristics, fairness considerations, retraining triggers, and regulatory rationale |
| E6: AI Quality Scenarios | Define measurable quality commitments using the source-stimulus-response format | Quantified quality contracts for model freshness, drift tolerance, explainability, and fairness |
| C4-E3: Non-Determinism Boundary | Partition architecture diagrams into deterministic and non-deterministic regions | Visual separation of probabilistic and deterministic system areas with annotated confidence contracts |

**Estimated effort:** 2 to 3 days. The bulk of the time is spent on AI-ADRs, which require input from ML engineers.

**How to execute:**

1. Begin with E5 (AI-ADRs). For each significant model selection or AI design decision, create an AI-ADR using the template at [`../templates/arc42-extensions/09-ai-adr-template.md`](../templates/arc42-extensions/09-ai-adr-template.md). The seven AI-specific fields (model alternatives, dataset characteristics, fairness/bias trade-offs, expected model lifetime, retraining triggers, explainability requirements, regulatory rationale) require collaboration with ML engineers.
2. Define E6 quality scenarios for your most critical AI components. Focus on scenarios that cross component boundaries (for example, what happens when an upstream model's accuracy degrades). Use the source-stimulus-environment-response format.
3. Create C4-E3 diagrams by adding a dashed boundary overlay to your existing Container or Component diagrams. Annotate each boundary with the confidence specification, fallback strategy, and degradation profile.

**Outcome:** After Stage 2, your documentation captures the "why" behind AI decisions and provides measurable quality commitments that can be monitored and audited.

### Stage 3: Operational Maturity

**Goal:** Document the full operational lifecycle of AI components, including data governance, responsible AI practices, technical debt, and operational procedures.

| Extension | Description | Key Outcome |
|---|---|---|
| E3: Data Pipeline View | Document complete data flows from collection through inference | Data pipeline diagrams with quality gates at each stage |
| E4: Responsible AI Concepts | Establish fairness, explainability, oversight, privacy, and safety practices | Concern matrix mapping each AI component to its responsible AI measures |
| E7: AI Debt Register | Track ML-specific technical debt | Structured register of boundary erosion, entanglement, hidden feedback loops, data dependency debt, and pipeline debt |
| E8: Operational AI View | Document monitoring, retraining, deployment, and rollback procedures | Complete operational runbook for AI components |
| C4-E2: Data Lineage Overlay | Trace data provenance from source to model consumption | Visual data lineage with schema expectations, freshness constraints, and privacy classifications |

**Estimated effort:** 3 to 5 days. This stage involves the most diverse set of stakeholders (data engineers, ML engineers, compliance staff, product owners).

**How to execute:**

1. Start with E3 (Data Pipeline View). Map the complete data flow for each AI component: collection, preprocessing, feature engineering, training, inference, and feedback loops. At each stage, specify quality gates with check type, threshold, and failure action.
2. Create E4 (Responsible AI Concepts) concern matrix. For each AI component, document the applicable metrics or methods for fairness, explainability, human oversight, privacy, and safety. Assign responsible parties and monitoring frequencies.
3. Populate E7 (AI Debt Register). Conduct a technical debt review focused on ML-specific categories. For each debt item, record the category, affected components, severity, estimated remediation effort, owner, and status.
4. Document E8 (Operational AI View) with four subsections: monitoring (metrics, dashboards, alerting), retraining policy (triggers, automation level, approval workflow), deployment strategy (canary/blue-green/shadow, promotion criteria), and rollback policy (triggers, version retention, downstream implications).
5. Add C4-E2 (Data Lineage Overlay) to your Container or Component diagrams. Trace data provenance from external sources through internal transformations to model consumption.

**Outcome:** After Stage 3, you have production-grade AI documentation covering the full lifecycle. For organizations subject to EU AI Act requirements, this stage brings you to near-complete Annex IV compliance readiness.

---

## 4. Compliance-Driven Adoption

For organizations where EU AI Act compliance is the primary motivation, the three-stage adoption path can be reprioritized based on which Annex IV categories represent the highest-risk gaps.

### Step 1: Determine Your Risk Classification

Use E1 (AI Boundary Delineation) to inventory all AI components and cross-reference them against EU AI Act Annex III categories. Determine whether your system is classified as high-risk, limited-risk, or minimal-risk. See [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md), Section 6, for detailed guidance.

### Step 2: Identify Your Highest-Risk Documentation Gaps

Use the compliance mapping table in [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md), Section 4, to identify which Annex IV categories are currently unaddressed in your documentation. Prioritize the extensions that fill these gaps.

### Step 3: Prioritize Extensions by Gap

The following prioritization targets organizations with high-risk AI systems facing the August 2, 2026 deadline:

**Highest priority (categories scored 0 for both standard frameworks):**
- E3 (Data Pipeline View) + C4-E2 (Data Lineage Overlay) for data governance (category 4)
- E4 (Responsible AI Concepts) for human oversight (category 9)
- E5 (AI-ADR) for training methodologies (category 5)

**High priority (categories scored 0 for one standard framework):**
- E7 (AI Debt Register) for risk management (category 6)
- E6 (AI Quality Scenarios) + C4-E3 (Non-Determinism Boundary) for performance metrics (category 8)
- E8 (Operational AI View) for post-market monitoring (category 10)

**Standard priority (categories partially addressed by standard frameworks):**
- E1 (AI Boundary Delineation) for general description (category 1)
- E2 (Model Registry View) for system elements (category 2)
- C4-E1 (AI Component Stereotypes) for design specifications (category 3)

### Step 4: Execute and Verify

After implementing the prioritized extensions, use the compliance checklist at [`../templates/compliance-checklist.md`](../templates/compliance-checklist.md) to verify coverage. Each of the 22 checklist items maps to a specific RAD-AI section and requires documented evidence.

---

## 5. Team Roles

Effective RAD-AI adoption requires input from multiple roles. No single role can fill in all extensions alone. The following table maps each extension to the roles best positioned to contribute.

| Extension | Primary Contributors | Reviewers |
|---|---|---|
| E1: AI Boundary Delineation | Software architects | ML engineers, product owners |
| E2: Model Registry View | ML engineers, software architects | DevOps/MLOps engineers |
| E3: Data Pipeline View | Data engineers, ML engineers | Software architects |
| E4: Responsible AI Concepts | Compliance/legal, product owners | Software architects, ML engineers |
| E5: AI Decision Records | ML engineers, software architects | Product owners, compliance/legal |
| E6: AI Quality Scenarios | Software architects, ML engineers | Product owners (quality priorities) |
| E7: AI Debt Register | ML engineers, software architects | Compliance/legal (risk review) |
| E8: Operational AI View | MLOps/DevOps engineers, ML engineers | Software architects |
| C4-E1: AI Component Stereotypes | Software architects | ML engineers |
| C4-E2: Data Lineage Overlay | Data engineers | ML engineers, compliance/legal |
| C4-E3: Non-Determinism Boundary | Software architects | ML engineers |

### Role-Specific Guidance

**Software architects** are the primary owners of RAD-AI adoption. They drive E1, E6, and all C4 extensions. Their existing arc42/C4 documentation expertise ensures consistency with the established architecture documentation.

**ML engineers** are essential contributors for E2, E3, E5, and E8. They hold the domain knowledge about model specifics, training procedures, data dependencies, and operational requirements. Without their input, these extensions will be incomplete or inaccurate.

**Data engineers** own E3 (Data Pipeline View) and C4-E2 (Data Lineage Overlay). They understand the full data flow from collection through transformation and can specify quality gates, schema expectations, and freshness constraints.

**Compliance and legal teams** should review E4 (Responsible AI Concepts) and E7 (AI Debt Register) and drive the compliance checklist process. They bring regulatory interpretation expertise that technical roles lack.

**Product owners** provide decision context for E5 (AI-ADRs) and quality prioritization for E6 (AI Quality Scenarios). Their involvement ensures that documented decisions align with business objectives and user needs.

---

## 6. Integration with Existing Tools

RAD-AI extensions are designed to integrate with tools already in use for ML development and operations. The extensions do not require new tooling; they provide documentation structure that can reference and link to artifacts produced by existing tools.

### Model Registries

**Tools:** MLflow, Weights and Biases (W&B), DVC, SageMaker Model Registry, Vertex AI Model Registry

**RAD-AI integration:** E2 (Model Registry View) references model registry entries. The model registry table in E2 can link to or be auto-generated from your model registry. Key fields to synchronize: model ID/version, framework, evaluation metrics, deployment status, and last-retrained date. Model Cards (integrated as sub-artifacts of E2) can link to W&B or MLflow run pages for detailed experiment tracking.

### Pipeline Orchestrators

**Tools:** Apache Airflow, Kubeflow Pipelines, Metaflow, Prefect, Dagster, ZenML

**RAD-AI integration:** E3 (Data Pipeline View) documents the architectural view of data pipelines. Link E3 pipeline stages to the corresponding DAG definitions in your orchestrator. Quality gate specifications in E3 should reference the monitoring and validation steps defined in your pipeline configuration.

### Monitoring Platforms

**Tools:** Evidently AI, Alibi Detect, Prometheus + Grafana, WhyLabs, Fiddler, Arize AI

**RAD-AI integration:** E8 (Operational AI View) documents the monitoring architecture. Reference dashboard URLs, alerting configurations, and drift detection thresholds from your monitoring platform. E6 (AI Quality Scenarios) can reference specific Evidently or Alibi Detect test suites as the implementation of quality scenario response criteria.

### C4 Tooling

**Tools:** Structurizr DSL, PlantUML, C4-PlantUML, Mermaid

**RAD-AI integration:** C4-E1 (AI Component Stereotypes) integrates directly with Structurizr DSL by adding stereotype tags to container and component definitions. C4-E2 (Data Lineage Overlay) and C4-E3 (Non-Determinism Boundary) can be implemented as supplementary diagram views in Structurizr workspaces or as additional PlantUML diagrams. The five stereotypes (`<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, `<<Human-in-the-Loop>>`) can be styled with distinct colors and shapes in any C4-compatible tool.

### Architecture Documentation Platforms

**Tools:** Confluence, Notion, GitLab/GitHub wikis, docs-as-code (Markdown in repository)

**RAD-AI integration:** All RAD-AI templates are provided in Markdown format and can be directly incorporated into any docs-as-code workflow. For wiki-based platforms, copy the template structure into your wiki and link from your existing arc42 pages to the new RAD-AI extension pages.

---

## 7. Templates

All RAD-AI extensions have corresponding templates in the [`../templates/`](../templates/) directory. Each template includes section headings, guidance comments, and example content to help you get started.

### Template Quick Reference

| Extension | Template File | Extends |
|---|---|---|
| E1: AI Boundary Delineation | [`templates/arc42-extensions/03-ai-boundary-delineation.md`](../templates/arc42-extensions/03-ai-boundary-delineation.md) | arc42 S3 |
| E2: Model Registry View | [`templates/arc42-extensions/05-model-registry-view.md`](../templates/arc42-extensions/05-model-registry-view.md) | arc42 S5 |
| E3: Data Pipeline View | [`templates/arc42-extensions/06-data-pipeline-view.md`](../templates/arc42-extensions/06-data-pipeline-view.md) | arc42 S6 |
| E4: Responsible AI Concepts | [`templates/arc42-extensions/08-responsible-ai-concepts.md`](../templates/arc42-extensions/08-responsible-ai-concepts.md) | arc42 S8 |
| E5: AI Decision Records | [`templates/arc42-extensions/09-ai-adr-template.md`](../templates/arc42-extensions/09-ai-adr-template.md) | arc42 S9 |
| E6: AI Quality Scenarios | [`templates/arc42-extensions/10-ai-quality-scenarios.md`](../templates/arc42-extensions/10-ai-quality-scenarios.md) | arc42 S10 |
| E7: AI Debt Register | [`templates/arc42-extensions/11-ai-debt-register.md`](../templates/arc42-extensions/11-ai-debt-register.md) | arc42 S11 |
| E8: Operational AI View | [`templates/arc42-extensions/12-operational-ai-view.md`](../templates/arc42-extensions/12-operational-ai-view.md) | New section |
| C4-E1: AI Component Stereotypes | [`templates/c4-extensions/ai-component-stereotypes.md`](../templates/c4-extensions/ai-component-stereotypes.md) | C4 diagrams |
| C4-E2: Data Lineage Overlay | [`templates/c4-extensions/data-lineage-overlay.md`](../templates/c4-extensions/data-lineage-overlay.md) | C4 diagrams |
| C4-E3: Non-Determinism Boundary | [`templates/c4-extensions/non-determinism-boundary.md`](../templates/c4-extensions/non-determinism-boundary.md) | C4 diagrams |
| Compliance Checklist | [`templates/compliance-checklist.md`](../templates/compliance-checklist.md) | N/A |

### How to Use Templates

1. Copy the relevant template file(s) into your project's documentation directory.
2. Read through the guidance comments (marked with `<!-- -->` or `> ` blockquotes) in each template.
3. Fill in the sections that apply to your system. Not every field in every template will be relevant; document what applies and mark irrelevant sections as "Not applicable" with a brief justification.
4. Link from your existing arc42 document or C4 workspace to the new extension files.
5. Remove guidance comments from the final documentation.

---

## 8. Common Pitfalls

Based on the design rationale behind RAD-AI and the patterns observed in the comparative analysis of Uber Michelangelo and Netflix Metaflow, the following pitfalls commonly arise during adoption.

### Pitfall 1: Adopting All Extensions at Once

**Problem:** Teams attempt to fill in all eleven extensions simultaneously, creating a large documentation effort that stalls or produces shallow content.

**Solution:** Follow the three-stage adoption path. Stage 1 (E1, E2, C4-E1) takes 1 to 2 days and provides immediate value. Pause there if needed. Add stages as capacity allows.

### Pitfall 2: Treating Templates as Forms

**Problem:** Teams fill in every field of every template mechanically without adapting the documentation to their specific system. The result is boilerplate documentation that fails to capture the actual architectural concerns.

**Solution:** Treat RAD-AI templates as starting structures, not checkbox exercises. Adapt, extend, or trim sections based on your system's characteristics. A model registry table with five well-documented models is more useful than a table with twenty models where every field contains placeholder text.

### Pitfall 3: Documenting AI Components Without ML Engineers

**Problem:** Software architects fill in AI-specific extensions (E2, E3, E5, E8) without consulting the ML engineers who built and maintain the models. The resulting documentation is inaccurate or incomplete.

**Solution:** See Section 5 (Team Roles). ML engineers are essential contributors for E2, E3, E5, and E8. Schedule dedicated sessions with ML engineers for these extensions. A 90-minute joint session is typically more productive than multiple rounds of asynchronous review.

### Pitfall 4: Skipping Quality Scenarios

**Problem:** Teams document model inventories and data pipelines (the "what") but skip E6 quality scenarios (the "how well"), relying on informal descriptions like "the model should be accurate" instead of measurable commitments.

**Solution:** Define at least one quality scenario per AI component in the source-stimulus-environment-response format. Include quantitative thresholds and deadlines. These scenarios serve as testable contracts and are directly useful for monitoring and alerting configuration.

### Pitfall 5: Forgetting to Update After Retraining

**Problem:** RAD-AI documentation is created once and never updated. Models are retrained, pipelines are modified, and new models are deployed, but the documentation still reflects the original state.

**Solution:** Tie documentation updates to your deployment workflow. When a model is retrained and promoted to production, update the E2 model registry entry and review E6 quality scenarios. When a new model is added, create an E5 AI-ADR. Consider adding documentation review to your model deployment checklist.

### Pitfall 6: Ignoring Ecosystem-Level Concerns

**Problem:** Teams document individual AI components in isolation without considering cross-component dependencies. Cascading drift, shared feature store governance, and differentiated compliance requirements remain invisible.

**Solution:** Use C4-E2 (Data Lineage Overlay) to trace data dependencies across component boundaries. Use E6 quality scenarios to define cross-component scenarios (for example, "what happens when the upstream model's accuracy degrades?"). If your system shares data or features across teams or organizations, document governance boundaries in E4 (Responsible AI Concepts).

### Pitfall 7: Over-Documenting Low-Risk Components

**Problem:** Teams apply the full set of RAD-AI extensions to every AI component, regardless of risk classification or complexity. A simple rule-based classifier gets the same documentation treatment as a safety-critical anomaly detection system.

**Solution:** Use E1 (AI Boundary Delineation) and the risk classification guide in [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md), Section 6, to differentiate documentation depth by risk level. High-risk components warrant full Stage 3 documentation. Minimal-risk components may only need Stage 1 (visibility).

---

*This guide is part of the RAD-AI framework. For the complete framework reference, see [framework-overview.md](framework-overview.md). For EU AI Act compliance specifics, see [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md).*
