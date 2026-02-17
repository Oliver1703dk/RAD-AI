# RAD-AI: Rethinking Architecture Documentation for AI-Augmented Ecosystems

Backward-compatible extensions to [arc42](https://arc42.org/) and the [C4 model](https://c4model.com/) for documenting AI-augmented systems, with EU AI Act Annex IV compliance mapping.

> **Companion repository** for the paper: *RAD-AI: Rethinking Architecture Documentation for AI-Augmented Ecosystems*, presented at the 1st International Workshop on Architecting Next Generation of Ecosystems (ANGE 2026), co-located with IEEE ICSA 2026, Amsterdam.

## The Problem

arc42 and C4 were designed for deterministic software. AI-augmented systems introduce non-deterministic behavior, data-dependent evolution, dual lifecycles, and regulatory requirements (EU AI Act) that these frameworks do not address. RAD-AI fills this gap with minimal, backward-compatible extensions.

## What's in This Repo

### [`templates/`](templates/)

Blank, reusable templates you can copy into your own project:

- **8 arc42 section extensions** — AI Boundary Delineation, Model Registry View, Data Pipeline View, Responsible AI Concepts, AI-ADR, AI Quality Scenarios, AI Debt Register, Operational AI View
- **3 C4 diagram extensions** — AI Component Stereotypes, Data Lineage Overlay, Non-Determinism Boundary Diagram
- **EU AI Act Annex IV compliance checklist**

### [`examples/smart-urban-mobility/`](examples/smart-urban-mobility/)

A fully worked example applying RAD-AI to a smart urban mobility ecosystem with route optimization, demand prediction, and anomaly detection components. Shows exactly what each template looks like when filled out.

## Key Contributions

| ID | Contribution |
|----|-------------|
| **C1** | First backward-compatible extension of arc42 + C4 for AI systems |
| **C2** | AI-specific Architecture Decision Records (AI-ADR) |
| **C3** | Systematic EU AI Act Annex IV compliance mapping |
| **C4** | Three-part evaluation: compliance assessment, comparative analysis, case study |

## Quick Start

1. Copy the `templates/` directory into your project
2. Fill in the arc42 extensions alongside your existing arc42 documentation
3. Add AI stereotypes to your C4 diagrams using the provided Mermaid snippets
4. Use the compliance checklist to track EU AI Act Annex IV coverage

## How RAD-AI Extends arc42

| arc42 Section | RAD-AI Extension |
|---------------|-----------------|
| §3 Context & Scope | AI Boundary Delineation |
| §5 Building Block View | Model Registry View |
| §6 Runtime View | Data Pipeline View |
| §8 Cross-Cutting Concepts | Responsible AI Concepts |
| §9 Architecture Decisions | AI Decision Records (AI-ADR) |
| §10 Quality Requirements | AI Quality Scenarios |
| §11 Risks & Technical Debt | AI Debt Register |
| New section | Operational AI View |

## How RAD-AI Extends C4

| Extension | Purpose |
|-----------|---------|
| AI Component Stereotypes | `<<ML Model>>`, `<<Data Pipeline>>`, `<<Feature Store>>`, `<<Monitor>>`, `<<Human-in-the-Loop>>` |
| Data Lineage Overlay | Traces data provenance from source through transformation to model consumption |
| Non-Determinism Boundary | Marks deterministic vs. non-deterministic regions with confidence levels and fallbacks |

## Citation

If you use RAD-AI in your work, please cite:

```bibtex
@inproceedings{larsen2026radai,
  title     = {{RAD-AI}: Rethinking Architecture Documentation for {AI}-Augmented Ecosystems},
  author    = {Larsen, Oliver},
  booktitle = {Proceedings of the 1st International Workshop on Architecting Next Generation of Ecosystems (ANGE 2026)},
  year      = {2026},
  note      = {Co-located with IEEE ICSA 2026, Amsterdam}
}
```

## License

[MIT](LICENSE)
