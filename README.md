# SNPTX

## Deterministic Orchestration for Reproducible Biomedical Machine Learning

SNPTX is a framework for biomedical machine learning that enforces deterministic execution, full artifact provenance, and structured evaluation across data modalities. It addresses the reproducibility gap in computational biology by providing infrastructure-level guarantees: identical inputs produce identical outputs across runs, environments, and time.

The system orchestrates the complete experimental lifecycle — data ingestion, model training, evaluation, cross-run synthesis, and hypothesis-driven feedback — within a pipeline architecture where every artifact is versioned, every configuration is explicit, and every result is traceable to its inputs.

SNPTX currently supports 8 data modality families across 46 dataset adapters, validated on real biomedical data with 1,056 tests passing and zero static analysis violations.

---

## Research Motivation

Computational workflows in genomics, clinical prediction, and drug discovery routinely fail to reproduce across labs and time horizons (Baker, 2016; Hutson, 2018). The causes are infrastructural: uncontrolled analytical drift, implicit state dependencies, and the absence of structured evaluation frameworks. An estimated 70% of computational biology results fail to reproduce when re-executed by independent groups.

SNPTX addresses these root causes architecturally rather than procedurally:

| Root Cause | Architectural Response |
|---|---|
| Analytical drift | Extensions are downstream-only, consuming immutable artifacts |
| Implicit state | All configuration is explicit; DAG ordering is deterministic |
| Unstructured comparison | Standardized evaluation artifacts enable systematic model comparison |
| Missing provenance | Artifact versioning with input hashing, commit embedding, and experiment logging |

---

## System Architecture

The framework separates **execution** (what runs and when) from **interpretation** (how results are analyzed and compared). This separation enables model architectures, evaluation methods, and feedback strategies to evolve independently.

### Core Infrastructure

| Component | Role | Implementation |
|---|---|---|
| **Pipeline Orchestration** | Reproducible DAGs from ingestion through evaluation | Snakemake (34 rules, 9 core DAG + 25 opt-in) |
| **Experiment Tracking** | Parameters, metrics, artifacts, and model versions | MLflow |
| **Artifact Management** | Dataset, model, and derived artifact versioning | DVC (48 pipeline stages) |
| **Configuration & Lineage** | Explicit control over datasets, features, and parameters | YAML |

### Extension System

Tier-1 extensions form the analytical surface for downstream interpretation. All extensions are strictly downstream of the core pipeline, consume and produce artifacts only (no shared state), declare explicit typed input/output contracts, and produce deterministic outputs. Extensions live in a physically separate repository, isolating analytical contributions from orchestration logic.

### Artifact-Driven Flow

```
Data Ingestion → Training → Evaluation → Diagnostics → Aggregation → Reporting → Feedback
```

Each stage boundary is defined by persisted, immutable artifacts. No stage reaches into the implementation of another, enabling independent verification at every boundary.

For detailed architecture, see [ARCHITECTURE.md](docs/ARCHITECTURE.md).

---

## Multi-Modal Experimental Results

SNPTX has been validated across 7 biomedical data modalities plus multi-modal fusion, each trained on real-world biomedical datasets with holdout evaluation and 8-check statistical audit.

### Per-Modality Performance

| Modality | Dataset | Model | Accuracy | Validation |
|---|---|---|---|---|
| Clinical tabular | Synthea readmission (n=6,625) | XGBoost | 77.2% | 5-fold CV, 76.4% ± 1.0% |
| Omics | Visium breast (200 HVG, n=3,798) | VAE | 89.6% | 5-fold CV, 89.9% ± 0.9% |
| Knowledge graphs | Hetionet link prediction (n=20,000) | GCN | 75.2% | 5-fold CV, 75.4% ± 1.2% |
| Histopathology | Visium H&E patches (n=3,798) | ResNet-50 | 36.4% | 8-class, random baseline 12.5% |
| Clinical text | MTSamples (n=4,652) | BERT | 34.1% | 22-specialty, random baseline 4.5% |
| Single-cell | PBMC 3k (n=2,700) | VAE | 76.9% | 5-fold CV, 76.5% ± 1.1% |
| Drug discovery | ChEMBL bioactivity (n=4,685) | GCN | 92.1% | 5-fold CV, 91.8% ± 0.9% |
| **Multi-modal fusion** | **Visium (omics + imaging + spatial)** | **Attention fusion** | **88.3%** | **Holdout; exceeds best non-omics unimodal (76.9%)** |

### Validation Methodology (8-Check Audit)

Every modality result is validated through an 8-check statistical audit framework:

1. **k-fold cross-validation** — 5-fold stratified CV with variance reporting
2. **Multi-seed stability** — 5-seed evaluation confirming variance < 1.2%
3. **Bootstrap confidence intervals** — 95% CIs on all reported metrics
4. **Spatial stratification** — Prevents spatial autocorrelation leakage in tissue data
5. **Fusion holdout** — PCA embeddings extracted from training set only; no leakage into test
6. **Conformal prediction** — Distribution-free prediction sets with coverage guarantees
7. **Learning curves** — Confirms models are not in data-starved regime
8. **Trivial baselines** — All models verified against majority-class and random baselines

---

## Self-Learning Intelligence Layer

The intelligence layer provides closed-loop feedback between evaluation outputs and upstream pipeline configuration, enabling the system to learn from its own experimental history.

### Core Intelligence (B1–B5)

| Module | Function | Theory |
|---|---|---|
| Experiment Catalog | DuckDB-backed persistent store, 16 columns, indexed | Cumulative experimentation |
| Meta-Analysis Engine | Cross-experiment pattern discovery, best-config ranking | Bayesian rank aggregation |
| Feedback Loop v2 | Bayesian confidence updating, Thompson sampling | Beta-Binomial posterior (Agrawal & Goyal, 2012) |
| Adaptive Defaults | Dataset-aware starting configuration | Sigmoid confidence scoring |
| Hypothesis Templates | 7 declarative templates with trigger conditions | Declarative hypothesis systems |

### Theoretical Intelligence (B.6.1–B.6.10)

| Module | Theory | Citation |
|---|---|---|
| Meta-Features | Algorithm selection from dataset characteristics | Rice, 1976 |
| Surrogates | GP-based Bayesian optimization with acquisition functions | Snoek et al., 2012 |
| Causal Feedback | ATE, IPW, interrupted time series validation | Pearl, 2009 |
| Experiment Design | Information gain, SPRT, active learning | Chaloner & Verdinelli, 1995 |
| Rule Mining | ILP rules with probabilistic calibration, Bayesian networks | Muggleton & de Raedt, 1994 |
| Multi-Objective | NSGA-II Pareto optimization with fairness constraints | Deb et al., 2002 |
| Continual Learning | EWC, drift detection, experience replay | Kirkpatrick et al., 2017 |
| Bayesian Testing | Signed-rank with ROPE, Friedman, bootstrap | Benavoli et al., 2017 |
| Information Theory | Entropy, MI, MDL, KL divergence, feature ranking | Cover & Thomas, 2006 |
| Scientific Discovery | Surprise detection, symbolic regression, novelty search | Peirce, 1903 |

---

## Theoretical Multi-Modal Hardening (Phase C.6)

Six modules providing formal theoretical grounding for cross-modal learning:

| Module | Methods | Theoretical Basis |
|---|---|---|
| Cross-modal alignment | InfoNCE, CLIP-style, Deep CCA | Information Bottleneck (Tishby et al., 2000) |
| Uncertainty quantification | Evidential DL, conformal prediction, per-modality calibration | Sensoy et al., 2018; Vovk et al., 2005 |
| E(n)-equivariant graphs | SE(3)-equivariance for molecular geometry | Thomas et al., 2018; Satorras et al., 2021 |
| Tensor fusion | Multiplicative outer products, Mixture-of-Experts | Zadeh et al., 2017 |
| Domain-adaptive NLP | Biomedical NER, negation detection, masked LM | Lee et al., 2020 (BioBERT) |
| Masked autoencoders | Patch reconstruction, domain divergence bounds | He et al., 2022 (MAE) |

---

## Autonomous Experimentation Engine

The autonomous engine executes structured experimental campaigns without manual intervention, using the intelligence layer to guide exploration.

| Metric | Value |
|---|---|
| Experiments completed | 1,100 |
| Discoveries identified | 104 (8 new-best + 96 novel-config) |
| Throughput | 1,037 experiments/hour |
| Elapsed time | 1.06 hours |
| Datasets | 5 (iris, wine, breast_cancer, digits, synthea_readmission) |
| Model types | 6 (logistic regression, gradient boosting, MLP, XGBoost, SVM, random forest) |
| Best per-dataset | Iris 100%, Wine 100%, Digits 99.4%, Breast Cancer 97.4%, Synthea 78.0% |

Discovery types are wired through the intelligence layer: `new_best` (improved top score), `novel_config` (previously untested configuration with competitive performance), and `surprise` (anomalous result warranting investigation).

---

## Supported Model Architectures

| Architecture | Application Domain | Status |
|---|---|---|
| **Transformers** | EHR sequence modeling, clinical note embedding, omics feature extraction | Validated |
| **Graph neural networks** | Molecular property prediction, knowledge graph reasoning, drug-target interaction | Validated |
| **Variational autoencoders** | Gene expression embeddings, single-cell representation learning | Validated |
| **Convolutional networks** | Histopathology classification, medical image feature extraction | Validated |
| **Multi-modal fusion** | Cross-modality prediction (attention-based, tensor fusion, MoE) | Validated |
| **Contrastive learning** | Patient similarity, cohort discovery, representation alignment | Validated |
| **Foundation model fine-tuning** | Domain adaptation of pre-trained biomedical models (BioBERT, etc.) | Validated |

All architectures operate within the same Snakemake + MLflow + DVC infrastructure. Model complexity scales; reproducibility guarantees remain constant.

---

## Embedding Registry

SNPTX maintains a versioned embedding registry: an artifact-driven store of learned representations linked to specific model versions, datasets, and training configurations. This enables:

- Downstream extensions to consume representations as stable artifacts
- Cross-model embedding comparison through standardized interfaces
- Transfer learning workflows with full provenance

---

## Current Framework Status

| Metric | Value |
|---|---|
| Total tests | 1,056 |
| Static analysis | 0 pyright errors, 0 ruff violations |
| Dataset adapters | 46 (37 production + 9 deferred) |
| Modality families | 8 |
| Intelligence modules | 15 (5 core + 10 theoretical) |
| Multi-modal modules | 6 (alignment, graph, imaging, NLP, fusion, uncertainty) |
| Autonomous experiments | 1,100 |
| Autonomous discoveries | 104 |
| Dashboard | 9 interactive pages (Streamlit) |
| Infrastructure | Python 3.11, PyTorch 2.5.1+cu121, EC2 g5.xlarge (A10G GPU) |

### Phase Completion

| Phase | Scope | Tests | Status |
|---|---|---|---|
| A | Data foundation (46 adapters, 8 modalities) | 247 | Complete |
| A.6 | Theoretical data hardening (10 modules) | 85 | Complete |
| B | Intelligence layer (5 core modules) | 70 | Complete |
| B.6 | Theoretical intelligence (10 modules) | 485 | Complete |
| C | Multi-modal expansion (7 modalities + fusion) | 169 | Complete |
| C.6 | Theoretical multi-modal hardening (6 modules) | — | Complete |
| D | Clinical deployment readiness | — | In Progress |

---

## Pilot Deployment

SNPTX is available for pilot deployment in academic research labs. The framework is designed for environments where reproducibility, provenance, and structured experimentation are primary requirements. Potential pilot configurations include:

- **Drug discovery pipelines** — GCN-based bioactivity prediction with knowledge graph integration
- **Multi-modal clinical research** — Combining clinical, omics, imaging, and text modalities
- **Autonomous experimentation** — Deploying the self-learning engine on lab-specific datasets
- **Cross-institutional reproducibility** — Demonstrating identical outputs across environments

For pilot deployment inquiries, see [PILOT_PROPOSAL.md](docs/PILOT_PROPOSAL.md).

---

## Documentation

| Document | Description |
|---|---|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | System architecture, layer design, artifact flow |
| [ROADMAP.md](docs/ROADMAP.md) | Phase-by-phase development timeline with milestones |
| [METHODS.md](docs/METHODS.md) | Experimental methodology, datasets, validation protocols |
| [PILOT_PROPOSAL.md](docs/PILOT_PROPOSAL.md) | Pilot deployment scope, requirements, engagement model |
| [DEVKIT_NOTES.md](docs/DEVKIT_NOTES.md) | Extension development model and API contracts |
| [VISION.md](docs/VISION.md) | Long-term research trajectory |

---

## Repository Structure

```
snptx-public/
├── README.md
└── docs/
    ├── ARCHITECTURE.md
    ├── ROADMAP.md
    ├── METHODS.md
    ├── PILOT_PROPOSAL.md
    ├── DEVKIT_NOTES.md
    ├── VISION.md
    └── assets/
        ├── discovery_loop.png
        ├── multimodal_framework.png
        ├── snptx_architecture.png
        ├── SNPTX_Project_roadmap.png
        ├── workflow_dag.png
        └── ...
```

This repository contains documentation and architectural specifications. Source code, datasets, model artifacts, and execution scripts are maintained in private repositories. Access is available for pilot collaborators.

---

**SNPTX**  
Deterministic Orchestration for Reproducible Biomedical ML

Dan Russell  
Harvard ALM in Data Science, '27 | MITx Statistics and Data Science  
drr508@g.harvard.edu | dan@snptx.ai
