# RAD-AI Glossary

Definitions of key terms used throughout the RAD-AI framework documentation, paper, and templates. Terms are listed in alphabetical order. Each definition uses the same formal register as the framework specification.

---

**AI-ADR (AI Architecture Decision Record)**
An extension of the Markdown Any Decision Records (MADR) template with seven AI-specific fields: model alternatives, dataset characteristics, fairness/bias trade-offs, expected model lifetime, retraining triggers, explainability requirements, and regulatory rationale. AI-ADRs are the key artifact of RAD-AI extension E5 and provide structured decision capture for ML-related architectural choices.

**AI Boundary Delineation**
RAD-AI extension E1, which augments arc42 Section 3 (Context and Scope) by requiring explicit marking of deterministic versus non-deterministic system boundaries. Each boundary crossing is annotated with a four-part contract: output type, confidence specification, update frequency, and fallback behavior.

**AI Debt Register**
RAD-AI extension E7, which augments arc42 Section 11 (Risks and Technical Debt) with ML-specific debt categories derived from Sculley et al. (2015) and Bogner et al. (2021). Each register entry records debt category, affected components, severity, estimated remediation effort, owner, and status.

**AI Quality Scenario**
RAD-AI extension E6, which augments arc42 Section 10 (Quality Requirements) with AI-specific quality scenarios following the source-stimulus-response format. Each scenario specifies an AI-specific source (e.g., data drift, model staleness, adversarial input), a stimulus with quantitative trigger, an environment (training, serving, or monitoring), and a measurable response with deadline.

**AI-augmented ecosystem**
An interconnected system where multiple AI components interact through shared data, infrastructure, and feedback loops. RAD-AI uses this term (rather than "AI system") to emphasize the multi-component, multi-stakeholder nature of modern AI deployments, aligned with the ANGE workshop focus on next-generation ecosystems.

**Annex IV (EU AI Act)**
The section of the EU AI Act (Regulation 2024/1689) that specifies nine sections of required technical documentation for high-risk AI systems. Requirements span system description, design specifications, data governance, training methodologies, risk management, lifecycle changes, performance metrics, human oversight, and post-market monitoring. Section 2(c) explicitly mandates architecture documentation. Enforcement for high-risk systems begins August 2, 2026.

**arc42**
A pragmatic twelve-section template for software architecture documentation, widely adopted in European industry. Created by Gernot Starke, arc42 structures documentation around sections covering goals, constraints, context, solution strategy, building blocks, runtime views, deployment, cross-cutting concepts, decisions, quality requirements, risks and technical debt, and a glossary. RAD-AI extends eight of these twelve sections.

**Backward compatibility**
A core design principle of RAD-AI requiring that all extensions augment existing arc42 sections and C4 diagrams without invalidating existing documentation. Teams with current arc42 or C4 documentation can adopt RAD-AI incrementally without rewriting their existing artifacts.

**Boundary erosion**
A category of AI technical debt in which the intended separation between deterministic and non-deterministic system regions degrades over time. This occurs when ML outputs are consumed without confidence checks, when fallback mechanisms are bypassed, or when model-to-model dependencies bypass boundary contracts.

**C4 model**
A hierarchical approach to software architecture visualization at four abstraction levels: System Context, Container, Component, and Code. Created by Simon Brown, the C4 model also defines supplementary diagram types (Dynamic, Deployment, System Landscape). RAD-AI extends the C4 model with three new diagram types: AI Component Stereotypes, Data Lineage Overlay, and Non-Determinism Boundary Diagram.

**CACE (Changing Anything Changes Everything)**
A property of ML systems identified by Sculley et al. (2015) in which no input feature, hyperparameter, or data source can be changed in isolation. Changes in one component propagate through data dependencies and feedback loops, potentially affecting all other components. RAD-AI's Data Lineage Overlay (C4-E2) and AI Debt Register (E7) help make these entanglements visible and trackable.

**Canary deployment**
A deployment strategy in which a new model version receives a small fraction of production traffic while the existing version serves the remainder. Predictions from both versions are compared against ground truth to assess the new version's quality before full promotion. Canary deployment is documented in RAD-AI extension E8 (Operational AI View).

**Cascading drift**
An ecosystem-level concern where data drift in one AI component triggers degraded behavior in downstream components that consume its outputs. For example, drift in an anomaly detection model may produce false safety alerts, triggering unnecessary rerouting in a route optimization model. Cascading drift is one of three ecosystem concerns (alongside differentiated compliance and federated governance) identified through RAD-AI's extended notation.

**Concept drift**
A change in the statistical relationship between input features and the target variable over time. Unlike data drift (which concerns input distributions), concept drift means the mapping from inputs to correct outputs has changed. Concept drift renders trained models less accurate even if input distributions remain stable.

**Confidence specification**
A component of the boundary contract in RAD-AI extensions E1 and C4-E3. The confidence specification defines the metric and threshold used to determine whether a non-deterministic component's output is trusted. Examples include precision thresholds, MAE bounds, and AUC floors.

**Data Card**
A structured documentation artifact (Google, 2022) describing a dataset's purpose, composition, collection process, recommended uses, and limitations. In RAD-AI, Data Cards attach as sub-artifacts at data source nodes in the Data Pipeline View (E3) and Data Lineage Overlay (C4-E2), integrating data governance documentation into the architecture.

**Data dependency debt**
A form of technical debt identified by Sculley et al. (2015) in which ML systems develop unstable, implicit dependencies on external data sources. Changes to upstream data (schema, distribution, freshness) can silently degrade model performance. RAD-AI's Data Lineage Overlay (C4-E2) and AI Debt Register (E7) address this by making data dependencies explicit.

**Data drift**
A change in the statistical distribution of input features over time, distinct from concept drift. Data drift can cause models to encounter inputs outside their training distribution, leading to degraded prediction quality. Drift detection is documented through RAD-AI extension E8 (Operational AI View) and the Monitor stereotype (C4-E1).

**Data lineage**
The record of data provenance from source through transformation to model consumption. Data lineage traces where data originates, how it is transformed at each pipeline stage, what quality constraints apply, and which models consume the resulting features. C4-E2 (Data Lineage Overlay) is the primary RAD-AI artifact for documenting data lineage.

**Data Pipeline View**
RAD-AI extension E3, which augments arc42 Section 6 (Runtime View) with the complete ML data flow: collection, preprocessing, feature engineering, training, inference, and feedback loops. Each pipeline stage is annotated with quality gates specifying check type, threshold, and action on failure.

**Degradation profile**
A component of the boundary contract in RAD-AI extensions E1 and C4-E3. The degradation profile documents how system quality decreases when a non-deterministic component is unavailable or producing low-confidence outputs, including user-facing impact, business impact, maximum acceptable duration, and recovery procedure.

**Design Science Research**
A research methodology for information systems and software engineering (Wieringa, 2014) that structures research into Problem Investigation, Treatment Design, and Treatment Validation. RAD-AI was developed following this methodology, with gap analysis informing design and multiple evaluation methods providing analytical validation.

**Deterministic region**
The portion of an architecture containing components whose outputs are fully determined by their inputs and configuration. In RAD-AI's Non-Determinism Boundary Diagram (C4-E3), deterministic regions include components such as API gateways, business rules engines, payment processors, and scheduling services.

**Differentiated compliance**
An ecosystem-level concern where different AI components within the same ecosystem face different regulatory requirements. For example, an anomaly detection system may be classified as high-risk under the EU AI Act (requiring full Annex IV documentation), while a route optimization service may fall under limited-risk obligations. RAD-AI extension E1 (AI Boundary Delineation) makes these risk classifications architecturally visible.

**Drift detection**
The process of monitoring input data distributions and model outputs for statistically significant changes over time. Drift detection components are represented by the Monitor stereotype (`[MON]`) in RAD-AI's C4 extensions and documented in the Operational AI View (E8).

**Dual lifecycle**
The characteristic of AI-augmented systems in which ML models follow a lifecycle (training, evaluation, deployment, monitoring, retraining) that runs asynchronously with the software release cycle. Standard architecture documentation assumes a single lifecycle; RAD-AI's Model Registry View (E2) and Operational AI View (E8) capture both lifecycles.

**Entanglement**
A property of ML systems in which components are implicitly coupled through shared features, data sources, or feedback loops. Entanglement makes it difficult to change one model without affecting others. It is tracked as a debt category in the AI Debt Register (E7).

**EU AI Act (Regulation 2024/1689)**
European Union legislation establishing harmonized rules on artificial intelligence, in force since August 2024. The Act classifies AI systems by risk level and imposes graduated documentation and compliance requirements. For high-risk systems, Annex IV mandates comprehensive technical documentation with enforcement beginning August 2, 2026.

**Fairness**
An emergent quality attribute of AI systems concerning equitable treatment across demographic groups or subpopulations. Fairness metrics (e.g., demographic parity ratio, equalized odds) are documented in RAD-AI extension E4 (Responsible AI Concepts) as part of the responsible AI concern matrix.

**Fallback behavior**
The system response when a non-deterministic component's output does not meet confidence thresholds. RAD-AI defines three common fallback types: rule-based default (a deterministic algorithm produces a lower-quality output), cached last-known-good (the most recent acceptable output is reused), and human escalation (the decision is routed to a human-in-the-loop component).

**Feature Store**
A shared feature computation and serving layer that provides semantically curated features to ML models. Feature Stores enforce training-serving consistency through feature DSLs, manage feature versioning, and propagate feature changes to all consuming models. Represented by the `[FS]` stereotype in RAD-AI's C4 extensions.

**Federated governance**
An ecosystem-level concern where shared infrastructure (e.g., a federated feature store) spans multiple organizational boundaries, requiring cross-organization agreements on data ownership, quality standards, and accountability. RAD-AI extension E4 (Responsible AI Concepts) documents shared resource ownership and cross-operator accountability boundaries.

**Feedback loop (hidden)**
A data flow in which an ML model's outputs influence the data it will later be trained on, creating a self-reinforcing cycle that can amplify biases or drive distributional shift. Hidden feedback loops are difficult to detect because they operate across time scales longer than typical monitoring windows. They are tracked as a debt category in the AI Debt Register (E7) and made visible through the Data Lineage Overlay (C4-E2).

**Human-in-the-Loop (HITL)**
A component or process in which a human reviews, overrides, or approves an AI system's output before it takes effect. HITL intervention points are represented by the `[HITL]` stereotype in RAD-AI's C4 extensions. They are architecturally significant because they introduce latency paths, create feedback loops, and represent deterministic overrides of non-deterministic behavior.

**ISO/IEC/IEEE 42010:2022**
The international standard for architecture descriptions of systems and software. It defines concepts including architecture viewpoints, views, concerns, and stakeholders. RAD-AI's extensions align with ISO 42010 viewpoint concepts while adding AI-specific concerns absent from the current standard.

**MADR (Markdown Any Decision Records)**
A lightweight template for recording architecture decisions using Markdown. RAD-AI extension E5 (AI Decision Records) extends the MADR template with seven AI-specific fields, creating the AI-ADR format.

**Model Card**
A structured documentation artifact (Mitchell et al., 2019) reporting a model's intended use, performance metrics, limitations, and ethical considerations. In RAD-AI, Model Cards attach as sub-artifacts of model building blocks in the Model Registry View (E2), integrating ML documentation into the architecture.

**Model freshness**
The degree to which a deployed model reflects current data distributions. Model freshness degrades as the time since last retraining increases and as data distributions shift. It is documented through retraining policies in the Operational AI View (E8) and monitored through the Monitor stereotype (C4-E1).

**Model Registry View**
RAD-AI extension E2, which augments arc42 Section 5 (Building Block View) by elevating AI models to first-class building blocks. Each model entry documents model ID and version, ML framework, training dataset hash, hyperparameter snapshot, evaluation metric with acceptance threshold, deployment status, owner, and last-retrained date.

**Model staleness**
The condition in which a deployed model no longer reflects current data distributions, resulting in degraded prediction quality. Model staleness is a common stimulus in AI Quality Scenarios (E6) and a trigger for retraining in the Operational AI View (E8).

**Monitor (component stereotype)**
The `[MON]` stereotype in RAD-AI's C4 extensions, representing a drift detection and model health monitoring component. Required properties include metrics tracked and alert thresholds. Monitors are architecturally significant because they trigger retraining, activate fallbacks, and detect silent model degradation.

**Non-determinism boundary**
The interface between deterministic and non-deterministic regions of an architecture. At each non-determinism boundary, a confidence gate evaluates the AI component's output and routes it to either the deterministic consumer (if confidence is sufficient) or a fallback mechanism. Documented through C4-E3 (Non-Determinism Boundary Diagram) and E1 (AI Boundary Delineation).

**Non-deterministic region**
The portion of an architecture containing components whose outputs are probabilistic, vary across model versions or retraining cycles, or depend on data distributions that may shift. In the Non-Determinism Boundary Diagram (C4-E3), non-deterministic regions include ML models, feature stores with real-time data, and drift detectors.

**Operational AI View**
RAD-AI extension E8, which adds a new section to arc42 (no equivalent in the standard template). It is structured around four subsections: monitoring (metrics, dashboards, alerting), retraining policy (triggers, automation level, approval workflow), deployment strategy (canary, blue-green, shadow), and rollback policy (triggers, version retention, downstream implications).

**Pipeline debt**
A form of technical debt in which ML data pipelines accumulate complexity, redundancy, or fragility over time. Pipeline debt includes duplicated transformation logic, hardcoded assumptions about data schemas, and missing quality gates. It is tracked as a debt category in the AI Debt Register (E7).

**prEN 18286**
A draft harmonized standard developed by CEN-CENELEC JTC 21 under the title "Artificial intelligence: Quality management system for EU AI Act regulatory purposes." The standard is currently in public enquiry and may introduce additional documentation requirements beyond those specified in Annex IV. RAD-AI's compliance mapping references prEN 18286 as an evolving regulatory context.

**Quality gate**
A checkpoint in a data pipeline or model lifecycle that evaluates whether data or a model meets specified quality criteria before proceeding to the next stage. Each quality gate specifies a check type (e.g., schema conformance, distribution test), a threshold (e.g., KS-statistic < 0.1), and an action on failure (e.g., halt pipeline, alert, activate fallback). Quality gates are documented in RAD-AI extension E3 (Data Pipeline View).

**RAD-AI**
Rethinking Architecture Documentation for AI-Augmented Ecosystems. A backward-compatible extension framework that augments arc42 with eight AI-specific sections (E1 through E8) and the C4 model with three diagram extensions (C4-E1 through C4-E3), complemented by a systematic EU AI Act Annex IV compliance mapping.

**Responsible AI Concepts**
RAD-AI extension E4, which augments arc42 Section 8 (Cross-Cutting Concepts) with a structured concern matrix. Rows represent AI components; columns represent five concern categories: fairness, explainability, human oversight, privacy, and safety. Each cell documents the applicable metric or method, acceptance threshold, monitoring frequency, and responsible party.

**Retraining trigger**
A condition that initiates retraining of a deployed ML model. Triggers may be scheduled (e.g., weekly), performance-based (e.g., accuracy drops below threshold for N consecutive days), or drift-based (e.g., input distribution shift exceeds statistical threshold). Retraining triggers are documented in AI-ADRs (E5) and the Operational AI View (E8).

**Rollback policy**
The specification of conditions under which a deployed model version is reverted to a previous version, including rollback triggers, model version retention depth, and downstream data implications. Rollback policies are documented in the Operational AI View (E8).

**Shadow deployment**
A deployment strategy in which a new model version receives production traffic but its predictions are logged rather than served to users. The shadow model's outputs are compared against the production model's outputs and ground truth to assess quality before promotion. Shadow deployment is documented in RAD-AI extension E8.

**Structurizr DSL**
A domain-specific language for describing software architecture using the C4 model, developed by Simon Brown. RAD-AI's C4 extensions are compatible with Structurizr DSL through its `tags` mechanism and custom element styles.
