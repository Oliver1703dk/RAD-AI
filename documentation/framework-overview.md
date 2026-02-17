# RAD-AI Framework Overview

## Table of Contents

1. [Introduction](#1-introduction)
2. [Motivation](#2-motivation)
3. [Design Principles](#3-design-principles)
4. [Framework Architecture](#4-framework-architecture)
5. [Identified Documentation Gaps (G1 through G5)](#5-identified-documentation-gaps-g1-through-g5)
6. [How the Extensions Address Each Gap](#6-how-the-extensions-address-each-gap)
7. [Relationship to Standards and Regulations](#7-relationship-to-standards-and-regulations)
8. [Comparison with Related Approaches](#8-comparison-with-related-approaches)

---

## 1. Introduction

### What is RAD-AI?

RAD-AI (Rethinking Architecture Documentation for AI-Augmented Ecosystems) is a backward-compatible extension framework that augments two widely adopted software architecture documentation frameworks:

- **arc42**, a pragmatic twelve-section template widely adopted in European industry for structuring software architecture documentation.
- **The C4 model**, a hierarchical diagram system (System Context, Container, Component, Code) commonly implemented through Structurizr DSL and PlantUML.

RAD-AI extends arc42 with eight AI-specific sections (E1 through E8) and adds three new diagram types to the C4 model (C4-E1 through C4-E3). It also provides a systematic compliance mapping between these extensions and the EU AI Act Annex IV documentation requirements (Regulation (EU) 2024/1689), offering a concrete path toward the August 2, 2026 enforcement deadline for high-risk AI systems.

RAD-AI does not replace arc42 or C4. It augments them. All existing documentation produced under standard arc42 or C4 remains valid after adopting RAD-AI. Practitioners add new sections and diagram types where AI-specific concerns arise, leaving the rest of their documentation unchanged.

### The Problem RAD-AI Solves

Software architecture documentation frameworks were designed for deterministic software systems. They assume that system behavior can be fully specified at design time, that building blocks maintain stable and well-defined contracts, and that system evolution occurs through code changes following a single release lifecycle. These assumptions hold for traditional software but break down for AI-augmented ecosystems, where:

- Model outputs are probabilistic rather than deterministic.
- System behavior changes through data distribution shifts rather than code commits.
- ML training and serving cycles run asynchronously with software release cycles.
- Quality attributes such as fairness, explainability, and drift resistance are first-class concerns with no corresponding documentation sections.
- Regulatory requirements (most notably the EU AI Act) mandate technical documentation that no existing framework provides structured support for.

The result is a structural gap: practitioners cannot document the most architecturally significant aspects of their AI-augmented systems using the frameworks they already rely on. RAD-AI closes this gap through targeted, backward-compatible extensions.

### Key Results at a Glance

Evaluation across three complementary methods provides preliminary evidence of RAD-AI's effectiveness:

| Metric | Standard Frameworks | RAD-AI Extended |
|--------|--------------------|-----------------|
| Annex IV coverage (arc42) | 35% (7/20) | 95% (19/20) |
| Annex IV coverage (C4) | 25% (5/20) | 75% (15/20) |
| AI concerns captured (Uber Michelangelo) | 0 full, 2 partial | 8 full, 2 partial |
| AI concerns captured (Netflix Metaflow) | 0 full, 2 partial | 8 full, 2 partial |
| Ecosystem-level concerns surfaced | 0 | 3 |

The most notable finding is cross-domain consistency: despite serving fundamentally different domains (marketplace versus content recommendation), both production systems exhibit identical gap patterns under standard frameworks and identical improvement under RAD-AI. This suggests the documentation deficiencies are structural properties of the frameworks rather than domain-specific oversights. For full evaluation details, see [evaluation-results.md](evaluation-results.md).

### Target Audience

RAD-AI is designed for:

- **Software architects** responsible for documenting and communicating system architecture in organizations that deploy AI components.
- **ML engineers** who need their model lifecycle, data pipeline, and operational concerns reflected in architecture documentation rather than siloed in ML-specific tooling.
- **Compliance officers** responsible for producing EU AI Act Annex IV technical documentation and who need a structured mapping from regulatory requirements to documentation artifacts.
- **Technical leads** overseeing teams that span both traditional software engineering and machine learning, who need a shared documentation language.

---

## 2. Motivation

### Five Challenges AI Introduces to Architecture Documentation

AI-augmented systems differ from traditional software in five fundamental ways that break the assumptions underlying current documentation frameworks. Each challenge represents a category of architectural concern that has no home in standard arc42 or C4.

#### Challenge 1: Non-Deterministic Behavior

Traditional software components produce deterministic outputs for given inputs. AI components produce probabilistic outputs that vary with training data, inference conditions, and input distributions. No existing architecture notation captures acceptable output ranges, confidence intervals, or degradation profiles.

For example, a fraud detection model might maintain 92% precision under normal conditions but degrade to 78% when input distributions shift seasonally. Standard arc42 building blocks and C4 containers have no way to express this behavioral range or the conditions under which it changes. The architecture documentation treats the model as a black box with a stable interface, hiding the most architecturally significant property of the component.

#### Challenge 2: Data-Dependent Evolution

In traditional systems, behavior changes through code commits that pass through version control, code review, and deployment pipelines. In AI-augmented systems, behavior changes primarily through data distribution shifts. The training data, feature distributions, and input patterns that determine model behavior evolve continuously, often without explicit human intervention.

Sculley et al. characterize this as "data dependency debt," noting that data dependencies in ML systems are more difficult to track than code dependencies. A model retrained on last month's data may behave differently from the same model retrained on this month's data, even though no code changed. Standard documentation frameworks have no mechanism to capture data lineage, schema expectations, freshness constraints, or the relationship between data characteristics and system behavior.

#### Challenge 3: Dual Lifecycle Complexity

Traditional software follows a single lifecycle: design, implement, test, deploy, maintain. AI-augmented systems follow two interleaved lifecycles: the software release cycle (versioning, CI/CD, deployment) and the ML lifecycle (data collection, feature engineering, training, evaluation, deployment, monitoring, retraining). These cycles run asynchronously; a model may be retrained and redeployed multiple times between software releases, or a software update may invalidate assumptions baked into a trained model.

Current documentation frameworks provide runtime views and deployment views oriented around the software lifecycle. They offer no way to document the ML lifecycle, its relationship to the software lifecycle, or the coordination points between them.

#### Challenge 4: Emergent Quality Attributes

Software architecture has well-established quality attributes: performance, reliability, security, maintainability, and so on. AI-augmented systems introduce additional quality attributes that are absent from architecture documentation templates:

- **Fairness:** Does the system produce equitable outcomes across demographic groups? What metrics are used (demographic parity, equalized odds, individual fairness)?
- **Explainability:** Can the system's decisions be understood and justified? What methods are employed (SHAP, LIME, feature importance)?
- **Drift resistance:** How well does the system maintain performance as input distributions change over time?
- **Robustness:** How does the system handle adversarial inputs, out-of-distribution data, or edge cases?

These are not incidental properties; they are first-class architectural concerns that affect design decisions, deployment strategies, and regulatory compliance. Standard quality scenario templates (arc42 Section 10) are structured around traditional quality attributes and provide no guidance for specifying or measuring AI-specific ones.

#### Challenge 5: Regulatory Requirements

The EU AI Act (Regulation (EU) 2024/1689) entered into force in August 2024 and introduces binding documentation requirements through its Annex IV. For high-risk AI systems, enforcement begins August 2, 2026. Annex IV specifies nine sections of required technical documentation spanning:

- General system description
- System elements and development process
- Design specifications and architecture
- Data and data governance
- Training methodologies
- Risk management
- Lifecycle changes
- Performance metrics
- Human oversight measures

Critically, Annex IV Section 2(c) explicitly requires documentation of "how software components build on or feed into each other and integrate into the overall processing"; this is a direct mandate for architecture documentation. No current framework provides structured guidance for producing the required documentation.

Additional regulatory frameworks reinforce this demand: NIST AI RMF 1.0 in the United States, ISO/IEC 42001:2023 for AI management systems, and harmonized European standards under development (prEN 18286). The regulatory landscape is converging on the expectation that AI systems will have comprehensive, structured technical documentation.

### The Scale of the Problem

Industry surveys suggest that 93% of organizations experience negative business outcomes from architecture-implementation misalignment (vFunction, 2025). For traditional software, this misalignment is an engineering concern affecting productivity, quality, and maintainability. For AI-augmented ecosystems, the consequences extend further: undocumented model behavior creates safety risks, undocumented data dependencies create silent failure modes, and undocumented AI components create regulatory liability under the EU AI Act. The documentation gap is not merely an inconvenience; it is a structural deficiency with engineering, safety, and legal consequences.

---

## 3. Design Principles

RAD-AI is governed by three design principles that guided every extension decision.

### Principle 1: Backward Compatibility

All extensions augment existing arc42 sections or add new diagram types to the C4 model. Existing documentation remains valid and unchanged. An organization that has documented its system using standard arc42 can adopt RAD-AI extensions incrementally without modifying any existing content. The extensions add new subsections, tables, and annotations; they do not alter the structure or semantics of existing sections.

This principle is critical for adoption. Practitioners have invested significant effort in their current documentation. Requiring them to restructure or rewrite existing content would create an adoption barrier disproportionate to the benefit. RAD-AI respects that investment.

### Principle 2: Minimal Disruption

Practitioners can adopt extensions incrementally, starting with whichever extensions are most relevant to their immediate needs. There is no requirement to adopt all eleven extensions simultaneously. A team concerned primarily with regulatory compliance might start with E1 (AI Boundary Delineation) and the compliance mapping. A team focused on operational maturity might begin with E8 (Operational AI View) and E7 (AI Debt Register).

The adoption guide (see [adoption-guide.md](adoption-guide.md)) proposes a three-stage path, but this is a recommendation rather than a requirement. Extensions are designed to be independently useful.

### Principle 3: Traceability

Each extension maps to one or more of the five identified documentation gaps (G1 through G5). This traceability serves two purposes: it justifies each extension by linking it to a demonstrated deficiency, and it enables practitioners to select extensions based on which gaps are most relevant to their context.

The gap-to-extension mapping is documented in Section 6 of this document and in the research paper.

---

## 4. Framework Architecture

### Overview

RAD-AI's extension structure consists of three layers:

1. **arc42 extensions (E1 through E8):** Eight extensions to arc42's twelve-section template. Seven extend existing sections; one (E8: Operational AI View) adds a new section with no equivalent in standard arc42.
2. **C4 extensions (C4-E1 through C4-E3):** Three new diagram types that augment C4's hierarchical visualization system. All are compatible with Structurizr DSL and PlantUML.
3. **Compliance mapping layer:** A systematic mapping between EU AI Act Annex IV requirement categories and RAD-AI sections, guiding practitioners to the documentation artifacts each regulatory requirement demands.

### Relationship to arc42

arc42 provides a twelve-section template for software architecture documentation:

| Section | Name | RAD-AI Extension |
|---------|------|------------------|
| S1 | Introduction and Goals | (unchanged) |
| S2 | Constraints | (unchanged) |
| S3 | Context and Scope | **E1: AI Boundary Delineation** |
| S4 | Solution Strategy | (unchanged) |
| S5 | Building Block View | **E2: Model Registry View** |
| S6 | Runtime View | **E3: Data Pipeline View** |
| S7 | Deployment View | (unchanged) |
| S8 | Cross-Cutting Concepts | **E4: Responsible AI Concepts** |
| S9 | Architecture Decisions | **E5: AI Decision Records (AI-ADR)** |
| S10 | Quality Requirements | **E6: AI Quality Scenarios** |
| S11 | Risks and Technical Debt | **E7: AI Debt Register** |
| S12 | Glossary | (unchanged) |
| (new) | (no equivalent) | **E8: Operational AI View** |

Five sections (S1, S2, S4, S7, S12) are unchanged. Seven sections receive AI-specific extensions. One entirely new section (E8) is added for operational AI concerns that have no counterpart in standard arc42.

### Relationship to the C4 Model

The C4 model provides four core diagram levels (System Context, Container, Component, Code) plus supplementary diagram types (Dynamic, Deployment, System Landscape). RAD-AI adds three new diagram types:

| Extension | Name | Relationship to C4 |
|-----------|------|---------------------|
| C4-E1 | AI Component Stereotypes | Adds five new stereotypes usable in any C4 diagram |
| C4-E2 | Data Lineage Overlay | A supplementary overlay applied to Container or Component diagrams |
| C4-E3 | Non-Determinism Boundary Diagram | A system-level overlay partitioning deterministic and non-deterministic regions |

These extensions are additive. Existing C4 diagrams remain valid. The new stereotypes (C4-E1) can be applied selectively to AI components in any existing diagram. The overlay diagrams (C4-E2, C4-E3) are supplementary views that complement rather than replace standard diagrams.

### Summary of All Extensions

The following table provides a consolidated view of all eleven RAD-AI extensions.

| ID | Name | Extends | Key Artifact | Gaps Addressed |
|----|------|---------|--------------|----------------|
| E1 | AI Boundary Delineation | arc42 S3 (Context and Scope) | Annotated context diagram with AI boundaries | G1, G2 |
| E2 | Model Registry View | arc42 S5 (Building Block View) | Model registry table and annotated diagram | G1, G5 |
| E3 | Data Pipeline View | arc42 S6 (Runtime View) | Data pipeline diagram with quality gates | G1, G5 |
| E4 | Responsible AI Concepts | arc42 S8 (Cross-Cutting Concepts) | Responsible AI concern matrix | G1 |
| E5 | AI Decision Records | arc42 S9 (Architecture Decisions) | AI-ADR template with 7 AI-specific fields | G4 |
| E6 | AI Quality Scenarios | arc42 S10 (Quality Requirements) | AI quality scenario table | G1 |
| E7 | AI Debt Register | arc42 S11 (Risks and Technical Debt) | AI debt register with remediation plan | G1 |
| E8 | Operational AI View | New section (no arc42 equivalent) | Operational AI architecture diagram | G1 |
| C4-E1 | AI Component Stereotypes | C4 diagram system | 5 new stereotypes for C4 diagrams | G2 |
| C4-E2 | Data Lineage Overlay | C4 Container/Component diagrams | Data provenance overlay diagram | G2, G5 |
| C4-E3 | Non-Determinism Boundary | C4 system-level diagrams | Boundary partition diagram | G2 |

### The Five C4 Stereotypes (C4-E1)

C4-E1 introduces five stereotypes that visually distinguish AI components from traditional software containers and components:

| Stereotype | Purpose | Distinguishes From |
|------------|---------|-------------------|
| `<<ML Model>>` | Marks components that perform inference or prediction | Standard software components |
| `<<Data Pipeline>>` | Marks data transformation and feature engineering workflows | Standard ETL or message queues |
| `<<Feature Store>>` | Marks shared feature computation and serving infrastructure | Standard databases or caches |
| `<<Monitor>>` | Marks drift detection, performance monitoring, and alerting systems | Standard logging or APM |
| `<<Human-in-the-Loop>>` | Marks components requiring human judgment or override capability | Standard user interfaces |

These stereotypes are compatible with Structurizr DSL and PlantUML notation. In practice, they serve a critical communication function: a `<<Feature Store>>` is immediately recognizable as architecturally distinct from a standard database, even though both store data. Similarly, a `<<Monitor>>` that tracks model drift has fundamentally different architectural responsibilities than a standard application performance monitoring service. By making these distinctions explicit in shared diagrams, the stereotypes bridge the vocabulary gap between data scientists and software architects.

### EU AI Act Compliance Mapping

The compliance mapping layer bridges RAD-AI sections to ten requirement categories derived from Annex IV's nine sections:

| # | Annex IV Category | RAD-AI Section(s) |
|---|-------------------|-------------------|
| 1 | General description | arc42 S1, S3 + E1 (AI Boundary Delineation) |
| 2 | System elements | S5 + E2 (Model Registry View) |
| 3 | Design and architecture | S5 through S7 + C4-E1 (AI Component Stereotypes) |
| 4 | Data governance | E3 (Data Pipeline View) + C4-E2 (Data Lineage Overlay) |
| 5 | Training methods | E5 (AI-ADR) + supplementary artifacts |
| 6 | Risk management | S11 + E7 (AI Debt Register) |
| 7 | Lifecycle changes | E2 (model versions) + E8 (Operational AI View) |
| 8 | Performance metrics | E6 (AI Quality Scenarios) + C4-E3 (Non-Determinism Boundary) |
| 9 | Human oversight | E4 (Responsible AI Concepts) + Human-in-the-Loop stereotype |
| 10 | Post-market monitoring | E8 (Operational AI View) |

For detailed compliance guidance, see [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md).

---

## 5. Identified Documentation Gaps (G1 through G5)

RAD-AI's extensions are grounded in a systematic gap analysis. Five documentation gaps were identified through structured concern extraction from academic literature, industry frameworks, regulatory requirements, and international standards, followed by gap mapping of each AI concern to existing arc42 and C4 sections.

### G1: No AI-Specific arc42 Sections

Standard arc42 provides no sections for documenting model lifecycle management, data pipeline architectures, drift monitoring strategies, operational AI concerns (retraining, canary deployment, rollback), or AI-specific quality attributes (fairness, explainability, drift resistance). These concerns must either be forced into ill-fitting existing sections or documented in ad hoc supplementary artifacts that are not integrated into the architecture documentation structure.

### G2: No C4 Diagram Types for ML Components

Standard C4 treats all components as software containers. There is no visual distinction between a database and a feature store, between a microservice and an ML model, or between a monitoring dashboard and a drift detection system. The absence of AI-specific notation means that the most architecturally significant properties of AI components (non-deterministic behavior, confidence contracts, data dependencies) are invisible in architecture diagrams.

### G3: No Mapping Between EU AI Act Annex IV and arc42/C4

No published work maps Annex IV's nine documentation sections to specific arc42 sections or C4 diagram types. Practitioners facing the August 2026 compliance deadline have no structured guidance for translating regulatory requirements into architecture documentation activities.

### G4: No AI-Specific Architecture Decision Record Templates

Standard MADR (Markdown Any Decision Records) templates capture context, decision, and consequences but include no fields for model alternatives considered, dataset characteristics, fairness and bias trade-offs, expected model lifetime, retraining triggers, explainability requirements, or regulatory rationale. AI-specific architectural decisions (which model architecture to use, what retraining strategy to adopt, how to handle fairness constraints) are either documented without critical AI context or not documented at all.

### G5: No Integration of Model Cards and Data Cards

Model Cards (Mitchell et al., 2019) and Data Cards (Google, 2022) are established documentation artifacts for individual models and datasets. However, they exist as standalone artifacts disconnected from architecture documentation. No framework integrates them as sub-artifacts of architectural building blocks, meaning that the relationship between a model's documented properties and its architectural role is not captured.

---

## 6. How the Extensions Address Each Gap

The following table maps each RAD-AI extension to the gaps it addresses, with a brief explanation of the mechanism.

| Extension | Gaps | Mechanism |
|-----------|------|-----------|
| E1: AI Boundary Delineation | G1, G2 | Marks deterministic vs. non-deterministic boundaries in the context diagram. Each boundary crossing is annotated with a four-part contract: output type, confidence specification, update frequency, and fallback behavior. |
| E2: Model Registry View | G1, G5 | Elevates AI models to first-class building blocks with versioning metadata (model ID, ML framework, training dataset hash, hyperparameters, evaluation metrics, deployment status, owner, last-retrained date). Integrates Model Cards as linked sub-artifacts. |
| E3: Data Pipeline View | G1, G5 | Documents the complete ML data flow with quality gates at each stage (schema conformance, distribution tests, completeness checks, with thresholds and failure actions). Integrates Data Cards at data source nodes. |
| E4: Responsible AI Concepts | G1 | Adds a structured concern matrix with rows for AI components and columns for five concern categories (fairness, explainability, human oversight, privacy, safety). Each cell specifies the metric, threshold, monitoring frequency, and responsible party. |
| E5: AI Decision Records | G4 | Extends MADR with seven AI-specific fields: model alternatives considered, dataset characteristics (size, time span, features), fairness/bias trade-offs, expected model lifetime, retraining triggers (with quantitative thresholds), explainability requirements (methods and audit context), and regulatory rationale (applicable AI Act articles). |
| E6: AI Quality Scenarios | G1 | Adds AI-specific quality scenarios following the established source-stimulus-response format from Bass et al. Each scenario specifies an AI-specific source (data drift, model staleness, adversarial input), a stimulus with quantitative trigger, an environment (training, serving, or monitoring), and a measurable response with deadline. Uniquely supports cross-component scenarios such as cascading drift across ecosystem boundaries. |
| E7: AI Debt Register | G1 | Tracks ML-specific technical debt categories (boundary erosion, entanglement, hidden feedback loops, data dependency debt, pipeline debt) with affected components, severity, remediation effort, owner, and status. |
| E8: Operational AI View | G1 | Adds a new arc42 section (no standard equivalent exists) structured around four required subsections: (a) monitoring, specifying metrics tracked per model, dashboard specifications, and alerting thresholds; (b) retraining policy, defining triggers (scheduled, performance-based, or drift-based), automation level, and approval workflow; (c) deployment strategy, covering canary, blue-green, or shadow deployment with promotion criteria and traffic split ratios; (d) rollback policy, specifying rollback triggers, model version retention depth, and downstream data implications. |
| C4-E1: AI Component Stereotypes | G2 | Introduces five stereotypes (`<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, `<<Human-in-the-Loop>>`) that make AI components visually distinct in C4 diagrams. |
| C4-E2: Data Lineage Overlay | G2, G5 | Traces data provenance from source through transformation to model consumption. Annotations capture schema expectations, data freshness constraints, and privacy classifications. Connects to Data Card sub-artifacts. |
| C4-E3: Non-Determinism Boundary | G2 | Partitions the architecture into deterministic and non-deterministic regions at the diagram level. Each boundary is annotated with confidence specification, fallback strategy, and degradation profile. |

### Gap Coverage Summary

| Gap | Extensions Addressing It |
|-----|--------------------------|
| G1: No AI-specific arc42 sections | E1, E2, E3, E4, E6, E7, E8 |
| G2: No C4 diagram types for ML | E1, C4-E1, C4-E2, C4-E3 |
| G3: No Annex IV mapping | Compliance mapping layer (see Section 4) |
| G4: No AI-specific ADR templates | E5 |
| G5: No Model/Data Card integration | E2, E3, C4-E2 |

---

## 7. Relationship to Standards and Regulations

RAD-AI is designed to complement and support compliance with several international standards and regulatory frameworks. This section summarizes the relationship to each.

### EU AI Act (Regulation (EU) 2024/1689)

The EU AI Act is the primary regulatory driver for RAD-AI. It establishes a risk-based classification system for AI systems and introduces binding documentation requirements through Annex IV for high-risk AI systems. Key milestones:

- **August 2024:** Regulation entered into force.
- **February 2025:** Prohibited AI practices took effect.
- **August 2025:** Transparency obligations for general-purpose AI models took effect.
- **August 2, 2026:** Enforcement begins for high-risk AI system documentation requirements (Annex IV).

Annex IV Section 2(c) explicitly requires documentation of "how software components build on or feed into each other and integrate into the overall processing," which is a direct mandate for architecture documentation. RAD-AI's compliance mapping (Section 4 of this document) provides a structured path from each Annex IV requirement to specific documentation artifacts.

RAD-AI evaluation demonstrates that standard arc42 achieves approximately 35% Annex IV addressability (7/20 in author-assessed scoring), while RAD-AI-extended arc42 achieves approximately 95% (19/20). The sole partial score relates to training methodology documentation, which requires supplementary artifacts beyond architecture documentation alone.

### ISO/IEC/IEEE 42010:2022

This international standard defines the concepts and practices of architecture description. Both arc42 and C4 align with its viewpoint and view concepts. RAD-AI's extensions are consistent with 42010's framework: each extension defines additional viewpoints (e.g., the Operational AI Viewpoint) or extends existing viewpoints with AI-specific concerns, using the standard's vocabulary of stakeholders, concerns, architecture views, and correspondence rules.

### ISO/IEC 25059:2023

This standard extends the SQuaRE quality model (ISO/IEC 25010) with AI-specific quality characteristics. RAD-AI's E6 (AI Quality Scenarios) aligns with 25059 by providing a documentation structure for AI-specific quality attributes. The quality scenario format (source, stimulus, environment, measurable response) enables practitioners to specify and track the quality characteristics defined in 25059 within their architecture documentation.

### ISO/IEC 42001:2023

This standard specifies requirements for establishing, implementing, maintaining, and continually improving an AI management system. RAD-AI supports 42001 compliance by providing the documentation structure needed for the technical documentation component of an AI management system. In particular, E4 (Responsible AI Concepts) and E7 (AI Debt Register) align with 42001's requirements for risk management and continual improvement.

### NIST AI Risk Management Framework 1.0

NIST AI RMF 1.0 provides a voluntary framework for managing AI risks. Its four core functions (Govern, Map, Measure, Manage) align with RAD-AI extensions: E4 (Responsible AI Concepts) supports Govern and Map; E6 (AI Quality Scenarios) supports Measure; E8 (Operational AI View) and E7 (AI Debt Register) support Manage. RAD-AI provides the documentation structure through which NIST AI RMF practices are recorded in architecture documentation.

### prEN 18286 (Harmonized Standards in Development)

CEN-CENELEC JTC 21 is developing harmonized standards for AI quality management systems aligned with the EU AI Act. prEN 18286 is currently in public enquiry. When finalized, these standards may introduce additional documentation requirements beyond current Annex IV text. RAD-AI's modular extension structure is designed to accommodate such additions: new requirements can be mapped to existing extensions or addressed through further extensions without disrupting the existing framework.

---

## 8. Comparison with Related Approaches

Several related approaches address aspects of AI documentation. None extends arc42 or C4 directly. This section positions RAD-AI relative to the most closely related work.

### Model Cards (Mitchell et al., 2019)

**What it does:** Model Cards provide a standardized format for documenting individual ML models, including intended use, performance metrics, ethical considerations, and limitations.

**Relationship to RAD-AI:** Model Cards are standalone artifacts that document individual models in isolation. They do not integrate into architecture documentation frameworks and cannot capture architectural relationships between models, their data dependencies, or their role in the broader system. RAD-AI integrates Model Cards as sub-artifacts within E2 (Model Registry View), connecting model-level documentation to architectural context. RAD-AI and Model Cards are complementary rather than competing.

### HASC System Cards (Sidhpurwala et al., 2025)

**What it does:** Hazard-Aware System Cards (HASC) propose machine-readable AI system cards with Assigned Safety Hazard (ASH) identifiers and ISO/IEC 42001 alignment. They provide a governance artifact for end-to-end transparency.

**Relationship to RAD-AI:** HASC operates as a standalone governance artifact. It provides hazard identification and safety documentation but does not extend arc42 or C4, and does not integrate into the architecture documentation workflow that practitioners use. RAD-AI addresses the architecture documentation layer specifically, embedding AI concerns into the framework structure. The two approaches could be used together: HASC for system-level safety governance and RAD-AI for architecture documentation.

### CLeAR Principles (Chmielinski et al., 2024)

**What it does:** The CLeAR documentation framework proposes four principles for AI transparency: Completeness, Legibility, Accessibility, and Relevance.

**Relationship to RAD-AI:** CLeAR provides high-level principles for what good AI documentation should achieve, but it does not provide concrete framework extensions, templates, or sections. RAD-AI provides the concrete structural extensions that embody such principles within arc42 and C4. CLeAR principles can serve as quality criteria for evaluating RAD-AI documentation outputs.

### TechOps Templates (Lucaj et al., 2025)

**What it does:** TechOps provides technical documentation templates specifically designed for EU AI Act compliance, mapping regulatory requirements to documentation sections.

**Relationship to RAD-AI:** TechOps focuses on regulatory compliance templates but operates independently of architecture documentation frameworks. It does not extend arc42 or C4 and does not address architectural concerns such as non-determinism boundaries, data lineage, or AI-specific quality scenarios. RAD-AI provides a compliance mapping (Section 4) that achieves a similar regulatory alignment but does so within the architecture documentation structure that practitioners already use.

### AWS Well-Architected ML Lens

**What it does:** The AWS Well-Architected ML Lens provides cloud-specific best practices and review questions for ML workloads deployed on AWS, organized around the Well-Architected Framework's six pillars.

**Relationship to RAD-AI:** The ML Lens is cloud-provider-specific and oriented toward operational review rather than architecture documentation. It provides valuable implementation guidance for AWS-hosted ML systems but does not extend any framework-agnostic documentation standard. RAD-AI is cloud-agnostic and extends frameworks (arc42, C4) that are used regardless of deployment platform. Organizations using AWS can apply the ML Lens for operational review and RAD-AI for architecture documentation.

### Summary Comparison

| Approach | Type | Extends arc42/C4? | Compliance Mapping? | Architecture Integration? |
|----------|------|-------------------|---------------------|---------------------------|
| Model Cards | Standalone artifact | No | No | No |
| HASC System Cards | Governance artifact | No | Partial (ISO 42001) | No |
| CLeAR Principles | Principles | No | No | No |
| TechOps Templates | Compliance templates | No | Yes (EU AI Act) | No |
| AWS ML Lens | Cloud-specific review | No | No | No |
| **RAD-AI** | **Framework extension** | **Yes** | **Yes (EU AI Act Annex IV)** | **Yes** |

RAD-AI is, to our knowledge, the first approach that provides concrete, backward-compatible extensions to established architecture documentation frameworks for AI-augmented systems, combined with a systematic regulatory compliance mapping.

### Complementarity

A key observation from this comparison is that RAD-AI is complementary to most of these approaches rather than competing with them. Model Cards and Data Cards become sub-artifacts within RAD-AI extensions (E2 and E3 respectively). HASC System Cards can serve as governance artifacts alongside RAD-AI architecture documentation. CLeAR principles can function as quality criteria for evaluating RAD-AI outputs. TechOps templates can supplement RAD-AI for compliance areas (such as training methodology) that extend beyond architecture documentation. The AWS ML Lens can guide implementation decisions that are then documented using RAD-AI extensions.

RAD-AI's unique contribution is providing the structural integration layer: it embeds AI-specific documentation into the frameworks practitioners already use, rather than creating additional standalone artifacts that risk becoming disconnected from the primary architecture documentation.

---

## Further Reading

- [arc42-extensions-reference.md](arc42-extensions-reference.md) for detailed specification of each arc42 extension (E1 through E8)
- [c4-extensions-reference.md](c4-extensions-reference.md) for detailed specification of each C4 extension (C4-E1 through C4-E3)
- [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md) for practical compliance guidance
- [adoption-guide.md](adoption-guide.md) for a staged adoption path
- [evaluation-results.md](evaluation-results.md) for evaluation methodology and results
