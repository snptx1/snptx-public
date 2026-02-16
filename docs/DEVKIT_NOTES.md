# SNPTX Extension Development Model

## Overview

SNPTX separates core pipeline execution from downstream analysis through a structured extension system. Extensions consume artifacts produced by the core pipeline, generate new outputs, and never modify upstream state. This boundary guarantees that interpretation does not alter the data it interprets.

Extensions are the mechanism by which SNPTX transforms raw experimental outputs into actionable intelligence: calibration diagnostics, evaluation summaries, metric aggregation, and cross-run synthesis.

---

## Extension Architecture

### Core vs. Extension Boundary

| Component | Core Pipeline | Extension |
|---|---|---|
| Purpose | Train, evaluate, produce artifacts | Analyze, synthesize, report |
| Artifact access | Read/write | Read-only |
| Execution trigger | Snakemake DAG | Post-pipeline invocation |
| Configuration | `configs/config.yaml` | Extension-specific config |
| Output location | `results/`, `metrics/`, `models/` | `results/extensions/{extension_name}/` |

Extensions exist downstream of the core pipeline. They cannot modify training artifacts, evaluation metrics, or model files. They can only consume these artifacts and produce new outputs.

### Contract Structure

Each extension declares a contract specifying:

1. **Required inputs.** Which artifacts the extension expects (metrics files, model artifacts, evaluation outputs).
2. **Output specification.** What the extension produces and where it writes results.
3. **Validation rules.** Conditions that must hold for the extension to execute (e.g., required files exist, schema constraints are met).

Contract definitions live in `extensions/{extension_name}/contract.yaml` and are validated before execution.

---

## Current Extensions

### Tier 1 (Operational)

| Extension | Purpose | Inputs | Outputs |
|---|---|---|---|
| `calibration_diagnostics` | Assess model calibration quality | Evaluation metrics, predictions | Calibration curves, reliability diagrams |
| `evaluation_summary_report` | Generate standardized model evaluation reports | Metrics JSON, model metadata | HTML/Markdown reports |
| `metric_aggregation` | Aggregate metrics across runs and configurations | Per-run metric files | Summary statistics, comparison tables |

### Planned Extensions

| Extension | Purpose | Phase |
|---|---|---|
| `cross_run_synthesis` | Compare and synthesize results across experimental runs | Phase 3 |
| `embedding_analysis` | Analyze learned representations from deep learning models | Phase 4 |
| `model_comparison_dashboard` | Interactive comparison of model architectures and performance | Phase 4 |
| `hypothesis_generator` | Suggest next experiments based on evaluation history | Phase 5 |

---

## Writing a New Extension

### Directory Structure

```
extensions/
  your_extension/
    contract.yaml
    run.py
    README.md
    tests/
      test_contract.py
```

### Contract Template

```yaml
name: your_extension
version: "0.1.0"
tier: 1

inputs:
  required:
    - path: "metrics/"
      description: "Per-run evaluation metrics"
    - path: "results/verified/"
      description: "Verified pipeline outputs"

outputs:
  directory: "results/extensions/your_extension/"
  files:
    - name: "output.json"
      description: "Primary analysis output"

validation:
  pre_run:
    - check: "files_exist"
      paths: ["metrics/summary.csv"]
```

### Development Guidelines

1. **Read-only artifact access.** Extensions must not write to any directory outside their designated output location.
2. **Deterministic output.** Given the same inputs, an extension must produce identical outputs. Use fixed seeds where randomness is involved.
3. **Schema compliance.** Contract declarations must match actual behavior. Pre-run validation checks are enforced.
4. **Self-contained dependencies.** Extensions should declare their own requirements and not assume packages beyond the base SNPTX environment.
5. **Test coverage.** Include contract validation tests that verify input requirements and output schema.

---

## Extension Execution

Extensions are invoked after the core pipeline completes:

```bash
# Run a specific extension
python -m extensions.calibration_diagnostics.run

# Run all tier-1 extensions
snakemake --snakefile Snakefile -R run_extensions
```

Extension outputs are written to `results/extensions/{extension_name}/` and are versioned alongside core artifacts.

---

## Extension Tiers

| Tier | Description | Review Level |
|---|---|---|
| Tier 1 | Core analysis capabilities shipped with SNPTX | Maintained by core team |
| Tier 2 | Community-contributed extensions with validated contracts | Contract review required |
| Tier 3 | Experimental extensions for research exploration | Self-serve, no guarantees |

Tier progression requires contract validation, test coverage, and documentation completeness.

---

## Deep Learning Extension Opportunities

The extension system is designed to support analysis of deep learning model outputs:

| Analysis Type | Extension Capability |
|---|---|
| Embedding visualization | Dimensionality reduction of learned representations, cluster analysis |
| Attention analysis | Transformer attention weight visualization and interpretation |
| Feature importance | Gradient-based attribution for deep learning predictions |
| Calibration under distribution shift | Out-of-distribution detection and calibration assessment |
| Cross-architecture comparison | Standardized evaluation across model families (CNN, transformer, GNN) |

These capabilities become available as the core pipeline integrates deep learning model types in Phases 4 and 5 of the roadmap.

---

## Contributing

See [CONTRIBUTING.md](../../snptx-extensions/CONTRIBUTING.md) for contribution guidelines, contract review process, and submission workflow.

---

## Contact

- drr508@g.harvard.edu
- dan@snptx.ai
