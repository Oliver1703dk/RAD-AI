# RAD-AI Framework Documentation

RAD-AI (Rethinking Architecture Documentation for AI-Augmented Ecosystems) provides backward-compatible extensions to two widely adopted software architecture documentation frameworks: arc42 (eight section extensions) and the C4 model (three new diagram types). These extensions enable practitioners to document AI-augmented ecosystems, capturing concerns such as non-deterministic behavior, data-dependent evolution, dual lifecycles, emergent quality attributes, and regulatory requirements. RAD-AI also includes a systematic EU AI Act Annex IV compliance mapping, providing a concrete documentation path toward the August 2, 2026 enforcement deadline for high-risk AI systems.

## Quick Start

- **Understand the framework.** Read [framework-overview.md](framework-overview.md) for design principles, architecture, and the rationale behind each extension.
- **Pick your starting point.** Consult the [adoption-guide.md](adoption-guide.md) for a staged adoption path. Stage 1 (E1 + E2) provides immediate visibility; later stages add operational maturity.
- **Use the templates.** Copy the relevant templates from `../templates/` into your project and fill in the sections that apply to your system.
- **Check compliance.** If your system falls under the EU AI Act, use [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md) to map Annex IV requirements to RAD-AI sections.

## Documentation Map

| File | Purpose |
|------|---------|
| [framework-overview.md](framework-overview.md) | Framework overview, design principles, and architecture |
| [arc42-extensions-reference.md](arc42-extensions-reference.md) | Detailed reference for all 8 arc42 extensions (E1 through E8) |
| [c4-extensions-reference.md](c4-extensions-reference.md) | Detailed reference for all 3 C4 extensions (C4-E1 through C4-E3) |
| [eu-ai-act-compliance-guide.md](eu-ai-act-compliance-guide.md) | EU AI Act Annex IV compliance mapping and practical guide |
| [adoption-guide.md](adoption-guide.md) | Incremental adoption guide with staged approach |
| [research-background.md](research-background.md) | Gap analysis, related work, and research methodology |
| [evaluation-results.md](evaluation-results.md) | Evaluation methodology and results |
| [case-studies/uber-michelangelo.md](case-studies/uber-michelangelo.md) | Uber Michelangelo case study |
| [case-studies/netflix-metaflow.md](case-studies/netflix-metaflow.md) | Netflix Metaflow/Maestro case study |
| [case-studies/smart-urban-mobility.md](case-studies/smart-urban-mobility.md) | Smart urban mobility ecosystem case study |
| [glossary.md](glossary.md) | Glossary of key terms |

## Related Resources

| Resource | Description |
|----------|-------------|
| [`../templates/`](../templates/) | Reusable templates for each arc42 and C4 extension |
| [`../examples/`](../examples/) | Worked examples applying RAD-AI to production AI platforms |
| [`../comparative_analysis/`](../comparative_analysis/) | Side-by-side standard vs. RAD-AI documentation comparisons |
| [`../paper/`](../paper/) | The research paper (IEEE ICSA / ANGE 2026 workshop) |

## Key Contributions

- **C1:** First backward-compatible extension of arc42 (8 section extensions) and C4 (3 diagram types) for documenting AI-augmented systems.
- **C2:** Systematic EU AI Act Annex IV compliance mapping with quantified coverage assessment.
- **C3:** Comparative analytical evidence on two production AI platforms (Uber Michelangelo, Netflix Metaflow/Maestro) demonstrating that documentation gaps are structural properties of current frameworks, not domain-specific oversights.
- **C4:** Identification of ecosystem-level documentation concerns (cascading drift, differentiated compliance, federated governance) that are invisible in standard frameworks.
