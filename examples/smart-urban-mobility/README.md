# Smart Urban Mobility Ecosystem — RAD-AI Example

A fully worked example applying the RAD-AI framework to a smart urban mobility ecosystem.

## System Overview

The Smart Urban Mobility (SUM) platform is a multi-stakeholder system serving a European metropolitan area, integrating public transit, ride-sharing, and micro-mobility services into a unified intelligent transport platform.

### AI Components

| Component | Model ID | Task | Criticality |
|-----------|----------|------|-------------|
| **Route Optimizer** | MDL-001 | Optimal route selection across transport modes using real-time traffic, weather, and demand data | Medium — affects user experience, not safety-critical |
| **Demand Predictor** | MDL-002 | Time-series forecasting of transport demand per mode, zone, and time window | Medium — drives fleet allocation and capacity planning |
| **Anomaly Detector** | MDL-003 | Real-time detection of safety-relevant anomalies (infrastructure failures, crowd surges, hazardous conditions) | High — safety-critical, requires explainability |

### Stakeholders

| Stakeholder | Role | Key Concerns |
|-------------|------|-------------|
| City transport authority | System owner, regulator | Safety, equity of service, EU AI Act compliance |
| Transit operators (3) | Service providers | Demand forecasts, fleet optimization |
| Commuters | End users | Route quality, transparency |
| Data protection officer | Privacy oversight | GDPR compliance, data minimization |
| ML engineering team | Model development & ops | Model performance, retraining, monitoring |

### EU AI Act Classification

The anomaly detection component (MDL-003) is classified as **high-risk** under EU AI Act Annex III (safety components of transport infrastructure). The route optimizer and demand predictor are classified as **limited-risk**.

## Contents

- [`arc42-extensions/`](arc42-extensions/) — Filled-out arc42 extension sections
- [`c4-diagrams/`](c4-diagrams/) — C4 diagrams with AI stereotypes (Mermaid)
- [`compliance-checklist.md`](compliance-checklist.md) — EU AI Act Annex IV compliance tracking
