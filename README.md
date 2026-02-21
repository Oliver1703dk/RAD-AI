# RAD-AI: Rethinking Architecture Documentation for AI-Augmented Ecosystems

Backward-compatible extensions to [arc42](https://arc42.org/) and the [C4 model](https://c4model.com/) for documenting AI-augmented systems, with EU AI Act Annex IV compliance mapping.

> **Companion repository** for the paper: *RAD-AI: Rethinking Architecture Documentation for AI-Augmented Ecosystems*, presented at the 1st International Workshop on Architecting Next Generation of Ecosystems ([ANGE 2026](https://2026.icsa-conferences.org/)), co-located with IEEE ICSA 2026, Amsterdam.

## The Problem

arc42 and C4 were designed for deterministic software. AI-augmented ecosystems introduce non-deterministic behavior, data-dependent evolution, dual lifecycles, and regulatory requirements (EU AI Act) that these frameworks do not address. RAD-AI fills this gap with minimal, backward-compatible extensions.

## Key Results

Compliance coverage scored by six experienced software-architecture practitioners (Fleiss' kappa = 0.68, substantial agreement). Totals are mean values across raters.

| Metric | Standard Frameworks | RAD-AI |
|--------|-------------------|--------|
| EU AI Act Annex IV coverage (arc42) | 36% (7.3/20) | **93% (18.5/20)** |
| EU AI Act Annex IV coverage (C4) | 26% (5.2/20) | **73% (14.6/20)** |
| AI concerns captured (Uber Michelangelo) | 0 full, 2 partial | **8 full, 2 partial** |
| AI concerns captured (Netflix Metaflow) | 0 full, 2 partial | **8 full, 2 partial** |
| Ecosystem-level concerns surfaced | 0 | **3** |

## Repository Structure

```
RAD-AI/
├── paper/                              # Research paper
│   └── full_paper/                     #   LaTeX source (main.tex) and PDF
├── templates/                          # Reusable blank templates
│   ├── arc42-extensions/               #   8 arc42 section extensions
│   ├── c4-extensions/                  #   3 C4 diagram extensions
│   └── compliance-checklist.md         #   EU AI Act Annex IV checklist
├── examples/
│   └── smart-urban-mobility/           # Illustrative ecosystem case study
│       ├── arc42-extensions/           #   Filled-in arc42 extensions
│       ├── c4-diagrams/               #   C4 diagrams with AI stereotypes
│       └── compliance-checklist.md    #   Filled-in compliance tracking
├── comparative_analysis/               # Evaluation: production AI platforms
│   ├── uber-michelangelo/              #   Comparative analysis: Uber
│   │   ├── standard/                  #     Standard arc42/C4 (showing gaps)
│   │   ├── rad-ai/                    #     RAD-AI extended documentation
│   │   └── gap-analysis.md            #     Side-by-side coverage comparison
│   └── netflix-metaflow/               #   Comparative analysis: Netflix
│       ├── standard/                  #     Standard arc42/C4 (showing gaps)
│       ├── rad-ai/                    #     RAD-AI extended documentation
│       └── gap-analysis.md            #     Side-by-side coverage comparison
└── documentation/                      # Reference documentation
    ├── framework-overview.md           #   RAD-AI framework overview
    ├── arc42-extensions-reference.md   #   Detailed arc42 extension reference
    ├── c4-extensions-reference.md      #   Detailed C4 extension reference
    ├── eu-ai-act-compliance-guide.md   #   Compliance mapping guide
    ├── adoption-guide.md               #   Incremental adoption guide
    ├── evaluation-results.md           #   Evaluation details
    ├── research-background.md          #   Research context
    ├── glossary.md                     #   Terminology
    └── case-studies/                   #   Extended case study write-ups
```

## What's in This Repo

### [`templates/`](templates/)

Blank, reusable templates you can copy into your own project. Each template supplements (not replaces) your existing documentation:

- **8 arc42 section extensions** -- AI Boundary Delineation, Model Registry View, Data Pipeline View, Responsible AI Concepts, AI-ADR, AI Quality Scenarios, AI Debt Register, Operational AI View
- **3 C4 diagram extensions** -- AI Component Stereotypes, Data Lineage Overlay, Non-Determinism Boundary Diagram
- **EU AI Act Annex IV compliance checklist**

See the [templates README](templates/README.md) for a full listing and usage instructions.

### [`examples/smart-urban-mobility/`](examples/smart-urban-mobility/)

A fully worked example applying RAD-AI to a smart urban mobility ecosystem with four AI components:

| Component | Task | EU AI Act Risk |
|-----------|------|----------------|
| Route Optimizer | Optimal route selection across transport modes | Limited |
| Demand Predictor | Time-series demand forecasting per zone/mode | Limited |
| Anomaly Detector | Real-time safety anomaly detection | **High** (Annex III) |
| Cross-Operator Data Sharing Platform | Federated feature store for anonymized data exchange | Limited |

Shows exactly what each template looks like when filled out, including C4 diagrams with Mermaid syntax and a completed compliance checklist. The high-risk anomaly detector demonstrates documentation requirements under the EU AI Act.

### [`comparative_analysis/uber-michelangelo/`](comparative_analysis/uber-michelangelo/)

Comparative analysis applying both standard arc42/C4 and RAD-AI to Uber's Michelangelo ML platform (5,000+ production models, 10M predictions/sec at peak). Includes:

- **Standard documentation** (arc42 S3, S5, S6; C4 Levels 1-2) showing what current frameworks can capture
- **RAD-AI documentation** (all 8 arc42 extensions + 3 C4 extensions) showing what the extensions reveal
- **Gap analysis** mapping 10 AI-specific concerns (Feature Store topology, model lifecycle, drift monitoring, non-deterministic boundaries, feedback loops, deployment safety, data lineage, responsible AI, AI quality, AI decisions) to standard vs. RAD-AI coverage

### [`comparative_analysis/netflix-metaflow/`](comparative_analysis/netflix-metaflow/)

Same comparative structure applied to Netflix's ML infrastructure (3,000+ ML projects, Metaflow/Maestro orchestration). Highlights different gaps than Uber: invisible DAG dependencies, signal-based cross-workflow coordination, streaming vs. batch distinction, and undocumented A/B testing infrastructure.

### [`documentation/`](documentation/)

Reference documentation covering the full RAD-AI framework, detailed extension references, EU AI Act compliance guide, and an incremental adoption guide.

## Quick Start

1. Copy the [`templates/`](templates/) directory into your project
2. Fill in the arc42 extensions alongside your existing arc42 documentation
3. Add AI stereotypes to your C4 diagrams using the provided Mermaid snippets
4. Use the compliance checklist to track EU AI Act Annex IV coverage
5. Refer to the [smart urban mobility example](examples/smart-urban-mobility/) for a filled-out reference

## Contributions

| ID | Contribution |
|----|-------------|
| **RC1** | First backward-compatible extension of arc42 (8 section extensions) and C4 (3 diagram types) for AI-augmented ecosystems |
| **RC2** | Systematic EU AI Act Annex IV compliance mapping with quantified coverage assessment (practitioner-based, n=6) |
| **RC3** | Comparative analytical evidence on two production AI platforms (Uber Michelangelo, Netflix Metaflow) demonstrating structural documentation gaps |
| **RC4** | Identification of ecosystem-level documentation concerns (cascading drift, differentiated compliance, federated governance) |

## How RAD-AI Extends arc42

| arc42 Section | RAD-AI Extension | Key Artifact |
|---------------|-----------------|--------------|
| S3 Context & Scope | AI Boundary Delineation | Annotated context diagram with AI boundaries |
| S5 Building Block View | Model Registry View | Model registry table with lifecycle metadata |
| S6 Runtime View | Data Pipeline View | Data pipeline diagram with quality gates |
| S8 Cross-Cutting Concepts | Responsible AI Concepts | Fairness, explainability, human oversight matrix |
| S9 Architecture Decisions | AI Decision Records (AI-ADR) | ADR template with 7 AI-specific fields |
| S10 Quality Requirements | AI Quality Scenarios | Drift tolerance, freshness SLAs, fairness constraints |
| S11 Risks & Technical Debt | AI Debt Register | ML-specific debt with remediation plans |
| New section | Operational AI View | Monitoring, retraining, canary deployment |

## How RAD-AI Extends C4

| Extension | Purpose |
|-----------|---------|
| AI Component Stereotypes | `<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, `<<Human-in-the-Loop>>` |
| Data Lineage Overlay | Traces data provenance from source through transformation to model consumption |
| Non-Determinism Boundary | Marks deterministic vs. non-deterministic regions with confidence levels and fallbacks |

## EU AI Act Compliance Mapping

RAD-AI maps all 10 Annex IV requirement categories to specific documentation sections. In a practitioner evaluation (n=6), standard arc42 covers approximately 36% of requirements; with RAD-AI extensions this rises to approximately 93%. The [compliance checklist template](templates/compliance-checklist.md) provides a concrete tracking tool for the August 2, 2026 enforcement deadline.

## Citation

If you use RAD-AI in your work, please cite:

```bibtex
@inproceedings{larsen2026radai,
  title     = {{RAD-AI}: Rethinking Architecture Documentation for {AI}-Augmented Ecosystems},
  author    = {Larsen, Oliver Aleksander and Moghaddam, Mahyar T.},
  booktitle = {Proceedings of the 1st International Workshop on Architecting
               Next Generation of Ecosystems (ANGE 2026)},
  year      = {2026},
  note      = {Co-located with IEEE ICSA 2026, Amsterdam}
}
```

## License

[MIT](LICENSE)
