# Case Study: Smart Urban Mobility Ecosystem

> **Case study type:** Synthetic ecosystem (illustrative)
>
> **Framework applied:** RAD-AI (arc42 extensions E1 through E8, C4 extensions C4-E1 through C4-E3)
>
> **Purpose:** Demonstrate RAD-AI at the ecosystem level, surfacing three ecosystem-level documentation concerns (cascading drift, differentiated compliance, federated governance) that are structurally invisible in standard architecture documentation

---

## 1. Ecosystem Overview

The Smart Urban Mobility (SUM) ecosystem is a synthetic case study designed to illustrate RAD-AI's capabilities at the ecosystem level, directly aligned with the ANGE workshop's focus on architecting next-generation ecosystems. SUM is a multi-stakeholder intelligent transport platform serving a European metropolitan area, integrating public transit, ride-sharing, and micro-mobility services across multiple transit operators into a unified AI-augmented system.

The ecosystem comprises four AI components that interact through shared data infrastructure and cross-component dependencies:

### 1.1 Route Optimization Service

An ML-based route optimization engine that continuously retrains on real-time traffic patterns. The service uses gradient-boosted trees (XGBoost) selected over LSTM for explainability in a public-service context; municipal regulations require that routing decisions be auditable, and XGBoost with SHAP values provides per-route feature attributions suitable for regulatory audits. The model optimizes across transport modes (bus, tram, bike-sharing, walking) considering real-time traffic, weather, demand forecasts from the Demand Prediction Engine, and transit schedules.

### 1.2 Demand Prediction Engine

A time-series forecasting system predicting transport demand across bus, tram, and bike-sharing operators. The engine produces 72-hour demand forecasts per transport mode, city zone, and 30-minute time window. Multiple transit operators consume these forecasts for fleet allocation and capacity planning, but each operator maintains independent service-level agreements and operational constraints.

### 1.3 Anomaly Detection System

A real-time safety monitoring system for the transport network, detecting infrastructure failures, crowd surges, and hazardous conditions from IoT sensor streams, CCTV crowd density estimates, and vehicle telemetry. This component is potentially classified as high-risk under the EU AI Act (Annex III: safety components of critical infrastructure), triggering full Annex IV documentation requirements.

### 1.4 Cross-Operator Data Sharing Platform

A federated feature store enabling anonymized data exchange across transit operators. The platform computes shared features (zone-level demand aggregates, network-wide traffic indices, weather impact scores) from operator-specific data while preserving data ownership and privacy boundaries. All operators contribute to and consume from the shared feature store, creating mutual dependencies.

### Key Dependencies

The four components form a dependency chain:

- Demand predictions feed route optimization (demand vectors are input features for route scoring)
- Anomaly detection can trigger route recomputation (safety alerts force rerouting)
- All AI components consume features from the federated feature store
- Route optimization quality depends on the freshness and accuracy of both demand predictions and anomaly detection

These dependencies create ecosystem-level concerns that only become visible when documentation notation can trace data flows across component and operator boundaries.

---

## 2. Documentation Gaps Under Standard Frameworks

Documenting the SUM ecosystem using standard arc42 (sections 1 through 12) and C4 (system context, container, component, code diagrams) reveals five categories of invisible architectural information.

### 2.1 Federated Feature Store Indistinguishable from a Database

In a standard C4 container diagram, the federated feature store appears as a generic "Database" container, identical to any PostgreSQL or Redis instance. The following properties are invisible:

- Feature inventory: shared features computed from cross-operator data
- Data ownership: which operator contributes which features
- Privacy handling: anonymization and aggregation rules applied to cross-operator data
- Freshness contracts: different feature groups have different freshness SLAs per operator
- Consumer mapping: which AI components consume which features from which operators

### 2.2 EU AI Act Risk Classifications Invisible

The anomaly detection system is classified as high-risk under the EU AI Act (Annex III), requiring full Annex IV documentation. The route optimization service and demand prediction engine are classified as limited-risk, requiring only transparency obligations. Standard arc42 and C4 have no mechanism to express risk classifications, differentiate documentation requirements by component, or indicate which components trigger regulatory obligations.

### 2.3 Cross-System Data Dependencies Undocumented

Demand prediction outputs feed as input features to route optimization. Anomaly detection alerts trigger route recomputation. The federated feature store creates shared dependencies across all operators. These cross-component data dependencies are architecturally significant; a change in demand prediction's output distribution silently affects route optimization quality. Standard C4 shows data flow arrows but cannot express the semantic nature of these dependencies or their implications for cascade failures.

### 2.4 Model Versioning Across Operators Creates Compatibility Risks

Each operator's instance of the demand prediction engine retrains independently on operator-specific data while consuming shared features from the federated feature store. When one operator's model retrains and starts consuming shared features differently (e.g., using a different feature subset or interpreting feature semantics differently), the impact on the shared ecosystem is undocumented.

### 2.5 Dual Lifecycle Amplified at Ecosystem Scale

The dual lifecycle problem (ML training/serving cycles running asynchronously with software release cycles) is amplified at the ecosystem level. Each operator independently retrains its models on different schedules, deploys software updates at different cadences, and maintains different monitoring thresholds. The interaction effects between these independent lifecycles across shared infrastructure are invisible in standard documentation.

---

## 3. Ecosystem-Level Concerns Revealed by RAD-AI

RAD-AI surfaces three ecosystem-level concerns that are structurally invisible in standard architecture documentation. These concerns arise specifically because the ecosystem involves multiple AI components interacting across organizational boundaries.

### 3.1 Cascading Drift

Cascading drift occurs when a data distribution shift in one AI component propagates through data dependencies to degrade other components in the ecosystem. In the SUM ecosystem, the cascade chain is:

1. **Origin:** The anomaly detection system experiences drift in its sensor input features (e.g., due to sensor calibration degradation or seasonal changes in baseline readings)
2. **First-order effect:** Drifted anomaly detection produces false safety alerts at an elevated rate
3. **Second-order effect:** False safety alerts trigger unnecessary rerouting in the route optimization service, which treats safety alerts as hard constraints
4. **Third-order effect:** Route optimization quality degrades because it is making routing decisions based on spurious safety constraints, increasing average travel times and reducing user satisfaction

This cascade chain crosses component boundaries and is invisible in standard documentation because:

- Standard C4 shows data flow arrows but cannot annotate them with distribution expectations or drift propagation paths
- Standard arc42 quality scenarios (Section 10) are per-component; they cannot express cross-component quality scenarios
- Standard arc42 has no mechanism to document how one component's degradation cascades to others

**RAD-AI captures this through:**

- **C4-E2 (Data Lineage Overlay):** Traces the data dependency chain from anomaly detection sensor inputs through anomaly alerts to route optimization rerouting decisions. The overlay visualizes the cascade path across operator and component boundaries.
- **E6 (AI Quality Scenarios):** Documents the cascading drift scenario as a cross-component quality scenario with quantitative thresholds and response measures (see Section 4.4 below).

### 3.2 Differentiated Compliance

Different AI components in the same ecosystem face different regulatory requirements under the EU AI Act's risk-based classification system. In SUM:

- **Anomaly Detection (high-risk):** Classified under Annex III as a safety component of critical infrastructure. Full Annex IV documentation is required, including: general system description, design specifications, data governance documentation, training methodology, risk management, lifecycle change documentation, performance metrics, human oversight assessment, and post-market monitoring.
- **Route Optimization (limited-risk):** Not classified as high-risk. Transparency obligations apply (users must be informed when AI influences routing), but full Annex IV documentation is not legally required.
- **Demand Prediction (limited-risk):** Same as route optimization. Transparency obligations only.

This differentiation has architectural consequences:

- Documentation effort must be targeted: full Annex IV documentation for anomaly detection, lighter documentation for the other components
- Shared infrastructure (the federated feature store) serves both high-risk and limited-risk components, creating questions about the documentation obligations for shared data infrastructure
- Changes to the feature store that affect the high-risk anomaly detection component may trigger Annex IV "substantial modification" documentation requirements, even if the change was motivated by a limited-risk component

**RAD-AI captures this through:**

- **E1 (AI Boundary Delineation):** Annotates each AI component in the context diagram with its EU AI Act risk classification (high-risk, limited-risk, minimal-risk). The boundary diagram makes it immediately visible which components are subject to full Annex IV requirements.
- **E4 (Responsible AI Concepts):** Documents per-component compliance obligations, shared infrastructure responsibilities, and the accountability boundaries for cross-component concerns.

### 3.3 Federated Governance

When multiple operators share data infrastructure and consume shared AI outputs, governance questions arise that have no counterpart in single-organization systems:

- **Feature store ownership:** Who is responsible for the accuracy and freshness of shared features? The contributing operator, the consuming operator, or the platform operator?
- **Cross-operator data agreements:** What data can each operator contribute? What anonymization and aggregation rules apply? How are data quality disputes resolved?
- **Accountability boundaries:** When shared features cause a downstream model to degrade, which organization is accountable? The feature contributor, the platform operator, or the model owner?
- **Privacy handling:** How is cross-operator data anonymized? What aggregation level is sufficient to prevent re-identification? How does GDPR's data minimization principle interact with the ML need for feature richness?

These governance concerns are architecturally significant because they constrain data flows, require specific privacy-preserving transformations in the data pipeline, and impose accountability requirements that affect system design.

**RAD-AI captures this through:**

- **E4 (Responsible AI Concepts):** Documents shared feature store ownership across operators, cross-operator data agreements, accountability boundaries, and privacy handling for anonymized cross-operator data. The Responsible AI concern matrix specifies, for each AI component, the fairness metric, threshold, monitoring frequency, and responsible party.
- **E3 (Data Pipeline View):** Documents the cross-operator data flows with privacy transformation points (where anonymization occurs), data ownership annotations, and quality gates at organizational boundaries.

---

## 4. RAD-AI Documentation Applied

This section walks through key RAD-AI extensions as applied to the SUM ecosystem.

### 4.1 E1: AI Boundary Delineation

The AI Boundary Delineation section (extending arc42 Section 3: Context and Scope) partitions the SUM ecosystem into deterministic and non-deterministic regions. The annotated context diagram shows:

- **Deterministic region:** GTFS schedule engine, payment and ticketing system, passenger information displays, fleet management system, operator dashboard
- **Non-deterministic region:** Route Optimization Service (MDL-001), Demand Prediction Engine (MDL-002), Anomaly Detection System (MDL-003), Federated Feature Store, Drift Monitor

Each AI component is annotated with its confidence specification, output type, update frequency, and fallback behavior:

| Component | Confidence Range | Fallback Strategy |
|-----------|-----------------|-------------------|
| Route Optimizer (MDL-001) | 0.75 to 0.97 | Static shortest-path routing from pre-computed GTFS timetables |
| Demand Predictor (MDL-002) | MAPE 6.1% to 12.4% | 8-week rolling average per zone/mode/timeslot |
| Anomaly Detector (MDL-003) | Precision 0.96, Recall 0.91 | Rule-based alerting on raw sensor thresholds; immediate escalation to human operator |

Each boundary crossing between the deterministic and non-deterministic regions passes through a confidence gate with defined threshold, acceptance behavior, and rejection behavior.

### 4.2 E2: Model Registry View

The Model Registry View (extending arc42 Section 5: Building Block View) documents all four ML models with versioning, metrics, and retraining information:

| Model ID | Name | Architecture | Version | Last Retrained | Key Metric | Retraining Trigger |
|----------|------|-------------|---------|----------------|------------|-------------------|
| MDL-001 | Route Optimizer | XGBoost (GBM) | v2.1 | 2026-02-10 | Accuracy 0.93 | MAE > 5.5 min for 3 consecutive days |
| MDL-002 | Demand Predictor | LSTM | v1.4 | 2026-02-08 | MAPE 8.2% | MAPE > 15% on rolling 24h window |
| MDL-003 | Anomaly Detector | Isolation Forest + rule engine | v3.0 | 2026-01-28 | Precision 0.96 | Quarterly mandatory retrain (high-risk) |
| FS-001 | Federated Feature Store | N/A (infrastructure) | v4.2 | N/A | 142 features | Feature schema change triggers downstream evaluation |

Each model entry links to its Model Card (per Mitchell et al., 2019) as a sub-artifact, integrating model-level documentation into the architectural context.

### 4.3 E5: AI Decision Records (AI-ADR)

The AI-ADR for route optimization documents the model selection decision:

- **Decision:** Use XGBoost over LSTM for route optimization
- **Model alternatives considered:** (1) LSTM: higher raw accuracy on historical test sets but poor explainability; no SHAP equivalent for per-route audit; (2) XGBoost: slightly lower accuracy but provides exact SHAP values for per-route feature attributions suitable for regulatory audits
- **Dataset characteristics:** 3 years of historical ridership, 12 city zones, 3 transport modes, augmented with weather and event calendar data
- **Fairness/bias trade-offs:** XGBoost enables zone-stratified evaluation to detect service quality disparities across neighborhoods
- **Expected model lifetime:** 1 to 3 months before retraining is needed
- **Retraining triggers:** MAE > 5.5 minutes for 3 consecutive days triggers automated retraining; major infrastructure changes (new transit line, road closure) trigger immediate retraining
- **Explainability requirements:** SHAP values for per-route feature attributions; aggregate SHAP summary plots for monthly city authority reports
- **Regulatory rationale:** Explainability requirement driven by municipal audit obligations for public-service AI; XGBoost selected specifically because it supports exact SHAP values

### 4.4 E6: AI Quality Scenarios (Cascading Drift Scenario)

The cascading drift scenario is documented as a cross-component quality scenario:

| Element | Description |
|---------|-------------|
| **Source** | Drift monitoring agent (hourly) |
| **Stimulus** | Feature distribution shift exceeding 2 sigma on 3 or more input features for more than 12 consecutive hours in the anomaly detection system |
| **Artifacts** | MDL-003 (Anomaly Detector), MDL-001 (Route Optimizer) |
| **Environment** | Production; normal operating conditions |
| **Response** | Stage 1: Anomaly detection confidence threshold tightens from 0.85 to 0.95 (reducing false alert rate at the cost of higher false negative rate). Stage 2: Route optimization activates last-known-good model version as fallback, suspending rerouting based on anomaly alerts until anomaly detection drift is resolved. Stage 3: ML engineering team notified for root cause analysis. |
| **Response Measure** | Drift detected within 2 hours of onset; false safety alerts remain below 5% during the cascade period; downstream route optimization MAE stays below 7 minutes; full resolution (retraining and redeployment of anomaly detection model) within 24 hours |

This scenario cannot be expressed in standard arc42 Section 10, which structures quality scenarios per-component. The cascading drift scenario spans two components and documents the propagation path, intermediate interventions, and cross-component acceptance criteria.

### 4.5 C4-E1: AI Component Stereotypes

The C4 component diagram for the SUM ecosystem uses all five RAD-AI stereotypes:

| Component | Stereotype | Visual Distinction |
|-----------|-----------|-------------------|
| Demand Prediction Engine | `<<ML Model>>` | Distinguishes from standard microservices |
| Anomaly Detection System | `<<ML Model>>` + high-risk annotation | Distinguishes from other ML models; indicates regulatory obligations |
| Route Optimization Service | `<<ML Model>>` | Distinguishes from standard route planning software |
| Drift Monitor | `<<Monitor>>` | Distinguishes from standard logging/APM |
| Federated Feature Store | `<<Feature Store>>` | Distinguishes from standard databases |
| Operator Dashboard | `<<Human-in-the-Loop>>` | Identifies human override capability for anomaly review |

The stereotypes provide immediate visual communication of each component's architectural nature. An architect reviewing the diagram can instantly identify which components are ML models (and therefore exhibit probabilistic behavior), which provide monitoring (and therefore observe other components), and which involve human judgment (and therefore have different latency and availability characteristics).

### 4.6 C4-E3: Non-Determinism Boundary

The Non-Determinism Boundary Diagram partitions the SUM ecosystem into four regions:

1. **Deterministic region** (below the boundary line): GTFS Schedule Engine, Payment and Ticketing, Passenger Information Displays, Fleet Management, Operator Dashboard, API Gateway, Business Rules Engine, Audit Logger
2. **Confidence Gate Layer** (at the boundary): Three confidence gates (CG-1 for Route Optimizer, CG-2 for Demand Predictor, CG-3 for Anomaly Detector) with defined thresholds and routing behavior
3. **Non-deterministic region** (above the boundary line): Route Optimizer, Demand Predictor, Anomaly Detector, Federated Feature Store, Drift Monitor
4. **Deterministic Fallback region**: Rule-Based Router (fallback for MDL-001), Historical Average (fallback for MDL-002), Manual Safety Protocol (fallback for MDL-003)

Propagation rules document cascade behavior:

- If the Federated Feature Store becomes unavailable, all three ML models switch to fallback simultaneously (cascade fallback)
- Feature staleness beyond threshold reduces downstream model confidence by a staleness penalty (0.05 per minute past SLA)
- The Business Rules Engine applies hard constraints (fare caps, schedule validity, operator SLAs) after ML output, ensuring deterministic post-processing regardless of ML component behavior

---

## 5. TikZ Diagram Description

The paper includes a C4 Component diagram for the SUM ecosystem rendered in TikZ. The diagram uses the following visual conventions:

- **Color-coded boxes** distinguish AI component types: orange for `<<ML Model>>` components (Demand Prediction, Anomaly Detection, Route Optimization), blue for `<<Monitor>>` (Drift Detection), green for `<<Feature Store>>` (Federated Feature Store), purple for `<<Human-in-the-Loop>>` (Operator Dashboard)
- **A dashed horizontal line** separates the non-deterministic region (above) from the deterministic region (below), making the system's behavioral partition immediately visible
- **Dotted arrows** trace data lineage between components, distinguishing data dependency flows from control flows (solid arrows)
- **High-risk annotation** on the Anomaly Detection component ("EU AI Act: high-risk") visually flags the component subject to full Annex IV documentation requirements

The component layout and data flows are:

- The **Federated Feature Store** (green, above the line) feeds feature vectors to both **Demand Prediction** (orange) and **Anomaly Detection** (orange, high-risk annotated)
- **Demand Prediction** outputs demand forecasts to **Route Optimization** (orange)
- **Anomaly Detection** outputs safety alerts to **Route Optimization**
- The **Drift Monitor** (blue) observes Route Optimization and can trigger fallback or retraining
- The **Operator Dashboard** (purple, below the line) receives alerts from Anomaly Detection and has override authority over Route Optimization decisions
- **Route Optimization** produces routing decisions consumed by deterministic downstream systems (Passenger Information Displays, Fleet Management)

The diagram communicates, in a single view, the system's AI component topology, data dependencies, non-determinism boundaries, regulatory classifications, and human oversight points. None of these properties are expressible in a standard C4 component diagram.

---

## 6. Key Insight

RAD-AI surfaces three ecosystem-level concerns (cascading drift, differentiated compliance, federated governance) that are structurally invisible in standard architecture documentation. These concerns are distinct from the per-component concerns identified in the Uber and Netflix case studies; they arise specifically because the SUM ecosystem involves multiple AI components interacting across organizational boundaries.

**Cascading drift** requires documentation notation that can trace data dependencies across component boundaries and express cross-component quality scenarios. Standard arc42 quality scenarios are per-component; they cannot capture the cascade chain from anomaly detection drift through false safety alerts to route optimization degradation. RAD-AI's Data Lineage Overlay (C4-E2) and AI Quality Scenarios (E6) provide this cross-component visibility.

**Differentiated compliance** requires documentation notation that can distinguish AI components by regulatory risk classification and express different documentation obligations for different components within the same system. Standard arc42 and C4 treat all components uniformly; there is no mechanism to mark a component as "high-risk under EU AI Act Annex III" and link it to specific documentation requirements. RAD-AI's AI Boundary Delineation (E1) and Responsible AI Concepts (E4) make these classifications and their documentation implications explicit.

**Federated governance** requires documentation notation that can express data ownership, cross-organizational accountability, and privacy-preserving data flows. Standard arc42 and C4 assume a single organizational boundary; they have no mechanism for documenting shared infrastructure responsibilities across operators, cross-operator data agreements, or the accountability chain when shared features cause downstream model degradation. RAD-AI's Responsible AI Concepts (E4) and Data Pipeline View (E3) provide structured sections for these governance concerns.

These three concerns become visible only when documentation notation can distinguish AI components from standard software (C4-E1 stereotypes), trace data lineage across operator boundaries (C4-E2 overlay), express non-deterministic behavior contracts (C4-E3 boundary), and document AI-specific quality scenarios that span multiple components (E6). The SUM case study demonstrates that RAD-AI's extensions are not only useful for per-component documentation (as shown in the Uber and Netflix case studies) but also enable ecosystem-level architectural reasoning that is entirely beyond the expressive capacity of standard frameworks.

---

## References

1. Starke, G. and Hruschka, P. "arc42: Effective, lean and pragmatic architecture documentation." 2023. https://arc42.org
2. Brown, S. "The C4 model for visualising software architecture." 2018. https://c4model.com
3. European Parliament and Council. "Regulation (EU) 2024/1689 (EU AI Act)." Official Journal of the European Union, 2024.
4. Mitchell, M. et al. "Model Cards for Model Reporting." In Proceedings of FAT*, 2019.
5. Sculley, D. et al. "Hidden Technical Debt in Machine Learning Systems." In Advances in Neural Information Processing Systems (NeurIPS), 2015.
6. Bass, L., Clements, P., and Kazman, R. "Software Architecture in Practice." 4th edition, Addison-Wesley, 2022.
