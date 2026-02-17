# EU AI Act Annex IV — Compliance Checklist

> Maps EU AI Act Annex IV technical documentation requirements to RAD-AI framework sections.
>
> **Usage:** Copy this checklist into your project. For each requirement, check the box when the corresponding RAD-AI section is filled in and covers the requirement.

## Instructions

1. Fill in the system name and risk classification
2. Work through each requirement, filling in the corresponding RAD-AI section
3. Mark each item as complete when documented
4. Review with your compliance team

---

**System Name:** _____________________
**Risk Classification:** [ ] High-risk  [ ] Limited-risk  [ ] Minimal-risk
**Assessment Date:** YYYY-MM-DD
**Assessor:** _____________________

---

## 1. General Description of the AI System

- [ ] **1.1** Intended purpose of the AI system
  - **RAD-AI Section:** arc42 §1 (Introduction & Goals)
  - **Evidence:** _____________________

- [ ] **1.2** Name and version of the AI system
  - **RAD-AI Section:** arc42 §1 (Introduction & Goals)
  - **Evidence:** _____________________

- [ ] **1.3** Description of interaction with hardware or software not part of the AI system
  - **RAD-AI Section:** arc42 §3 (Context & Scope) + AI Boundary Delineation
  - **Evidence:** _____________________

- [ ] **1.4** Versions of relevant software or firmware
  - **RAD-AI Section:** Model Registry View (version history)
  - **Evidence:** _____________________

## 2. Detailed Description of System Elements

- [ ] **2.1** Development process and methods
  - **RAD-AI Section:** arc42 §5 (Building Block View) + Model Registry View
  - **Evidence:** _____________________

- [ ] **2.2** Design specifications and system architecture
  - **RAD-AI Section:** arc42 §5–§7 + C4 diagrams + AI Component Stereotypes
  - **Evidence:** _____________________

- [ ] **2.3** Description of computational resources used
  - **RAD-AI Section:** arc42 §7 (Deployment View)
  - **Evidence:** _____________________

## 3. Data and Data Governance

- [ ] **3.1** Description of training data — characteristics, requirements, and data governance measures
  - **RAD-AI Section:** Data Pipeline View + Data Lineage Overlay
  - **Evidence:** _____________________

- [ ] **3.2** Description of data collection and provenance
  - **RAD-AI Section:** Data Lineage Overlay (provenance tracking)
  - **Evidence:** _____________________

- [ ] **3.3** Information about data preparation (annotation, labeling, cleaning)
  - **RAD-AI Section:** Data Pipeline View (preprocessing stages)
  - **Evidence:** _____________________

- [ ] **3.4** Formulation of assumptions about training data
  - **RAD-AI Section:** AI-ADR (dataset characteristics section)
  - **Evidence:** _____________________

## 4. Training Methodology and Techniques

- [ ] **4.1** Description of training methodologies and techniques used
  - **RAD-AI Section:** AI-ADR (model selection and training decisions)
  - **Evidence:** _____________________

- [ ] **4.2** Description of training choices, optimization techniques
  - **RAD-AI Section:** AI-ADR (considered alternatives, hyperparameters)
  - **Evidence:** _____________________

## 5. Risk Assessment and Management

- [ ] **5.1** Description of known and foreseeable risks
  - **RAD-AI Section:** arc42 §11 (Risks) + AI Debt Register
  - **Evidence:** _____________________

- [ ] **5.2** Description of risk management measures
  - **RAD-AI Section:** AI Debt Register (mitigation plans) + Non-Determinism Boundary
  - **Evidence:** _____________________

## 6. Lifecycle Changes

- [ ] **6.1** Description of changes made to the system through its lifecycle
  - **RAD-AI Section:** Model Registry View (version history, deployment history)
  - **Evidence:** _____________________

## 7. Performance and Accuracy

- [ ] **7.1** Description of performance metrics
  - **RAD-AI Section:** AI Quality Scenarios + Model Registry View (performance baselines)
  - **Evidence:** _____________________

- [ ] **7.2** Description of accuracy levels and expected foreseeable unintended outcomes
  - **RAD-AI Section:** AI Quality Scenarios + Non-Determinism Boundary (degradation behavior)
  - **Evidence:** _____________________

## 8. Human Oversight

- [ ] **8.1** Description of human oversight measures
  - **RAD-AI Section:** Responsible AI Concepts (human oversight) + AI Component Stereotypes (`<<HITL>>`)
  - **Evidence:** _____________________

- [ ] **8.2** Description of how the system supports human oversight
  - **RAD-AI Section:** Responsible AI Concepts (override and escalation)
  - **Evidence:** _____________________

## 9. Post-Market Monitoring

- [ ] **9.1** Description of post-market monitoring system
  - **RAD-AI Section:** Operational AI View (monitoring architecture, drift detection)
  - **Evidence:** _____________________

- [ ] **9.2** Description of logging and audit capabilities
  - **RAD-AI Section:** Operational AI View (incident response) + Responsible AI Concepts (transparency)
  - **Evidence:** _____________________

---

## Summary

| Section | Items | Completed | Coverage |
|---------|-------|-----------|----------|
| 1. General Description | 4 | /4 | % |
| 2. System Elements | 3 | /3 | % |
| 3. Data Governance | 4 | /4 | % |
| 4. Training | 2 | /2 | % |
| 5. Risk Assessment | 2 | /2 | % |
| 6. Lifecycle Changes | 1 | /1 | % |
| 7. Performance | 2 | /2 | % |
| 8. Human Oversight | 2 | /2 | % |
| 9. Post-Market Monitoring | 2 | /2 | % |
| **Total** | **22** | **/22** | **%** |
