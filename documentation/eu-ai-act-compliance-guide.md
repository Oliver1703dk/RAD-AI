# EU AI Act Compliance Guide

**Using RAD-AI to satisfy Annex IV technical documentation requirements**

> This guide maps the EU AI Act's Annex IV technical documentation requirements to specific RAD-AI framework sections, providing practitioners with a concrete documentation path toward regulatory compliance.

---

## Table of Contents

1. [Overview of the EU AI Act](#1-overview-of-the-eu-ai-act)
2. [Annex IV Requirements Overview](#2-annex-iv-requirements-overview)
3. [The Compliance Gap](#3-the-compliance-gap)
4. [RAD-AI Compliance Mapping](#4-rad-ai-compliance-mapping)
5. [Compliance Coverage Assessment](#5-compliance-coverage-assessment)
6. [Risk Classification Guide](#6-risk-classification-guide)
7. [Compliance Checklist](#7-compliance-checklist)
8. [Limitations](#8-limitations)

---

## 1. Overview of the EU AI Act

Regulation (EU) 2024/1689, commonly known as the EU AI Act, is the world's first comprehensive legal framework for artificial intelligence. It entered into force on August 1, 2024, following publication in the Official Journal of the European Union. The regulation establishes harmonized rules for the placing on the market, putting into service, and use of AI systems within the European Union.

### Key Timeline

The EU AI Act follows a phased enforcement schedule:

| Date | Milestone |
|------|-----------|
| August 1, 2024 | Regulation enters into force |
| February 2, 2025 | Prohibited AI practices take effect (Title II) |
| August 2, 2025 | Transparency obligations for general-purpose AI models (Chapter V) |
| **August 2, 2026** | **High-risk AI system documentation requirements take effect (Annex IV)** |
| August 2, 2027 | Full applicability for AI systems already on market |

The August 2, 2026 deadline is the most architecturally consequential milestone. From that date, providers of high-risk AI systems must maintain technical documentation that satisfies the requirements of Annex IV before placing their systems on the EU market or putting them into service.

### Annex IV: Technical Documentation

Annex IV specifies nine sections of required technical documentation for high-risk AI systems. These sections span the full lifecycle of an AI system, from general description through post-market monitoring. The requirements are intentionally technology-neutral, specifying *what* must be documented without prescribing *how*.

A key passage from Annex IV, Section 2(c) mandates documentation of:

> "how software components build on or feed into each other and integrate into the overall processing"

This constitutes a direct mandate for architecture documentation. No existing architecture documentation framework (arc42, C4, or otherwise) was designed to produce documentation satisfying this requirement in the context of AI-augmented systems.

### Harmonized Standards

CEN-CENELEC Joint Technical Committee 21 (JTC 21) is developing harmonized standards to support the EU AI Act. The most relevant standard in development is prEN 18286, titled "Artificial intelligence: Quality management system for EU AI Act regulatory purposes." As of early 2026, prEN 18286 remains in public enquiry. Once finalized and referenced in the Official Journal, compliance with these harmonized standards will create a presumption of conformity with the corresponding EU AI Act requirements. Practitioners should monitor JTC 21 publications for updates that may introduce additional documentation requirements beyond those currently specified in Annex IV.

---

## 2. Annex IV Requirements Overview

Annex IV's nine sections translate into ten requirement categories for documentation purposes. We derive these ten categories by separating compound sections into distinct documentation concerns, following the decomposition used in the RAD-AI compliance assessment.

| # | Requirement Category | Annex IV Source | Core Documentation Demand |
|---|---|---|---|
| 1 | General system description | Section 1 | System purpose, intended use, versions, provider identity |
| 2 | System elements and development process | Section 2(a-b) | Development methods, computational resources, component inventory |
| 3 | Design specifications and architecture | Section 2(c) | Architecture description showing how components integrate |
| 4 | Data and data governance | Section 2(d) | Training data characteristics, provenance, labeling, governance measures |
| 5 | Training methodologies and techniques | Section 2(e) | Training approach, optimization techniques, hyperparameters |
| 6 | Risk assessment and management | Section 3 | Known risks, foreseeable risks, mitigation measures |
| 7 | Lifecycle change description | Section 4 | Version history, changes made throughout the system lifecycle |
| 8 | Performance metrics and accuracy | Section 5 | Performance metrics, accuracy levels, foreseeable unintended outcomes |
| 9 | Human oversight measures | Section 6 | Oversight mechanisms, override capabilities, escalation paths |
| 10 | Post-market monitoring | Section 7-9 | Monitoring system, logging, audit capabilities |

### What Annex IV Demands from Architecture Documentation

Three aspects of Annex IV are particularly relevant for software architects:

**Architecture description (Section 2(c)).** The explicit requirement to describe "how software components build on or feed into each other" means architecture documentation is not optional for compliance. This goes beyond a system context diagram; it requires documentation of component interactions, data flows, and integration points.

**Data governance (Section 2(d)).** Documentation must cover data collection processes, data preparation operations (annotation, labeling, cleaning, enrichment), data provenance, and assumptions about data adequacy. For AI systems, this extends architecture documentation into data pipeline territory.

**Human oversight (Section 6).** The regulation requires documentation of how the system is designed to support human oversight, including measures to enable human operators to correctly interpret outputs, decide not to use the system, override or reverse outputs, and intervene in or interrupt system operation.

---

## 3. The Compliance Gap

Standard architecture documentation frameworks were designed for deterministic software systems. When applied to AI-augmented systems, they leave significant portions of Annex IV requirements unaddressed.

### Standard arc42 Coverage

Standard arc42 scores **7 out of 20 (35%)** in our addressability assessment. While arc42's twelve-section template provides strong coverage for general system description (Section 1 Introduction and Goals) and partial coverage for design specifications, risk management, and lifecycle tracking, it has no sections capable of addressing:

- **Data governance (category 4):** arc42 does not include data pipeline documentation, data provenance tracking, or data quality gate specifications.
- **Training methodologies (category 5):** No section in arc42 is designed to capture ML training decisions, hyperparameter choices, or dataset characteristics.
- **Performance metrics (category 8):** While arc42 Section 10 covers quality requirements, it has no notation for probabilistic accuracy, confidence intervals, or degradation profiles.
- **Human oversight (category 9):** arc42 has no concept of human-in-the-loop components, override mechanisms, or escalation paths specific to AI systems.

### Standard C4 Coverage

Standard C4 scores **5 out of 20 (25%)** in our assessment. C4's hierarchical diagram system (System Context, Container, Component, Code) provides visual architecture descriptions, but all components are represented as generic software containers. C4 has no mechanism to:

- Distinguish AI components from traditional software components
- Represent non-deterministic behavior or confidence intervals
- Document data lineage or provenance
- Capture human oversight mechanisms

### The Critical Gap: Three Completely Unaddressed Categories

Three Annex IV requirement categories receive a score of 0 (not addressable) under both standard arc42 and standard C4:

1. **Data governance (category 4):** Neither framework includes data pipeline documentation, provenance tracking, or quality gate specifications.
2. **Training methodologies (category 5):** Neither framework provides a section for ML training decisions or dataset documentation.
3. **Human oversight (category 9):** Neither framework has notation for human-in-the-loop components or oversight mechanisms.

These three categories represent the most AI-specific documentation demands in Annex IV. They are not gaps that practitioners can address by "filling in existing sections more carefully"; they are structural absences in the frameworks themselves.

---

## 4. RAD-AI Compliance Mapping

The following table provides a detailed mapping between each Annex IV requirement category and the specific RAD-AI sections that address it. For each category, the table identifies which RAD-AI extensions apply and what practitioners should document.

| # | Annex IV Category | RAD-AI Section(s) | What to Document |
|---|---|---|---|
| 1 | General description | arc42 S1, S3 + E1 (AI Boundary Delineation) | System purpose, AI component inventory with boundary annotations, context diagram marking deterministic vs. non-deterministic boundaries |
| 2 | System elements | S5 + E2 (Model Registry View) | Model inventory with version history, ML frameworks used, development methods, computational resources in deployment view |
| 3 | Design and architecture | S5-7 + C4-E1 (AI Component Stereotypes) | Architecture diagrams using AI stereotypes (`<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, `<<Human-in-the-Loop>>`), building block decomposition |
| 4 | Data governance | E3 (Data Pipeline View) + C4-E2 (Data Lineage Overlay) | Data provenance from source to consumption, quality gates with check types and thresholds, privacy classifications, Data Card sub-artifacts |
| 5 | Training methods | E5 (AI-ADR) + supplementary artifacts | Model selection rationale with considered alternatives, dataset characteristics (size, features, time span), hyperparameter snapshots, optimization choices |
| 6 | Risk management | S11 + E7 (AI Debt Register) | Known risks and foreseeable risks, ML-specific debt categories (boundary erosion, entanglement, hidden feedback loops), severity assessment, mitigation plans |
| 7 | Lifecycle changes | E2 (version history) + E8 (Operational AI View) | Model version history with deployment dates, retraining policies (scheduled, performance-based, drift-based), deployment strategies (canary, blue-green, shadow) |
| 8 | Performance metrics | E6 (AI Quality Scenarios) + C4-E3 (Non-Determinism Boundary) | Quality scenarios with measurable response criteria, accuracy levels with confidence specifications, degradation profiles and fallback behavior |
| 9 | Human oversight | E4 (Responsible AI Concepts) + `<<Human-in-the-Loop>>` stereotype | Oversight mechanisms per component (concern matrix), override procedures, escalation paths, responsible parties, monitoring frequency |
| 10 | Post-market monitoring | E8 (Operational AI View) | Monitoring architecture (metrics per model, dashboards, alerting thresholds), drift detection mechanisms, incident response procedures, rollback policies |

### How the Extensions Work Together

No single RAD-AI extension covers all ten Annex IV categories. The extensions are designed to work in combination, with arc42 extensions providing detailed textual documentation and C4 extensions providing visual architecture representations.

For example, to satisfy category 4 (Data governance):
- **E3 (Data Pipeline View)** documents the complete data flow with quality gates at each stage, specifying check types, thresholds, and failure actions.
- **C4-E2 (Data Lineage Overlay)** provides a visual representation of data provenance, showing how data flows from sources through transformations to model consumption, with schema expectations and freshness constraints annotated.
- **Data Cards** (integrated as sub-artifacts at data source nodes in E3) capture dataset-level governance information.

Together, these three artifacts provide complementary perspectives on the same regulatory requirement: procedural (E3), visual (C4-E2), and dataset-specific (Data Cards).

---

## 5. Compliance Coverage Assessment

### Scoring Methodology

Each of the ten Annex IV requirement categories is scored for addressability under four configurations:

- **Standard arc42:** The original twelve-section arc42 template without modifications
- **Standard C4:** The original four-level C4 model without modifications
- **RAD-AI arc42:** arc42 with all eight RAD-AI extensions (E1 through E8)
- **RAD-AI C4:** C4 with all three RAD-AI extensions (C4-E1 through C4-E3)

The three-point addressability scale:
- **0 = Not addressable:** No section in the framework covers this concern. Practitioners cannot document this requirement within the framework, regardless of effort.
- **1 = Partial:** A section exists that could partially address the requirement, but lacks AI-specific detail. Documentation would require significant adaptation beyond the framework's intended use.
- **2 = Fully addressable:** A dedicated section or artifact directly addresses the requirement. Practitioners following the framework's guidance will produce documentation that satisfies this category.

### Scoring Results

| # | Category | arc42 | C4 | RAD-AI arc42 | RAD-AI C4 |
|---|---|---|---|---|---|
| 1 | General system description | 2 | 1 | 2 | 2 |
| 2 | System elements and development process | 1 | 1 | 2 | 2 |
| 3 | Design specifications and architecture | 1 | 1 | 2 | 2 |
| 4 | Data and data governance | 0 | 0 | 2 | 1 |
| 5 | Training methodologies and techniques | 0 | 0 | 1 | 1 |
| 6 | Risk assessment and management | 1 | 0 | 2 | 1 |
| 7 | Lifecycle change description | 1 | 1 | 2 | 2 |
| 8 | Performance metrics and accuracy | 0 | 1 | 2 | 2 |
| 9 | Human oversight measures | 0 | 0 | 2 | 1 |
| 10 | Post-market monitoring | 1 | 0 | 2 | 1 |
| **Total** | | **7/20 (35%)** | **5/20 (25%)** | **19/20 (95%)** | **15/20 (75%)** |

### Analysis of Results

**Standard frameworks.** Standard arc42 achieves full coverage only for general system description (category 1), with partial coverage for five additional categories. Standard C4 provides no full coverage for any category. Combined, the two standard frameworks leave three categories completely unaddressed (data governance, training methodologies, human oversight).

**RAD-AI arc42.** Achieves full addressability for nine of ten categories. The sole partial score is for training methodology documentation (category 5), which receives a score of 1 because the AI-ADR template (E5) captures model selection rationale and dataset characteristics but does not provide a dedicated section for documenting training procedures, optimization hyperparameters, or convergence criteria in full detail. These details require supplementary artifacts beyond architecture documentation.

**RAD-AI C4.** Achieves full addressability for six categories and partial addressability for four. The diagram-level extensions cannot fully capture procedural requirements (risk management processes, responsible AI governance, monitoring incident response) that require textual documentation.

**Combined RAD-AI (arc42 + C4).** When used together, RAD-AI arc42 and C4 extensions address all ten Annex IV categories. Nine categories are fully addressable; only training methodology requires supplementary artifacts beyond architecture documentation. This combined coverage provides organizations with a concrete, structured documentation path toward the August 2, 2026 compliance deadline.

---

## 6. Risk Classification Guide

The EU AI Act classifies AI systems into risk tiers, each with different documentation obligations. RAD-AI's E1 (AI Boundary Delineation) and E4 (Responsible AI Concepts) extensions are particularly useful for establishing and documenting your system's risk classification.

### High-Risk AI Systems (Article 6)

High-risk AI systems are subject to the full Annex IV documentation requirements. A system is classified as high-risk if it falls within one of the categories listed in Annex III, which includes:

- Biometric identification and categorization
- Management and operation of critical infrastructure
- Education and vocational training (access, assessment)
- Employment, workers management, and access to self-employment
- Access to essential private and public services
- Law enforcement
- Migration, asylum, and border control management
- Administration of justice and democratic processes

**RAD-AI guidance:** Use E1 (AI Boundary Delineation) to explicitly mark components that fall within Annex III categories. Annotate each high-risk boundary with the specific Annex III category and the applicable conformity assessment procedure. Use E4 (Responsible AI Concepts) to document the full concern matrix for high-risk components, including fairness metrics, explainability methods, human oversight mechanisms, and safety measures.

**Documentation obligation:** Full Annex IV technical documentation is mandatory. All ten requirement categories must be addressed. Use the compliance checklist (Section 7) to verify complete coverage.

### Limited-Risk AI Systems (Article 50, Transparency Obligations)

Limited-risk AI systems are subject to transparency obligations but not the full Annex IV documentation requirements. These include:

- AI systems intended to interact directly with natural persons (chatbots)
- Emotion recognition systems
- Biometric categorization systems
- Systems generating synthetic content (deepfakes)

**RAD-AI guidance:** Use E1 to mark components subject to transparency obligations with the specific Article 50 requirement. Document the transparency mechanism (disclosure label, watermark, or notification) in E4 (Responsible AI Concepts). While full Annex IV documentation is not legally required, organizations may choose to adopt a subset of RAD-AI extensions for internal governance purposes.

**Documentation obligation:** Transparency disclosure mechanisms must be documented. Full Annex IV documentation is not required but may be adopted voluntarily.

### Minimal-Risk AI Systems (Voluntary Codes of Conduct)

AI systems not classified as high-risk or limited-risk fall under the minimal-risk category. These systems are not subject to mandatory documentation requirements under the EU AI Act, though Article 95 encourages voluntary codes of conduct.

**RAD-AI guidance:** Even for minimal-risk systems, adopting Stage 1 of the RAD-AI adoption path (E1 and E2) provides valuable internal documentation benefits: visibility into AI component boundaries and a model inventory. This documentation incurs minimal overhead and positions teams to respond quickly if risk classifications change.

**Documentation obligation:** None mandatory. Voluntary adoption recommended for internal governance.

### Using RAD-AI to Determine Classification

When your system's risk classification is uncertain, the following RAD-AI artifacts help establish the correct tier:

1. **E1 (AI Boundary Delineation):** By marking all AI components with their intended purpose and domain, you create a systematic inventory that can be cross-referenced against Annex III categories.
2. **E4 (Responsible AI Concepts):** The concern matrix forces explicit consideration of fairness, safety, and oversight requirements, surfacing high-risk characteristics that might otherwise be overlooked.
3. **E5 (AI-ADR):** The regulatory rationale field in each AI-ADR requires documenting the applicable risk classification and the reasoning behind it.

---

## 7. Compliance Checklist

A detailed compliance checklist template with 22 items across 9 sections is available at [`../templates/compliance-checklist.md`](../templates/compliance-checklist.md). That template includes per-item RAD-AI section references and evidence fields for tracking documentation completion.

The following summary checklist provides a condensed reference for quick assessment. Each item maps to one or more RAD-AI sections.

### Section 1: General Description (4 items)

- [ ] 1.1 Intended purpose of the AI system (arc42 S1)
- [ ] 1.2 System name and version (arc42 S1)
- [ ] 1.3 External hardware/software interactions (arc42 S3 + E1)
- [ ] 1.4 Relevant software and firmware versions (E2 Model Registry View)

### Section 2: System Elements and Development (3 items)

- [ ] 2.1 Development process and methods (arc42 S5 + E2)
- [ ] 2.2 Design specifications and architecture (arc42 S5-7 + C4-E1)
- [ ] 2.3 Computational resources (arc42 S7 Deployment View)

### Section 3: Data and Data Governance (4 items)

- [ ] 3.1 Training data characteristics and governance (E3 + C4-E2)
- [ ] 3.2 Data collection and provenance (C4-E2 Data Lineage Overlay)
- [ ] 3.3 Data preparation processes (E3 Data Pipeline View)
- [ ] 3.4 Assumptions about training data (E5 AI-ADR)

### Section 4: Training Methodology (2 items)

- [ ] 4.1 Training methodologies and techniques (E5 AI-ADR + supplementary)
- [ ] 4.2 Training choices and optimization (E5 AI-ADR + supplementary)

### Section 5: Risk Assessment (2 items)

- [ ] 5.1 Known and foreseeable risks (arc42 S11 + E7)
- [ ] 5.2 Risk management measures (E7 AI Debt Register + C4-E3)

### Section 6: Lifecycle Changes (1 item)

- [ ] 6.1 System changes throughout lifecycle (E2 version history + E8)

### Section 7: Performance and Accuracy (2 items)

- [ ] 7.1 Performance metrics (E6 AI Quality Scenarios + E2)
- [ ] 7.2 Accuracy levels and foreseeable unintended outcomes (E6 + C4-E3)

### Section 8: Human Oversight (2 items)

- [ ] 8.1 Human oversight measures (E4 + `<<Human-in-the-Loop>>` stereotype)
- [ ] 8.2 System support for human oversight (E4 Responsible AI Concepts)

### Section 9: Post-Market Monitoring (2 items)

- [ ] 9.1 Post-market monitoring system (E8 Operational AI View)
- [ ] 9.2 Logging and audit capabilities (E8 + E4)

### Using the Checklist

1. Copy the detailed checklist from [`../templates/compliance-checklist.md`](../templates/compliance-checklist.md) into your project.
2. Fill in your system name, risk classification, and assessment date.
3. For each item, complete the corresponding RAD-AI section using the templates from [`../templates/`](../templates/).
4. Mark each checklist item as complete when the documentation is in place.
5. Review the completed checklist with your compliance or legal team.
6. Schedule periodic reviews (at minimum annually, and after each significant system change).

---

## 8. Limitations

This compliance guide and the underlying RAD-AI compliance mapping are subject to several important limitations:

### Coverage Limitations

**Training methodology (category 5) remains partially addressed.** The AI-ADR template (E5) captures model selection rationale, dataset characteristics, and high-level training decisions, but it does not provide a dedicated, detailed section for training procedures, convergence analysis, or full hyperparameter documentation. Organizations subject to Annex IV should supplement RAD-AI with dedicated ML experiment tracking artifacts (for example, from MLflow or Weights and Biases experiment logs) to fully satisfy this category.

**Combined coverage is required.** The 95% addressability figure applies to RAD-AI arc42 extensions specifically. C4 extensions alone achieve 75%. Full coverage requires using both arc42 and C4 extensions in combination. Organizations using only C4 diagrams will need to supplement with textual documentation for procedural requirements.

### Validation Limitations

**Author-assessed scoring.** The addressability scores in Section 5 were assessed by the RAD-AI authors. Independent validation by regulatory experts, legal professionals, and practitioners would strengthen the reliability of these scores. A multi-rater study with inter-rater reliability measures is planned for future work.

**No legal precedent.** As of early 2026, no enforcement actions have been taken under Annex IV, so there is no regulatory precedent for what constitutes "sufficient" technical documentation. The mapping in this guide represents our best interpretation of the regulatory text.

### Regulatory Limitations

**Harmonized standards in development.** CEN-CENELEC JTC 21's prEN 18286 is still in public enquiry. Once finalized and published in the Official Journal, these harmonized standards may introduce additional requirements, modify existing ones, or provide more specific guidance that supersedes or supplements the current Annex IV text. Practitioners should monitor JTC 21 publications.

**Jurisdiction-specific interpretation.** National authorities in EU member states may issue guidance that interprets Annex IV requirements differently. This guide is based on the regulation text as published in the Official Journal and does not account for member-state-specific interpretations.

### Scope Limitations

**Architecture documentation, not legal advice.** RAD-AI provides documentation structure and guidance for producing technical documentation. It does not constitute legal advice. Organizations should consult qualified legal counsel for compliance strategy, particularly for the conformity assessment procedures required for high-risk AI systems.

**Classical ML focus.** The current RAD-AI extensions target classical ML pipelines (training, serving, monitoring). LLM and foundation model systems introduce additional documentation concerns (prompt versioning, retrieval-augmented generation pipelines, guardrail architectures, fine-tuning provenance) that may require further extensions. These are identified as a priority direction for future RAD-AI development.

---

## References

- European Parliament and Council, "Regulation (EU) 2024/1689 laying down harmonised rules on artificial intelligence (AI Act)," *Official J. EU*, 2024.
- CEN-CENELEC JTC 21, "prEN 18286: Artificial intelligence: Quality management system for EU AI Act regulatory purposes," 2025.
- Larsen, O. A. and Moghaddam, M. T., "RAD-AI: Rethinking Architecture Documentation for AI-Augmented Ecosystems," in *Proc. ANGE 2026 Workshop at IEEE ICSA*, 2026.
- Sovrano, F., Hine, E., Anzolut, S., and Bacchelli, A., "Simplifying software compliance: AI technologies in drafting technical documentation for the AI Act," *Empir. Softw. Eng.*, vol. 30, no. 4, Art. 91, 2025.
- Lucaj, L., Loosley, A., Jonsson, H., Gasser, U., and van der Smagt, P., "TechOps: Technical documentation templates for the AI Act," *arXiv:2508.08804*, 2025.

---

*This guide is part of the RAD-AI framework. For the full framework documentation, see [framework-overview.md](framework-overview.md). For adoption guidance, see [adoption-guide.md](adoption-guide.md).*
