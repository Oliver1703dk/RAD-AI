# Evaluation Results

**Three complementary analytical methods evaluating RAD-AI**

This document presents the evaluation methodology and results for RAD-AI, covering three complementary methods of increasing ecological validity: a compliance coverage assessment against the EU AI Act Annex IV, a comparative analysis on two production AI platforms, and an illustrative ecosystem case study. It concludes with an evaluation synthesis, threats to validity, and planned future evaluation.

---

## Table of Contents

1. [Evaluation Strategy](#1-evaluation-strategy)
2. [Evaluation 1: Compliance Coverage Assessment](#2-evaluation-1-compliance-coverage-assessment)
3. [Evaluation 2: Comparative Analysis on Production AI Systems](#3-evaluation-2-comparative-analysis-on-production-ai-systems)
4. [Evaluation 3: Illustrative Ecosystem Case Study](#4-evaluation-3-illustrative-ecosystem-case-study)
5. [Evaluation Synthesis](#5-evaluation-synthesis)
6. [Threats to Validity](#6-threats-to-validity)
7. [Planned Future Evaluation](#7-planned-future-evaluation)

---

## 1. Evaluation Strategy

### 1.1 Design Science Evaluation Context

RAD-AI was developed through Design Science Research (Wieringa, 2014): Problem Investigation, Treatment Design, Treatment Validation. The evaluation phase must demonstrate that the designed artifact (the RAD-AI extensions) addresses the identified problem (documentation gaps G1 through G5) with sufficient rigor for the current stage of research.

### 1.2 FEDS Framework

The evaluation follows the **Framework for Evaluation in Design Science (FEDS)**, which recommends selecting evaluation methods based on the artifact's maturity and the desired level of ecological validity. For a framework at the initial design stage, analytical evaluation methods are appropriate before progressing to empirical methods with practitioners.

### 1.3 Three Complementary Methods

The three evaluation methods are ordered by increasing ecological validity:

| Method | Type | What It Assesses | Ecological Validity |
|--------|------|-----------------|---------------------|
| Compliance coverage assessment | Analytical + practitioner | Regulatory addressability improvement | Medium (six practitioners, Fleiss' kappa approximately 0.68) |
| Comparative analysis | Analytical | Concern coverage on real systems | Medium (real production platforms) |
| Ecosystem case study | Demonstrative | Ecosystem-level concern visibility | Medium (synthetic but realistic) |

The use of multiple methods provides triangulation: if independent methods converge on the same findings, confidence in the results is strengthened even at an analytical evaluation stage.

---

## 2. Evaluation 1: Compliance Coverage Assessment

### 2.1 Objective

Assess whether RAD-AI extensions improve the ability of arc42 and C4 to address EU AI Act Annex IV documentation requirements.

### 2.2 Method

**Requirement categories.** Ten requirement categories were derived from Annex IV's nine sections, covering: general system description, system elements and development process, design specifications and architecture, data and data governance, training methodologies and techniques, risk assessment and management, lifecycle change description, performance metrics and accuracy, human oversight measures, and post-market monitoring.

**Framework configurations.** Each category was scored under four configurations:
1. Standard arc42 (unextended)
2. Standard C4 (unextended)
3. RAD-AI-extended arc42
4. RAD-AI-extended C4

**Scoring scale.** A three-point addressability scale was used:
- **0 (not addressable):** No framework section covers this concern. A practitioner using the framework has no designated place to produce this documentation.
- **1 (partial):** A section exists that could accommodate some aspects of this concern, but it lacks AI-specific structure and guidance. The practitioner must improvise.
- **2 (fully addressable):** A dedicated section or artifact directly addresses the requirement with AI-specific structure, fields, and guidance.

**Assessors.** Six experienced software-architecture practitioners (domain experts) independently scored each configuration. Scores per category represent the modal rating across participants; total scores represent mean values across raters. Inter-rater reliability was assessed using Fleiss' kappa (approximately 0.68), indicating substantial agreement.

### 2.3 Results

The full scoring matrix is presented below. Individual category scores represent the modal rating across six practitioners; total scores reflect mean values across raters (standard deviations reported in the analysis).

| # | Annex IV Requirement Category | arc42 | C4 | RAD-AI arc42 | RAD-AI C4 |
|---|-------------------------------|-------|----|--------------|-----------|
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
| | **Total (mean, out of 20)** | **7.3** | **5.2** | **18.5** | **14.6** |
| | **Addressability (mean)** | **36%** | **26%** | **93%** | **73%** |

### 2.4 Analysis

**Standard framework scores.**
- Standard arc42 averaged 7.3/20 (36%, sigma approximately 0.5). It fully covers general system description (Section 1 provides introduction and goals) and partially addresses five categories: system elements (Section 5 Building Block View), design specifications (Sections 5 through 7), risk management (Section 11 Risks and Technical Debt), lifecycle changes (version tracking in Section 5), and post-market monitoring (operational concerns in Section 12). It scores zero on data governance, training methodologies, performance metrics, and human oversight.
- Standard C4 averaged 5.2/20 (26%, sigma approximately 0.4). It provides partial coverage through its hierarchical diagram system (general description, system elements, design, lifecycle, and performance through visual notation) but scores zero on data governance, training methodologies, risk management, human oversight, and post-market monitoring. Diagram-level notation cannot express procedural requirements.

**RAD-AI extended scores.**
- RAD-AI-extended arc42 averaged 18.5/20 (93%, sigma approximately 0.5). The sole partial category is training methodology documentation (category 5). Architecture documentation can capture training decisions (through AI-ADR) and training infrastructure (through Data Pipeline View), but detailed training methodology documentation (hyperparameter search spaces, cross-validation strategies, training convergence criteria) requires supplementary artifacts beyond architecture documentation.
- RAD-AI-extended C4 averaged 14.6/20 (73%, sigma approximately 0.6). Diagram-level extensions fully address six categories and partially address four. The partial scores reflect that diagrammatic notation cannot fully express procedural requirements (risk management processes, human oversight procedures, post-market monitoring workflows, data governance policies). These require textual documentation in arc42 sections.

**Categories completely unaddressable by standard frameworks.**
Three Annex IV categories receive a score of 0 under both standard arc42 and standard C4:
1. **Data and data governance** (category 4): No section in either framework accommodates data provenance, labelling procedures, data quality monitoring, or bias assessment.
2. **Training methodologies and techniques** (category 5): No section documents model training approaches, evaluation methodologies, or the rationale for algorithmic choices.
3. **Human oversight measures** (category 9): No section documents how humans monitor, intervene in, or override AI component behavior.

These three categories represent the most AI-specific documentation demands of Annex IV. RAD-AI extensions map directly to each: E3 and C4-E2 for data governance, E5 for training methodology rationale, and E4 for human oversight.

**Combined RAD-AI coverage.**
When both arc42 and C4 extensions are used together (as intended), the combined RAD-AI framework addresses all but one Annex IV category in the practitioner-based evaluation, offering organizations a concrete documentation path toward the August 2, 2026 compliance deadline.

**Significance.**
To our knowledge, this is the first systematic addressability assessment of architecture documentation frameworks against EU AI Act requirements. Prior work has mapped AI Act requirements to general SE artifacts (Sovrano et al., 2025) or proposed compliance templates (Lucaj et al., 2025), but none has scored the specific addressability of arc42 and C4 against Annex IV requirement categories. The use of six independent practitioner raters with substantial inter-rater agreement (Fleiss' kappa approximately 0.68) strengthens the reliability of these findings beyond author-assessed scoring.

---

## 3. Evaluation 2: Comparative Analysis on Production AI Systems

### 3.1 Objective

Assess whether documentation gaps are structural properties of the frameworks (appearing consistently across domains) or domain-specific oversights, and quantify the improvement RAD-AI provides.

### 3.2 Method

**System selection criteria.** Two production AI platforms were selected based on three criteria:
1. Sufficiently well-documented through public engineering blogs, academic publications, and technical talks to enable meaningful architectural documentation.
2. From different application domains to test cross-domain consistency.
3. Operating at production scale with mature ML infrastructure.

**Documentation process.** For each system:
1. Document the system using standard arc42/C4, following established templates.
2. Identify AI-specific concerns that remain undocumented or inadequately captured.
3. Apply RAD-AI extensions and assess which additional concerns are captured.

**Assessment instrument.** A ten-item concern coverage matrix (Table below) was used to evaluate each system under standard and RAD-AI frameworks. Concerns were scored as:
- Fully captured: A designated section or diagram element directly and completely documents this concern.
- Partially captured: Some aspects are documentable but the framework lacks AI-specific structure.
- Not captured: No framework section accommodates this concern.

### 3.3 System 1: Uber Michelangelo

**System overview.**
Uber Michelangelo is Uber's end-to-end ML platform managing over 5,000 production models and serving 10 million predictions per second at peak. Key architectural components include: a centralized Feature Store (20,000+ features), the Gallery model management system with four-stage model lifecycle and rule-based deployment automation, and the D3 drift detection system (100,000+ data quality monitors).

**Concerns invisible under standard arc42/C4.**
Three architectural concerns are invisible when documenting Michelangelo with standard frameworks:

1. **Feature Store indistinguishable from generic database.** The Feature Store manages 20,000+ engineered features with lineage tracking, versioning, and cross-team sharing. In a standard C4 Container diagram, it appears as a database container, visually identical to Cassandra or any other data store. The architectural significance of the Feature Store (its role in ensuring feature consistency between training and serving, its cross-team sharing semantics, its lineage tracking) is invisible.

2. **Gallery lifecycle collapses into static building blocks.** Gallery's four-stage model lifecycle (exploration, training, evaluation, production) with rule-based deployment automation (e.g., `WHEN metrics[mae] <= 5`) is a core architectural pattern. In standard arc42 Building Block View (Section 5), Gallery appears as a static building block. The lifecycle stages, the transition rules, the version management, and the automated promotion criteria are undocumentable.

3. **D3 drift detection has no home.** The D3 system tracks 100,000+ data quality monitors and reduced Uber's time-to-detect data quality issues from 45 days to 2 days. In standard arc42 Runtime View (Section 6), there is no place to document drift monitoring infrastructure, alerting thresholds, detection latencies, or the relationship between drift detection and retraining triggers. D3's monitoring infrastructure is architecturally significant but has no designated documentation location.

**How RAD-AI addresses each gap.**
- The `<<Feature Store>>` stereotype (C4-E1) visually distinguishes the Feature Store from regular databases in C4 diagrams.
- The Model Registry View (E2) captures Gallery's versioned lifecycle, deployment statuses, and rule-based promotion criteria.
- The Operational AI View (E8) documents D3's monitoring infrastructure, alerting thresholds, and retraining trigger policies.
- The Non-Determinism Boundary Diagram (C4-E3) separates deterministic serving infrastructure from probabilistic ML inference.

### 3.4 System 2: Netflix Metaflow/Maestro

**System overview.**
Netflix's ML infrastructure supports over 3,000 ML projects through Metaflow (Python-native ML workflow framework) and Maestro (cross-workflow orchestration). The platform processes 15,000+ Flink streaming jobs alongside batch workloads, with A/B testing conducted through the ABlaze platform.

**Concerns invisible under standard arc42/C4.**
Four architectural concerns are undocumentable under standard frameworks:

1. **Metaflow DAGs invisible in Runtime View.** Metaflow's Python-native directed acyclic graphs (DAGs) with event-triggered chaining (`@trigger_on_finish`) define how data flows through ML workflows. Standard arc42 Runtime View (Section 6) documents runtime sequences between building blocks, but cannot express DAG-structured ML workflows with conditional branching, retry logic, and event-triggered execution.

2. **Maestro cross-workflow coordination undocumentable.** Maestro's signal-based coordination, where one step's output unblocks dependent steps across different workflows, has no documentation counterpart in standard arc42 or C4. This cross-workflow dependency pattern is architecturally critical (failure in one workflow can cascade to block dependent workflows) but invisible in standard documentation.

3. **Streaming vs. batch distinction lost.** Netflix operates 15,000+ Flink streaming jobs alongside batch processing. The architectural distinction between streaming and batch pipelines (different latency guarantees, different failure semantics, different scaling characteristics) is collapsed in standard C4, where both appear as generic containers.

4. **A/B testing infrastructure undocumented.** The ABlaze platform governs model experimentation, traffic allocation, and statistical significance testing. In standard arc42, there is no section for documenting experimentation infrastructure, and A/B testing does not fit into Quality Requirements (Section 10), Architecture Decisions (Section 9), or Runtime View (Section 6).

**How RAD-AI addresses each gap.**
- `<<ML Model>>` and `<<Data Pipeline>>` stereotypes (C4-E1) visually distinguish ML workflow components from general-purpose services.
- The Data Lineage Overlay (C4-E2) traces data from ingestion through Flink streaming and batch processing to model consumption, making streaming/batch distinctions explicit.
- AI-ADR (E5) provides a structured format for documenting experiment tracking decisions, including model alternatives and evaluation criteria.
- The Operational AI View (E8) documents A/B testing infrastructure, canary deployment strategies, and traffic allocation policies.

### 3.5 Results: Ten-Item Concern Coverage Matrix

| AI-Specific Architectural Concern | Uber std. | Uber RAD-AI | Netflix std. | Netflix RAD-AI |
|-----------------------------------|-----------|-------------|--------------|----------------|
| Model versioning and lifecycle management | Not captured | Fully captured | Not captured | Fully captured |
| Feature store architecture and sharing | Not captured | Fully captured | Not captured | Fully captured |
| Data pipeline with quality gates | Partial | Fully captured | Partial | Fully captured |
| Drift detection and monitoring | Not captured | Fully captured | Not captured | Fully captured |
| Retraining triggers and automation | Not captured | Fully captured | Not captured | Fully captured |
| Non-deterministic behavior boundaries | Not captured | Fully captured | Not captured | Fully captured |
| A/B testing and canary model deployment | Not captured | Partial | Not captured | Fully captured |
| Data lineage and provenance | Not captured | Fully captured | Partial | Fully captured |
| ML-specific technical debt tracking | Partial | Fully captured | Not captured | Partial |
| Responsible AI and fairness concerns | Not captured | Partial | Not captured | Partial |
| **Summary** | **0 full, 2 partial** | **8 full, 2 partial** | **0 full, 2 partial** | **8 full, 2 partial** |

### 3.6 Key Findings

**Finding 1: Cross-domain consistency of gaps.**
The most notable finding is the identical gap pattern across both systems. Despite serving fundamentally different domains (marketplace logistics vs. content recommendation), both systems exhibit the same pattern under standard frameworks: 0 concerns fully captured, 2 partially captured (data pipeline and one other). This consistency strongly suggests that the documentation deficiencies are structural properties of the frameworks themselves rather than domain-specific oversights.

**Finding 2: Cross-domain consistency of improvement.**
The improvement under RAD-AI is also identical across both systems: 8 concerns fully captured, 2 partially. The two remaining partial scores are the same in both cases: ML-specific technical debt tracking and responsible AI/fairness concerns. These partial scores reflect that technical debt and fairness are domains where architecture documentation can provide structure (through E7 and E4) but cannot be fully comprehensive without supplementary artifacts (dedicated fairness audit reports, detailed debt remediation plans).

**Finding 3: Scale of invisible infrastructure.**
Both organizations have invested heavily in model management, drift detection, and deployment governance. Uber's D3 system monitors 100,000+ data quality dimensions; Netflix's Metaflow supports 3,000+ ML projects with sophisticated orchestration. Yet standard arc42/C4 provides no place to record any of this infrastructure. The gap is not a matter of documentation effort or discipline; it is a structural limitation of frameworks designed for deterministic systems.

**Finding 4: A/B testing asymmetry.**
The one difference between systems is A/B testing coverage. Under RAD-AI, Netflix achieves "fully captured" (ABlaze is well-documented and maps cleanly to Operational AI View), while Uber achieves only "partial" (Uber's experimentation infrastructure is less publicly documented, limiting the completeness of RAD-AI documentation). This asymmetry reflects a limitation of the comparative analysis method (reliance on public documentation) rather than a limitation of RAD-AI itself.

### 3.7 Replication Package

Complete side-by-side documentation (standard arc42/C4 and RAD-AI extended) for both systems is available in the repository's `comparative_analysis/` directory, enabling direct inspection of what each framework captures and misses. See:
- `comparative_analysis/uber-michelangelo/`
- `comparative_analysis/netflix-metaflow/`

---

## 4. Evaluation 3: Illustrative Ecosystem Case Study

### 4.1 Objective

Demonstrate that RAD-AI surfaces ecosystem-level concerns that are structurally invisible in standard architecture documentation, aligning with the ANGE workshop's focus on next-generation ecosystems.

### 4.2 Scenario: Smart Urban Mobility Ecosystem

The case study applies RAD-AI to a smart urban mobility scenario integrating four AI components across multiple transit operators:

1. **Route Optimization Service.** ML-based continuous retraining on real-time traffic patterns. Uses gradient-boosted trees (XGBoost), selected over LSTM for explainability in a public-service context.
2. **Demand Prediction Engine.** Time-series forecasting across bus, tram, and bike-sharing operators. Provides demand signals that feed route optimization.
3. **Anomaly Detection System.** Real-time safety monitoring of the transport network. Potentially classified as high-risk under the EU AI Act (safety component of critical infrastructure).
4. **Cross-Operator Data Sharing Platform.** Federated feature store enabling anonymized data exchange across operators. Shared infrastructure with multi-stakeholder governance.

**Architectural dependencies.** Demand predictions feed route optimization. Anomaly detection can trigger route recomputation. All systems consume features from the shared federated feature store.

### 4.3 Standard Documentation Gaps

Under standard arc42/C4, these four components appear as generic containers. The following ecosystem-level concerns are invisible:

- The federated feature store is indistinguishable from a shared database.
- EU AI Act risk classifications (anomaly detection as high-risk vs. route optimization as limited-risk) are not expressible.
- Cross-system data dependencies are undocumented.
- Model versioning across operators creates silent compatibility risks.
- The dual lifecycle challenge is amplified at ecosystem scale: each operator's models retrain independently but all consume shared features.

### 4.4 Ecosystem-Level Concerns Revealed by RAD-AI

**Cascading drift.**
When the anomaly detection model experiences data drift (e.g., feature distribution shift exceeding 2 sigma on three or more features sustained for more than 12 hours), it begins generating false safety alerts. These false alerts trigger unnecessary rerouting in the route optimization service, degrading user experience across the transit network. The Data Lineage Overlay (C4-E2) visualizes this dependency chain across operator boundaries, making the cascading risk explicit. The AI Quality Scenario (E6) documents the expected response: anomaly detection tightens its confidence threshold from 0.85 to 0.95; route optimization activates a last-known-good fallback; drift is detected within 2 hours; false safety alerts remain below 5%; downstream route MAE stays below 7 minutes.

**Differentiated compliance.**
The anomaly detection system, as a safety component of critical infrastructure, is potentially high-risk under the EU AI Act and requires full Annex IV documentation. The route optimization service, while important, may fall outside the high-risk classification. AI Boundary Delineation (E1) distinguishes high-risk from limited-risk components in the ecosystem diagram, enabling targeted compliance documentation where legally required rather than applying uniform documentation overhead across all components.

**Federated governance.**
The shared feature store spans multiple transit operators, each with different data governance policies, privacy constraints, and organizational accountability structures. Responsible AI Concepts (E4) documents the shared feature store ownership model, cross-operator data agreements, and accountability boundaries. This governance structure is invisible in standard documentation, where the feature store appears as a single shared database without governance annotations.

**Concrete AI-ADR.**
The route optimization service's model selection (gradient-boosted trees over LSTM for explainability in a public-service context) is documented through an AI-ADR (E5) that captures: model alternatives considered, dataset characteristics (18 months, 2.1M trips, GPS and weather data), fairness monitoring (under-served districts, max 15% prediction gap), expected model lifetime (approximately 12 months), retraining trigger (MAE exceeding 5.5 minutes for 3 consecutive days), explainability method (SHAP values for per-route regulatory audits), and regulatory classification (not high-risk, transparency per Art. 50).

### 4.5 Significance

The case study demonstrates that RAD-AI surfaces three categories of ecosystem-level concerns (cascading drift, differentiated compliance, federated governance) that are structurally invisible in standard architecture documentation. These concerns emerge only when AI components interact across organizational boundaries with shared infrastructure, making them particularly relevant to the ANGE workshop's focus on next-generation ecosystems.

Full case study documentation is available in:
- `examples/smart-urban-mobility/`

---

## 5. Evaluation Synthesis

### 5.1 Convergence of Findings

The strongest finding across all three evaluation methods is the **structural consistency of documentation gaps**. The compliance assessment and comparative analysis independently converge on the same deficiency pattern:

- The compliance assessment identifies three Annex IV categories completely unaddressable by standard frameworks (data governance, training methodologies, human oversight). These are the most AI-specific documentation demands.
- The comparative analysis finds identical gap patterns across two unrelated production domains (0 of 10 concerns fully captured, 2 partially). The gaps appear in the same categories regardless of application domain.
- The ecosystem case study reveals additional concerns (cascading drift, differentiated compliance, federated governance) that are invisible not merely because of missing AI-specific sections but because standard frameworks lack notation for cross-component, cross-organization AI interactions.

This convergence from independent methods strengthens confidence that the documentation deficiencies are structural properties of the frameworks rather than artifacts of the evaluation method.

### 5.2 Quantified Improvement

Compliance scores are means across six practitioner raters (Fleiss' kappa approximately 0.68).

| Metric | Standard Frameworks | RAD-AI | Improvement |
|--------|-------------------|--------|-------------|
| Annex IV coverage (arc42) | 36% (7.3/20) | 93% (18.5/20) | +57 percentage points |
| Annex IV coverage (C4) | 26% (5.2/20) | 73% (14.6/20) | +47 percentage points |
| AI concerns captured, Uber | 0 full, 2 partial | 8 full, 2 partial | +8 fully captured |
| AI concerns captured, Netflix | 0 full, 2 partial | 8 full, 2 partial | +8 fully captured |
| Ecosystem-level concerns surfaced | 0 | 3 | +3 concerns visible |

### 5.3 What the Evidence Supports

The three evaluation methods provide converging *analytical* evidence that RAD-AI addresses the identified documentation gaps. Specifically, the evidence supports the following claims:

1. Standard arc42 and C4 have structural gaps for AI-specific documentation concerns (demonstrated by compliance assessment and cross-domain comparative analysis).
2. RAD-AI extensions address the majority of these gaps (demonstrated by scoring improvements across all evaluation methods).
3. Ecosystem-level concerns require documentation notation that goes beyond individual AI components (demonstrated by the case study).

### 5.4 What the Evidence Does Not Support

The evaluation does not support claims about:

- **Practitioner usability.** Whether software architects find RAD-AI templates usable, learnable, or efficient in practice (time-to-document, SUS/TAM) has not been tested beyond addressability scoring.
- **Documentation quality.** Whether RAD-AI-produced documentation is more useful to downstream stakeholders (developers, auditors, regulators) has not been measured.
- **Adoption cost.** The effort required to adopt RAD-AI extensions in real projects is unknown.
- **Regulatory acceptance.** Whether Annex IV compliance assessors would accept RAD-AI-produced documentation has not been tested.

While the compliance assessment was conducted by six independent practitioners with substantial inter-rater agreement, the sample size remains small and broader replication is needed.

---

## 6. Threats to Validity

### 6.1 Construct Validity

**Annex IV decomposition.** The ten requirement categories and three-point addressability scale reflect the authors' interpretation of Annex IV. Alternative decompositions (e.g., finer-grained categories, different scoring scales) could yield different absolute scores. The relative improvement (standard vs. RAD-AI) would likely persist under alternative decompositions, but the specific percentages (36% to 93%) should be interpreted as indicative rather than precise.

**Concern coverage matrix.** The ten AI-specific concerns in the comparative analysis were selected based on the gap analysis (G1 through G5) and the literature on AI architecture challenges. A different selection of concerns could change the absolute coverage counts. The selection was designed to span the full range of AI-specific documentation needs identified in the literature.

### 6.2 Internal Validity

**DSR circularity.** The gap analysis (G1 through G5) informed both the design of RAD-AI and the evaluation instruments. This circularity is inherent to Design Science Research: the artifact is evaluated against the problem it was designed to solve. The comparative analysis on independently documented production systems partially mitigates this concern, because the systems were not selected or documented to favor RAD-AI.

**Practitioner sample size.** The compliance coverage assessment was scored by six independent practitioners, yielding substantial inter-rater agreement (Fleiss' kappa approximately 0.68). While this mitigates author bias compared to author-only assessment, the sample size is small. The concern coverage matrix for the comparative analysis was assessed by the authors. A larger multi-rater study would further strengthen the evaluation and is planned for the extended journal version.

### 6.3 External Validity

**Domain coverage.** Two production systems from two domains (marketplace logistics and content recommendation) and one synthetic ecosystem (smart urban mobility) provide limited evidence for generalizability. The cross-domain consistency of gap patterns provides some evidence that findings are not domain-specific, but additional domains (healthcare, finance, autonomous vehicles, manufacturing) would strengthen external validity.

**Public documentation reliance.** The comparative analysis relies on publicly available documentation (engineering blogs, academic publications, technical talks). Internal architectural documentation at Uber and Netflix may address concerns that appear as gaps in the public documentation. The analysis reflects what is publicly documentable about these systems, not necessarily the complete internal architectural knowledge.

**Synthetic ecosystem.** The smart urban mobility case study is a synthetic scenario designed to illustrate ecosystem-level concerns. While realistic (based on real transit system architectures and AI applications), it does not carry the weight of evidence from a deployed system. The ecosystem-level findings (cascading drift, differentiated compliance, federated governance) require validation on real multi-stakeholder AI ecosystems.

### 6.4 Regulatory Validity

**Evolving standards.** The EU AI Act is in force, but harmonized standards (prEN 18286) are still under development by CEN-CENELEC JTC 21. The final harmonized standard may introduce additional documentation requirements beyond those currently specified in Annex IV. RAD-AI's modular design is intended to accommodate such requirements, but the compliance mapping may require updates when harmonized standards are finalized.

**Interpretation uncertainty.** Annex IV's requirements are written in regulatory language that admits interpretation. For example, what constitutes adequate documentation of "how software components build on or feed into each other" (Section 2(c)) is not precisely defined. The compliance mapping reflects one reasonable interpretation; regulatory guidance and enforcement practice may refine the requirements.

---

## 7. Planned Future Evaluation

### 7.1 Controlled Practitioner Study

The primary planned evaluation is a controlled study with 10 to 15 software architects, designed for an extended journal version:

**Participants.** Software architects with experience in both traditional and AI-augmented systems, recruited from industry and academia.

**Task.** Each participant documents a provided AI system using standard arc42/C4 and then using RAD-AI extensions (within-subjects design with counterbalancing to mitigate ordering effects).

**Measures.**
- **Completeness.** Percentage of AI-specific concerns captured in the produced documentation, scored against a reference checklist.
- **Time-to-document.** Time required to produce documentation under each framework configuration.
- **Perceived usefulness.** System Usability Scale (SUS) and Technology Acceptance Model (TAM) questionnaires administered after each documentation session.

**Analysis.** Paired comparisons (Wilcoxon signed-rank for small samples) between standard and RAD-AI conditions on all three measures.

### 7.2 Expanded Multi-Rater Reliability Study

The current evaluation includes six practitioners with Fleiss' kappa approximately 0.68 (substantial agreement), meeting the target threshold of kappa >= 0.61. To further strengthen the findings:

**Design.** Expand to 10--15 independent raters (software architects or AI engineers not involved in RAD-AI development) independently scoring the compliance assessment and concern coverage matrices.

**Measure.** Inter-rater reliability assessed using Fleiss' kappa, with the goal of confirming substantial agreement at a larger sample size and across a broader range of practitioner backgrounds.

**Purpose.** Strengthen the reliability of the scoring and assess whether findings generalize across a wider practitioner population.

### 7.3 Additional Planned Directions

- **Tooling evaluation.** Assess Structurizr DSL extensions implementing RAD-AI stereotypes and diagram types.
- **Compliance assessment with practitioners.** Evaluate whether Annex IV compliance assessors find RAD-AI-produced documentation sufficient.
- **LLM/foundation model extension.** Extend RAD-AI for documentation concerns specific to LLM systems (prompt versioning, retrieval-augmented generation pipelines, guardrail architectures, fine-tuning provenance) and evaluate on production LLM deployments.

---

## References

The following references are cited in this document and correspond to entries in the RAD-AI paper bibliography:

- Amershi, S. et al. (2019). Software engineering for machine learning: A case study. *Proc. ICSE-SEIP*, pp. 291-300.
- Bogner, J. et al. (2021). Characterizing technical debt and antipatterns in AI-based systems. *Proc. TechDebt*, pp. 64-73.
- Chmielinski, K. et al. (2024). The CLeAR documentation framework for AI transparency. Shorenstein Center, Harvard Kennedy School.
- European Parliament and Council (2024). Regulation (EU) 2024/1689 (AI Act). *Official J. EU*.
- Google (2022). Data Cards Playbook. https://sites.research.google/datacardsplaybook/
- Hermann, J. and Del Balso, M. (2017). Meet Michelangelo: Uber's ML platform. Uber Engineering Blog.
- Lucaj, L. et al. (2025). TechOps: Technical documentation templates for the AI Act. *arXiv:2508.08804*.
- Mitchell, M. et al. (2019). Model cards for model reporting. *Proc. FAccT*, pp. 220-229.
- Moin, A. et al. (2023). Enhancing architecture frameworks by including modern stakeholders and their views/viewpoints. *arXiv:2308.05239*.
- Nazir, R. et al. (2024). Architecting ML-enabled systems: Challenges, best practices, and design decisions. *J. Syst. Softw.*, vol. 207, Art. no. 111860.
- Netflix Technology Blog (2024). Supporting diverse ML systems at Netflix.
- Sculley, D. et al. (2015). Hidden technical debt in machine learning systems. *Proc. NeurIPS*, pp. 2503-2511.
- Sidhpurwala, H. et al. (2025). Blueprints of trust: AI system cards for end-to-end transparency and governance. *arXiv:2509.20394*.
- Sovrano, F. et al. (2025). Simplifying software compliance: AI technologies in drafting technical documentation for the AI Act. *Empir. Softw. Eng.*, vol. 30, no. 4, Art. 91.
- Stol, K.-J. and Fitzgerald, B. (2018). The ABC of software engineering research. *ACM Trans. Softw. Eng. Methodol.*, vol. 27, no. 3, Art. no. 11.
- Struber, D. et al. (2025). Architectural tactics for ML-enabled systems: A systematic literature review. *J. Syst. Softw.*, vol. 210, Art. no. 111942.
- Sun, C. et al. (2020). Gallery: A machine learning model management system at Uber. *Proc. EDBT*, pp. 474-477.
- Uber Engineering (2023). D3: An automated system to detect data drifts. Uber Blog.
- Uber Engineering (2025). Raising the bar on ML model deployment safety. Uber Blog.
- Wieringa, R. (2014). *Design Science Methodology for Information Systems and Software Engineering*. Springer.
