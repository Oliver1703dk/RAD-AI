# Research Background

**Gap analysis, related work, and research methodology behind RAD-AI**

This document provides the research foundations underlying the RAD-AI framework: the research question, the Design Science Research methodology, a comprehensive analysis of related work, the five documentation gaps that motivate the framework, the five challenges AI introduces for architecture documentation, and the relevant standards landscape.

---

## Table of Contents

1. [Research Question](#1-research-question)
2. [Methodology: Design Science Research](#2-methodology-design-science-research)
3. [Related Work Analysis](#3-related-work-analysis)
4. [Documentation Gap Analysis (G1 through G5)](#4-documentation-gap-analysis-g1-through-g5)
5. [Five Challenges AI Introduces](#5-five-challenges-ai-introduces)
6. [Standards Landscape](#6-standards-landscape)

---

## 1. Research Question

RAD-AI is motivated by a single research question:

> **RQ:** How can established software architecture documentation frameworks be systematically extended to address the unique concerns of AI-augmented ecosystems while maintaining backward compatibility and satisfying emerging regulatory requirements?

This question sits at the intersection of three domains:

- **Software architecture documentation practice.** arc42 and the C4 model dominate practitioner usage, particularly in European industry. Both assume deterministic software with stable, well-defined interfaces.
- **AI/ML engineering concerns.** AI-augmented systems introduce non-deterministic behavior, data-dependent evolution, dual training/serving lifecycles, and emergent quality attributes (fairness, explainability, drift resistance) that have no home in current documentation frameworks.
- **Regulatory compliance.** The EU AI Act (Regulation (EU) 2024/1689) mandates technical documentation through Annex IV, with enforcement for high-risk systems beginning August 2, 2026. No existing framework provides structured guidance for producing the mandated documentation.

The question deliberately scopes to *extending* established frameworks rather than proposing new ones, reflecting the design principle that practitioners are more likely to adopt incremental enhancements to tools they already use than to migrate to entirely new documentation systems.

---

## 2. Methodology: Design Science Research

### 2.1 Research Paradigm

RAD-AI follows **Design Science Research (DSR)** as described by Wieringa (2014), structured around three phases:

1. **Problem Investigation.** Systematic identification of documentation gaps through literature analysis, industry framework review, and regulatory requirement extraction.
2. **Treatment Design.** Development of backward-compatible extensions to arc42 and C4, guided by three design principles (backward compatibility, minimal disruption, traceability).
3. **Treatment Validation.** Evaluation through three complementary analytical methods (compliance coverage assessment, comparative analysis, illustrative case study).

This approach is consistent with the solution-design research strategies described by Stol and Fitzgerald (2018) in their ABC framework for software engineering research, where the goal is to design and evaluate an artifact that addresses an identified problem.

### 2.2 Gap Analysis Methodology

The gap analysis that underpins RAD-AI follows a structured three-step methodology:

**Step 1: Concern Extraction.**
AI-specific documentation concerns were extracted from four source categories:

- **Academic literature.** Systematic literature reviews on ML architecture (Nazir et al., 2024; Struber et al., 2025), expert surveys (Moin et al., 2023), MLOps mapping studies (Amou Najafabadi et al., 2024), and foundation model taxonomies (Lu et al., 2024).
- **Industry frameworks.** Standalone documentation artifacts including Model Cards (Mitchell et al., 2019), Data Cards Playbook (Google, 2022), Hazard-Aware System Cards (Sidhpurwala et al., 2025), and the CLeAR documentation principles (Chmielinski et al., 2024).
- **Regulatory requirements.** EU AI Act Annex IV (ten requirement categories spanning system description, data governance, training, risk management, human oversight, and post-market monitoring), NIST AI RMF 1.0, and ISO/IEC 42001.
- **Standards bodies.** ISO/IEC/IEEE 42010:2022 (architecture description), ISO/IEC 25059:2023 (AI quality model), and CEN-CENELEC prEN 18286 (harmonized standard, in development).

**Step 2: Gap Mapping.**
Each extracted concern was mapped to existing arc42 sections (1 through 12) and C4 diagram types (System Context, Container, Component, Code, plus supplementary Dynamic, Deployment, and System Landscape diagrams). A concern was classified as a *gap* if no existing section or diagram type could accommodate it without ad hoc modification.

**Step 3: Extension Design.**
For each identified gap, an extension was designed following the principle of minimal backward-compatible disruption. Extensions either augment an existing section with AI-specific subsections and artifacts (E1 through E7) or add a new section where no equivalent exists (E8). Each extension was required to trace to at least one identified gap (G1 through G5).

### 2.3 Evaluation Strategy

Evaluation follows the **Framework for Evaluation in Design Science (FEDS)** with three complementary methods of increasing ecological validity:

1. **EU AI Act Compliance Coverage Assessment** (analytical): Scoring ten Annex IV categories under four framework configurations.
2. **Comparative Analysis on Production AI Systems** (analytical): Applying standard and RAD-AI documentation to two well-documented production platforms and comparing concern coverage.
3. **Illustrative Ecosystem Case Study** (demonstrative): Applying RAD-AI to a smart urban mobility scenario to surface ecosystem-level concerns.

Full evaluation results are documented in [evaluation-results.md](evaluation-results.md).

---

## 3. Related Work Analysis

### 3.1 Related Work Summary

The table below positions RAD-AI relative to adjacent work spanning AI architecture research, standalone documentation artifacts, regulatory compliance mapping, and generative AI applications in software architecture.

| Work | Contribution | Limitation | Type |
|------|-------------|------------|------|
| Moin et al. (2023) | Expert survey (61 participants); finds ISO 42010 lacks AI stakeholders and viewpoints | No concrete framework extensions proposed | Survey |
| Nazir et al. (2024) | 35 challenges and 42 best practices for architecting ML-enabled systems | Decision catalogs, not documentation structures | SLR |
| Struber et al. (2025) | 16 architectural tactics and 85 quality trade-offs for ML-enabled systems | Tactics not mapped to documentation templates | SLR |
| Amou Najafabadi et al. (ECSA 2024) | 35 MLOps architecture components identified through systematic mapping | System mapping, not documentation framework | SMS |
| Lu et al. (CAIN 2024) | Foundation model taxonomy and architectural patterns | No documentation framework proposed | Taxonomy |
| Mitchell et al. (2019) | Model Cards for model-level reporting (intended uses, evaluation, bias) | Standalone artifact; not integrated into architecture documentation | Artifact |
| Google (2022) | Data Cards Playbook for structured data governance documentation | Standalone data governance; no framework integration | Playbook |
| Sidhpurwala et al. (2025) | HASC: Hazard-Aware System Cards with ASH IDs and ISO 42001 alignment | Standalone governance artifact; no framework integration | Artifact |
| Chmielinski et al. (2024) | CLeAR: four principles for AI documentation (Comprehensive, Legible, Actionable, Reproducible) | Principles, not framework extensions | Principles |
| Mamirov et al. (2025) | Automated transparency scoring pipeline for model card evaluation | Model card evaluation tool, not architecture documentation | Eval. tool |
| Sovrano et al. (2025) | AI for drafting Annex IV technical documentation | Automates generation of documentation; does not extend frameworks | Empirical |
| Bucaioni et al. (2025) | Survey of 14 AI contributions to software architecture | AI *for* architecture, not architecture *for* AI | SLR |
| Esposito et al. (2026) | Survey of generative AI applications in software architecture | Applications of GenAI to SA, not documentation extensions | Survey |
| Bucaioni et al. (2025) | Architecture-as-Code: formalizing architecture descriptions | Advances formalization but retains deterministic assumptions | Framework |
| Autili et al. (2025) | Ethical-aware reference architectures (with Inverardi, Pelliccione) | Reference architecture, not documentation extension | Ref. Arch. |

### 3.2 Analysis of Research Landscape

The related work reveals substantial progress across four directions:

**Direction 1: Identifying AI-specific architectural concerns.**
Nazir et al. (2024), Moin et al. (2023), and Struber et al. (2025) collectively provide a rich catalog of challenges, best practices, and architectural tactics for ML-enabled systems. However, these works produce decision catalogs, surveys, and tactic collections rather than documentation structures. A practitioner reading Nazir et al.'s 42 best practices receives valuable guidance on *what to decide* but no guidance on *where to document it* within an existing arc42 or C4 workflow.

**Direction 2: Proposing standalone documentation artifacts.**
Model Cards (Mitchell et al., 2019) and Data Cards (Google, 2022) represent important advances in model-level and data-level transparency. HASC (Sidhpurwala et al., 2025) extends this with machine-readable system cards incorporating safety hazard identifiers and ISO/IEC 42001 alignment. CLeAR (Chmielinski et al., 2024) proposes four documentation principles. However, all of these operate as standalone governance artifacts. They are not integrated into the architecture documentation frameworks practitioners use for day-to-day communication, review, and evolution tracking. A Model Card sitting alongside an arc42 document creates a documentation silo; RAD-AI instead embeds Model Cards as sub-artifacts of model building blocks within the architecture documentation itself.

**Direction 3: Mapping regulatory requirements.**
Sovrano et al. (2025) demonstrate that AI can assist in drafting Annex IV documentation, and Lucaj et al. (2025) propose compliance templates (TechOps). These valuable efforts focus on generating or templating compliance artifacts, but neither extends arc42 or C4 to produce the documentation that Annex IV mandates. The architecture documentation that Annex IV Section 2(c) explicitly requires ("how software components build on or feed into each other") must come from architecture frameworks, not from standalone compliance tools.

**Direction 4: AI for software architecture (rather than architecture for AI).**
Bucaioni et al. (2025) survey 14 ways AI contributes to software architecture practice (such as LLM-assisted design decisions and automated architecture recovery). Esposito et al. (2026) provide a comprehensive survey of generative AI applications in software architecture. These works address the inverse direction: using AI to improve architecture, not adapting architecture documentation to accommodate AI components.

### 3.3 The Key Distinction

Across all fifteen works surveyed, none extends the frameworks practitioners actually use (arc42 and C4). There are rich catalogs of concerns, valuable standalone artifacts, useful compliance tools, and insightful surveys of AI's impact on architecture practice. What is missing is the step of embedding these concerns directly into the documentation templates and diagram notations that practitioners already work with daily.

RAD-AI takes this complementary approach. Rather than proposing yet another standalone artifact, it augments arc42 with eight section extensions and C4 with three diagram extensions. Existing documentation remains valid. Model Cards and Data Cards are integrated as sub-artifacts rather than separate documents. Regulatory requirements map to specific framework sections rather than requiring a parallel compliance workflow.

---

## 4. Documentation Gap Analysis (G1 through G5)

The gap analysis identifies five structural documentation gaps that no existing work addresses.

### G1: No AI-Specific arc42 Sections

**Problem.** Arc42's twelve sections assume deterministic building blocks with stable interfaces. There is no designated place to document:

- Model lifecycle stages (training, evaluation, deployment, monitoring, retraining)
- Data pipelines with quality gates and schema validation
- Drift monitoring infrastructure and alerting thresholds
- Retraining automation policies and deployment strategies
- AI-specific quality attributes (fairness, explainability, drift resistance)
- ML-specific technical debt (boundary erosion, entanglement, hidden feedback loops, data dependency debt)

A practitioner documenting an AI-augmented system in arc42 must improvise, scattering AI-specific information across sections not designed for it or omitting it entirely.

**Addressed by:** E1 (AI Boundary Delineation), E2 (Model Registry View), E3 (Data Pipeline View), E4 (Responsible AI Concepts), E6 (AI Quality Scenarios), E7 (AI Debt Register), E8 (Operational AI View).

### G2: No C4 Diagram Types for ML Components

**Problem.** Standard C4 treats all containers uniformly. In a C4 Container diagram:

- A Feature Store (containing 20,000+ engineered features with lineage tracking) is visually indistinguishable from a regular PostgreSQL database.
- An ML Model serving probabilistic predictions is indistinguishable from a deterministic REST microservice.
- A monitoring agent tracking drift across 100,000+ data quality dimensions has no visual distinction from a logging service.
- There is no notation for non-determinism boundaries: the line where system behavior transitions from deterministic to probabilistic.

This means that the most architecturally significant distinction in an AI-augmented system (where probabilistic behavior enters) is invisible in the primary visualization tool.

**Addressed by:** C4-E1 (AI Component Stereotypes), C4-E2 (Data Lineage Overlay), C4-E3 (Non-Determinism Boundary Diagram), and partially E1 (AI Boundary Delineation).

### G3: No Mapping Between EU AI Act Annex IV and arc42/C4

**Problem.** Annex IV of the EU AI Act specifies nine sections of required technical documentation. Section 2(c) explicitly mandates documentation of "the system architecture explaining how software components build on or feed into each other and integrate into the overall processing." This is a direct mandate for architecture documentation.

Yet no existing architecture documentation framework provides structured guidance for producing Annex IV-compliant documentation. Practitioners face a gap between:

- What Annex IV requires (architecture documentation, data governance, training methodologies, risk management, human oversight, post-market monitoring)
- What arc42/C4 can produce (deterministic building block views, runtime sequences, deployment diagrams)

CEN-CENELEC JTC 21 is developing harmonized standards (prEN 18286), but these remain in public enquiry and do not address architecture documentation frameworks specifically. With enforcement beginning August 2, 2026, practitioners need a concrete documentation path now.

**Addressed by:** The compliance mapping table (Table V in the paper), which maps ten Annex IV requirement categories to specific RAD-AI sections, and the compliance checklist template.

### G4: No AI-Specific Architecture Decision Record Templates

**Problem.** Standard Architecture Decision Record (ADR) templates, including the widely used Markdown Any Decision Records (MADR) format, lack fields for AI-specific decision factors:

- **Model alternatives considered** (e.g., XGBoost vs. LSTM vs. linear regression, with rationale)
- **Dataset characteristics** (size, timespan, feature count, known biases)
- **Fairness and bias trade-offs** (which demographic groups are monitored, acceptable disparity thresholds)
- **Expected model lifetime** (how long before a full refit is expected)
- **Retraining triggers** (conditions under which the model should be retrained: scheduled, performance-based, drift-based)
- **Explainability requirements** (which XAI methods are used, for which stakeholders)
- **Regulatory rationale** (which AI Act risk classification applies, which articles govern transparency)

Without these fields, AI-specific decision rationale is either lost or scattered across unrelated documents. A future architect inheriting the system cannot reconstruct why a particular model was chosen, what trade-offs were accepted, or what conditions should trigger model replacement.

**Addressed by:** E5 (AI Decision Records), which extends MADR with seven AI-specific fields.

### G5: No Integration of Model/Data Cards into Architecture Documentation

**Problem.** Model Cards (Mitchell et al., 2019) and Data Cards (Google, 2022) are valuable standalone artifacts that document model-level and data-level properties. However, they exist as separate documents with no structural connection to architecture documentation.

In practice, this creates documentation silos:

- Model Cards describe model properties but not how the model fits into the architectural context (what feeds it data, what consumes its outputs, what monitors its drift).
- Data Cards describe dataset properties but not how data flows through the architectural pipeline (what transformations occur, what quality gates apply, what privacy classifications govern each stage).
- Architecture documentation describes building blocks and runtime behavior but contains no reference to the Model Cards or Data Cards that describe the AI-specific properties of those blocks.

The result is that no single documentation artifact provides a complete picture. Stakeholders must cross-reference multiple disconnected documents, and consistency between them is not enforced.

**Addressed by:** E2 (Model Registry View, which integrates Model Cards as sub-artifacts of model building blocks), E3 (Data Pipeline View, which integrates Data Cards at data source nodes), and C4-E2 (Data Lineage Overlay).

---

## 5. Five Challenges AI Introduces

AI-augmented systems differ from traditional software in five fundamental ways that current documentation frameworks cannot capture. These challenges motivate and structure the RAD-AI extensions.

### 5.1 Non-Deterministic Behavior

Traditional software produces predictable outputs for given inputs. AI components produce probabilistic outputs that vary with training data, inference conditions, and input distributions. A recommendation model may return different results for the same query depending on recently observed user behavior; a computer vision model's confidence varies with lighting conditions and image quality.

No existing architecture notation captures acceptable output ranges, confidence intervals, or degradation profiles. Standard arc42 Context and Scope (Section 3) documents system boundaries with deterministic contracts; there is no way to express "this interface returns predictions with precision >= 0.92 at P95 latency < 50ms, degrading gracefully to a rule-based fallback when confidence drops below 0.7."

RAD-AI addresses this through E1 (AI Boundary Delineation), which annotates each AI boundary crossing with a four-part contract (output type, confidence specification, update frequency, fallback behavior), and C4-E3 (Non-Determinism Boundary Diagram), which partitions the architecture into deterministic and non-deterministic regions.

### 5.2 Data-Dependent Evolution

Traditional software evolves through code commits: developers modify source code, tests verify behavior, and releases deploy changes. AI-augmented systems evolve primarily through data changes. A model's behavior shifts when the underlying data distribution shifts, even if no code has changed. Sculley et al. (2015) characterize this as "data dependency debt," noting that data dependencies are more difficult to track than code dependencies because they lack the explicit versioning and dependency management tools available for source code.

Standard arc42 Runtime View (Section 6) documents how building blocks interact at runtime, but assumes those interactions are defined by code. There is no place to document data flows with quality gates, schema validation, distribution monitoring, or the feedback loops through which inference results influence future training data.

RAD-AI addresses this through E3 (Data Pipeline View), which documents the complete ML data flow with quality gates at each stage, and C4-E2 (Data Lineage Overlay), which traces data provenance from source through transformation to model consumption.

### 5.3 Dual Lifecycle Complexity

Traditional software follows a single release lifecycle (develop, test, release, operate). AI-augmented systems follow two asynchronous lifecycles: the software release cycle and the ML model lifecycle (train, evaluate, deploy, monitor, retrain). These cycles interact but run at different cadences and with different triggers. A model may be retrained weekly based on drift detection while the surrounding software is released quarterly based on feature requirements.

Standard arc42 has no mechanism to document this dual lifecycle. Building Block View (Section 5) captures static structure; Runtime View (Section 6) captures dynamic behavior sequences. Neither accommodates the notion that an architectural component periodically replaces its own internal logic through retraining, or that deployment involves canary strategies with gradual traffic shifting rather than binary release/rollback.

RAD-AI addresses this through E2 (Model Registry View), which tracks model versions, deployment status, and last-retrained dates; E8 (Operational AI View), which documents retraining policies, deployment strategies, and rollback procedures; and E5 (AI-ADR), which captures the decision rationale for model selection including expected lifetime and retraining triggers.

### 5.4 Emergent Quality Attributes

Traditional quality attributes (performance, availability, security, maintainability) are well-established in architecture documentation. AI-augmented systems introduce additional quality attributes that are first-class architectural concerns:

- **Fairness.** The system must produce equitable outcomes across demographic groups. This requires monitoring demographic parity ratios, equalized odds, or other fairness metrics, with thresholds that represent architectural commitments rather than aspirational goals.
- **Explainability.** Stakeholders (end users, regulators, internal auditors) require understanding of why the system produces specific outputs. The choice of XAI method (SHAP, LIME, counterfactual explanations) and its integration into the architecture are design decisions.
- **Drift resistance.** The system must detect and respond to distribution shifts within defined time bounds and degradation tolerances. The monitoring infrastructure and response automation are architectural components.

Standard arc42 Quality Requirements (Section 10) uses a utility tree and quality scenario format that does not accommodate AI-specific sources (data drift, model staleness, adversarial input), environments (training vs. serving vs. monitoring), or cross-component concerns (cascading drift across ecosystem boundaries).

RAD-AI addresses this through E4 (Responsible AI Concepts), which provides a structured concern matrix, E6 (AI Quality Scenarios), which extends the standard source-stimulus-response format with AI-specific elements, and E7 (AI Debt Register), which tracks ML-specific technical debt categories.

### 5.5 Regulatory Requirements

The EU AI Act (Regulation (EU) 2024/1689), in force since August 2024, introduces binding documentation requirements through Annex IV. Key mandates include:

- **Section 2(c):** Architecture documentation explaining "how software components build on or feed into each other and integrate into the overall processing."
- **Section 2(d):** Data provenance and labelling procedures.
- **Section 2(e):** Human oversight assessment and mechanisms.
- **Section 2(h):** Cybersecurity measures for AI-specific attack vectors.

The enforcement timeline is staggered: prohibited AI practices took effect in February 2025; transparency obligations for general-purpose AI models apply from August 2025; high-risk AI system documentation requirements apply from August 2, 2026. Complementary frameworks include NIST AI RMF 1.0 (US-focused risk management) and ISO/IEC 42001 (AI management system). None of these provide structured guidance on how to produce the mandated documentation using existing architecture frameworks.

RAD-AI addresses this through the compliance mapping (which maps ten Annex IV requirement categories to specific RAD-AI sections) and through individual extensions that directly produce the documentation artifacts Annex IV demands.

---

## 6. Standards Landscape

RAD-AI operates within a broader standards landscape that shapes its design and evaluation.

### 6.1 Architecture Documentation Standards

**ISO/IEC/IEEE 42010:2022: Architecture Description.**
The international standard for architecture description, defining concepts such as architecture viewpoints, views, and stakeholders. Both arc42 and C4 align with 42010's viewpoint concepts but share the critical assumption that all building blocks represent deterministic code modules. RAD-AI's extensions are designed to be expressible as additional viewpoints within a 42010-conformant architecture description.

### 6.2 AI Quality and Management Standards

**ISO/IEC 25059:2023: Quality Model for AI Systems.**
Extends the SQuaRE quality model (ISO/IEC 25010) with AI-specific quality characteristics. RAD-AI's E6 (AI Quality Scenarios) aligns with 25059's quality model by structuring AI-specific quality requirements in a scenario-based format compatible with architectural quality analysis.

**ISO/IEC 42001:2023: AI Management System.**
Provides requirements for establishing, implementing, maintaining, and continually improving an AI management system. HASC (Sidhpurwala et al., 2025) aligns with 42001 at the governance level; RAD-AI complements this by providing the architecture documentation artifacts that a 42001-compliant management system would reference.

### 6.3 Risk Management Frameworks

**NIST AI RMF 1.0: Artificial Intelligence Risk Management Framework.**
Published by the US National Institute of Standards and Technology, the AI RMF provides a voluntary framework for managing AI risks across governance, mapping, measuring, and managing functions. RAD-AI's E7 (AI Debt Register) and E4 (Responsible AI Concepts) align with the risk identification and documentation aspects of the AI RMF.

### 6.4 Harmonized Standards (In Development)

**prEN 18286: CEN-CENELEC Harmonized Standard.**
CEN-CENELEC JTC 21 is developing prEN 18286, a harmonized standard for AI quality management systems aligned with the EU AI Act. As of early 2026, this standard remains in public enquiry. Its final form may introduce additional documentation requirements beyond those currently specified in Annex IV. RAD-AI's modular, extensible design is intended to accommodate such additional requirements through further extensions without disrupting the core framework.

### 6.5 Standards Alignment Summary

| Standard | Scope | RAD-AI Alignment |
|----------|-------|------------------|
| ISO/IEC/IEEE 42010:2022 | Architecture description | Extensions expressible as additional viewpoints |
| ISO/IEC 25059:2023 | AI quality model | E6 aligns with AI-specific quality characteristics |
| ISO/IEC 42001:2023 | AI management system | RAD-AI provides architecture artifacts referenced by 42001 |
| NIST AI RMF 1.0 | AI risk management | E4 and E7 align with risk identification and documentation |
| EU AI Act Annex IV | Technical documentation | Direct compliance mapping (10 categories) |
| prEN 18286 | Harmonized AI standard | Modular design accommodates future requirements |

---

## References

The following references are cited in this document and correspond to entries in the RAD-AI paper bibliography:

- Amershi, S. et al. (2019). Software engineering for machine learning: A case study. *Proc. ICSE-SEIP*, pp. 291-300.
- Amou Najafabadi, F. et al. (2024). An analysis of MLOps architectures: A systematic mapping study. *Proc. ECSA*, LNCS 14889, pp. 69-85.
- Autili, M. et al. (2025). A reference architecture for ethical-aware autonomous systems. *J. Syst. Softw.*, vol. 235, Art. no. 112749.
- Bogner, J. et al. (2021). Characterizing technical debt and antipatterns in AI-based systems. *Proc. TechDebt*, pp. 64-73.
- Bosch, J. et al. (2021). Engineering AI systems: A research agenda. *arXiv:2001.07522*.
- Brown, S. (2018). *The C4 Model for Visualising Software Architecture*. Leanpub.
- Bucaioni, A. et al. (2025). Artificial intelligence for software architecture: Literature review and the road ahead. *arXiv:2504.04334*.
- Bucaioni, A. et al. (2025). Architecture as code. *Proc. 22nd IEEE ICSA*, pp. 187-198.
- Chmielinski, K. et al. (2024). The CLeAR documentation framework for AI transparency. Shorenstein Center, Harvard Kennedy School.
- Clements, P. et al. (2010). *Documenting Software Architectures: Views and Beyond*, 2nd ed. Addison-Wesley.
- Esposito, M. et al. (2026). Generative AI for software architecture: Applications, challenges, and future directions. *J. Syst. Softw.*, vol. 231, Art. no. 112607.
- European Parliament and Council (2024). Regulation (EU) 2024/1689 (AI Act). *Official J. EU*.
- Google (2022). Data Cards Playbook. https://sites.research.google/datacardsplaybook/
- Lu, Q. et al. (2024). Towards a foundation model taxonomy. *Proc. CAIN*, pp. 1-10.
- Lu, Q. et al. (2024). Responsible AI pattern catalogue. *ACM Comput. Surv.*, vol. 56, no. 7, Art. no. 171.
- Lucaj, L. et al. (2025). TechOps: Technical documentation templates for the AI Act. *arXiv:2508.08804*.
- Mamirov, A. et al. (2025). AI Transparency Atlas: Framework, scoring, and real-time model card evaluation pipeline. *arXiv:2512.12443*.
- Mitchell, M. et al. (2019). Model cards for model reporting. *Proc. FAccT*, pp. 220-229.
- Moin, A. et al. (2023). Enhancing architecture frameworks by including modern stakeholders and their views/viewpoints. *arXiv:2308.05239*.
- Nazir, R. et al. (2024). Architecting ML-enabled systems: Challenges, best practices, and design decisions. *J. Syst. Softw.*, vol. 207, Art. no. 111860.
- NIST (2023). Artificial Intelligence Risk Management Framework (AI RMF 1.0).
- Sculley, D. et al. (2015). Hidden technical debt in machine learning systems. *Proc. NeurIPS*, pp. 2503-2511.
- Sidhpurwala, H. et al. (2025). Blueprints of trust: AI system cards for end-to-end transparency and governance. *arXiv:2509.20394*.
- Sovrano, F. et al. (2025). Simplifying software compliance: AI technologies in drafting technical documentation for the AI Act. *Empir. Softw. Eng.*, vol. 30, no. 4, Art. 91.
- Starke, G. (2023). arc42: The pragmatic architecture documentation template. https://arc42.org/
- Stol, K.-J. and Fitzgerald, B. (2018). The ABC of software engineering research. *ACM Trans. Softw. Eng. Methodol.*, vol. 27, no. 3, Art. no. 11.
- Struber, D. et al. (2025). Architectural tactics for ML-enabled systems: A systematic literature review. *J. Syst. Softw.*, vol. 210, Art. no. 111942.
- Wieringa, R. (2014). *Design Science Methodology for Information Systems and Software Engineering*. Springer.
