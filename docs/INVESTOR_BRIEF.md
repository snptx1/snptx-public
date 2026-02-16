# SNPTX Investor Brief

## Problem

Biomedical machine learning generates enormous experimental volume, but lacks the infrastructure to make that experimentation cumulative. Labs run thousands of model variants without systematic comparison. Results drift as downstream analysis modifies interpretation. No single system connects experimentation, evaluation, and hypothesis generation into a closed loop.

The reproducibility crisis in biomedical AI is fundamentally an infrastructure problem.

---

## Solution

SNPTX is a biomedical ML infrastructure framework that enforces deterministic execution, tracks every experiment with full provenance, versions all artifacts immutably, and provides a structured extension system for downstream analysis. It is designed to evolve toward a self-learning architecture where each experimental cycle informs the next.

![Discovery Loop](assets/discovery_loop.png)

SNPTX does not replace existing tools. It integrates Snakemake (orchestration), MLflow (experiment tracking), and DVC (artifact versioning) into a unified framework with architectural constraints that guarantee reproducibility and enable cumulative learning.

---

## Architecture Moat

SNPTX's competitive advantage is architectural, not feature-based:

1. **Deterministic DAG execution.** Snakemake pipelines with fixed seeds, versioned configuration, and rule-level provenance.
2. **Structured extension boundary.** Extensions consume artifacts read-only, preventing analytical drift and enabling standardized model comparison.
3. **Immutable artifact lineage.** Every model, metric, and evaluation output is versioned and traceable to its inputs, configuration, and code version.
4. **Deep learning integration trajectory.** Architecture supports transformer-based models, graph neural networks, representation learning, and multi-modal fusion as first-class pipeline components.
5. **Self-learning feedback design.** Evaluation artifacts feed back into configuration synthesis and hypothesis generation, creating closed-loop experimentation.
6. **Agentic workflow architecture.** The deterministic, contract-driven design enables autonomous AI agents to author extensions, execute pipelines, and synthesize results without manual intervention.

---

## Deep Learning Architecture Directions

| Direction | Application Domain | SNPTX Integration |
|---|---|---|
| Transformers | Clinical text, genomic sequences | Pipeline rule, embedding registry |
| Graph Neural Networks | Protein interaction, drug-target networks | DAG-orchestrated, artifact-versioned |
| Representation Learning | Low-dimensional biomedical feature spaces | Embedding persistence, cross-run comparison |
| Multi-Modal Fusion | Imaging + genomics + clinical records | Parallel pipeline branches, unified evaluation |
| Contrastive Learning | Label-efficient biomedical pretraining | Extension-based evaluation, calibration diagnostics |
| Foundation Model Fine-Tuning | Domain adaptation for clinical tasks | MLflow tracking, artifact lineage, hyperparameter management |

---

## Technical Differentiation

![Positioning Matrix](assets/positioning_matrix.png)

| Capability | MLflow | DVC | Kubeflow | SNPTX |
|---|---|---|---|---|
| DAG orchestration | No | Partial | Yes | Yes |
| Experiment tracking | Yes | Partial | Yes | Yes |
| Artifact versioning | Partial | Yes | Partial | Yes |
| Structured extensions | No | No | No | Yes |
| Self-learning trajectory | No | No | No | Planned |

No existing platform combines all five capabilities. SNPTX occupies this unique intersection.

---

## Early Traction

- **Core pipeline operational.** Snakemake DAG orchestrates training, evaluation, and artifact persistence.
- **MLflow integration live.** Experiment tracking with parameterized sweeps across multiple configurations.
- **DVC artifact versioning configured.** S3 and GCS remote storage for model and data artifacts.
- **Extension system functional.** First-tier extensions (calibration diagnostics, evaluation summary reports, metric aggregation) operational.
- **CI pipeline configured.** GitHub Actions for dry-run validation on every push.

---

## Roadmap

| Phase | Focus | Timeline |
|---|---|---|
| 1. Foundation | Deterministic training, evaluation, artifact persistence | Complete |
| 2. Extension System | Contract-driven downstream analysis model | Complete |
| 3. Multi-Run Intelligence | Cross-experiment synthesis, ensemble selection | Near-term |
| 4. Deep Learning Integration | Transformers, GNNs, foundation models, embedding registry | Medium-term |
| 5. Self-Learning Feedback | Configuration synthesis from evaluation history | Medium-term |
| 6. Multi-Modal Expansion | Imaging + genomics + clinical fusion, federated deployment | Long-term |

---

## Funding Direction

Seeking pre-seed investment to:

1. Expand deep learning integration across biomedical modalities (text, genomics, imaging, molecular graphs).
2. Build cross-run synthesis and self-learning feedback infrastructure.
3. Recruit domain-specific ML engineers for clinical and pharmaceutical verticals.
4. Establish pilot partnerships with academic research labs and pharma R&D teams.

---

## Team

- **Dan Russell** (Founder), MITx (SDS), Harvard ALM (DS) '27. Systems architecture, biomedical ML infrastructure.

---

## Documentation

For detailed technical architecture, see [ARCHITECTURE.md](ARCHITECTURE.md).  
For the development roadmap, see [ROADMAP.md](ROADMAP.md).  
For the extension development model, see [DEVKIT_NOTES.md](DEVKIT_NOTES.md).  
For strategic positioning, see [POSITIONING.md](POSITIONING.md).  

---

## Repository Structure

```
snptx-public/
+-- README.md
+-- docs/
    +-- ARCHITECTURE.md
    +-- ROADMAP.md
    +-- DEVKIT_NOTES.md
    +-- VISION.md
    +-- POSITIONING.md
    +-- INVESTOR_BRIEF.md
    +-- assets/
        +-- agentic_workflow.png
        +-- discovery_loop.png
        +-- extension_lifecycle.png
        +-- multimodal_framework.png
        +-- positioning_matrix.png
        +-- self_learning_trajectory.png
        +-- snptx_architecture.png
        +-- SNPTX_Project_roadmap.png
        +-- workflow_dag.png
```

This repository contains documentation and architectural specifications only. Source code, datasets, model artifacts, and execution scripts are maintained in private repositories.

---

## Team

- **Dan Russell** (Founder), MITx (SDS), Harvard ALM (DS) '27. Systems architecture, biomedical ML infrastructure.  

- drr508@g.harvard.edu
- dan@snptx.ai
