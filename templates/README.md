# RAD-AI Templates

Reusable templates for extending your existing arc42 and C4 documentation with AI-specific concerns.

## Usage

1. Copy the relevant template files into your project's documentation directory
2. Fill in each section following the guidance comments
3. Link from your existing arc42 document to the extension files

These templates are designed to **supplement** your existing arc42 and C4 documentation, not replace it. Each template corresponds to an arc42 section or C4 diagram type and adds AI-specific content.

## Contents

### arc42 Extensions

| File | Extends | Purpose |
|------|---------|---------|
| [`03-ai-boundary-delineation.md`](arc42-extensions/03-ai-boundary-delineation.md) | §3 Context & Scope | Mark deterministic vs. non-deterministic system boundaries |
| [`05-model-registry-view.md`](arc42-extensions/05-model-registry-view.md) | §5 Building Block View | Document AI models as first-class building blocks |
| [`06-data-pipeline-view.md`](arc42-extensions/06-data-pipeline-view.md) | §6 Runtime View | Document data flows from collection to inference |
| [`08-responsible-ai-concepts.md`](arc42-extensions/08-responsible-ai-concepts.md) | §8 Cross-Cutting Concepts | Fairness, explainability, human oversight |
| [`09-ai-adr-template.md`](arc42-extensions/09-ai-adr-template.md) | §9 Architecture Decisions | AI-specific decision records |
| [`10-ai-quality-scenarios.md`](arc42-extensions/10-ai-quality-scenarios.md) | §10 Quality Requirements | AI quality attributes and scenarios |
| [`11-ai-debt-register.md`](arc42-extensions/11-ai-debt-register.md) | §11 Risks & Technical Debt | ML-specific technical debt tracking |
| [`12-operational-ai-view.md`](arc42-extensions/12-operational-ai-view.md) | New section | Drift monitoring and continuous learning |

### C4 Extensions

| File | Purpose |
|------|---------|
| [`ai-component-stereotypes.md`](c4-extensions/ai-component-stereotypes.md) | Visual stereotypes for AI components in C4 diagrams |
| [`data-lineage-overlay.md`](c4-extensions/data-lineage-overlay.md) | Data provenance tracing overlay |
| [`non-determinism-boundary.md`](c4-extensions/non-determinism-boundary.md) | Deterministic/non-deterministic boundary diagram |

### Compliance

| File | Purpose |
|------|---------|
| [`compliance-checklist.md`](compliance-checklist.md) | EU AI Act Annex IV documentation requirements checklist |

## Filled-Out Example

See [`examples/smart-urban-mobility/`](../examples/smart-urban-mobility/) for a fully worked example applying these templates to a smart urban mobility ecosystem.
