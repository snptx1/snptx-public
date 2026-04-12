# SNPTX Pilot Deployment Proposal

## Overview

SNPTX is available for pilot deployment in academic research labs working at the intersection of biomedical data science, drug discovery, and clinical machine learning. This document describes what a pilot engagement involves, what the framework provides, and how to evaluate fit.

---

## What SNPTX Provides

A pilot deployment delivers a configured instance of the SNPTX framework operating on a lab's own data and research questions. The framework handles:

- **Data ingestion** — Standardized adapters for clinical, omics, imaging, graph, text, single-cell, and molecular data
- **Model training** — Deterministic pipelines for classical ML and deep learning (XGBoost, GNNs, VAEs, transformers, ResNets, attention fusion)
- **Evaluation** — 8-check statistical audit with cross-validation, bootstrap CIs, conformal prediction, and trivial baselines
- **Experiment management** — MLflow tracking with full provenance for every run
- **Autonomous experimentation** — Self-learning engine that executes structured experimental campaigns (1,037 experiments/hour demonstrated)
- **Reproducibility** — Every result traceable to exact inputs, configuration, code version, and random seed

---

## Pilot Configurations

### Configuration A: Drug Discovery Pipeline

Deploy the GCN-based drug discovery pipeline with knowledge graph integration on lab-specific compound libraries.

**Requirements:** Molecular activity data (SMILES + labels), or access to ChEMBL/PubChem subsets.

**Deliverables:**
- Trained GCN model with bioactivity predictions
- Validation report (5-fold CV, bootstrap CIs, calibration analysis)
- Integration with existing compound screening workflows
- Artifact-versioned pipeline reproducible on any compatible environment

**Demonstrated performance:** 92.1% accuracy on ChEMBL bioactivity (binary), 91.8% ± 0.9% 5-fold CV.

---

### Configuration B: Multi-Modal Clinical Research

Deploy the multi-modal fusion pipeline combining clinical, omics, imaging, and/or text modalities.

**Requirements:** At least 2 modalities of patient-level or sample-level data with shared identifiers.

**Deliverables:**
- Per-modality baseline models with individual performance metrics
- Attention-based fusion model demonstrating cross-modal benefit
- Fusion holdout validation (leakage-free methodology)
- Uncertainty quantification with conformal prediction sets

**Demonstrated performance:** 88.3% fusion accuracy (holdout), exceeding best non-omics unimodal by 11.4 pp on Visium breast tissue.

---

### Configuration C: Autonomous Experimentation

Deploy the self-learning engine on lab-specific datasets to systematically explore model configurations.

**Requirements:** One or more supervised learning datasets (any modality SNPTX supports).

**Deliverables:**
- Autonomous campaign results: best configurations, discovery catalog, performance leaderboard
- Intelligence layer outputs: meta-analysis, adaptive defaults for the lab's data
- Experiment catalog (DuckDB) for post-hoc analysis and hypothesis generation
- Throughput benchmarks on lab-specific compute

**Demonstrated performance:** 1,100 experiments in 1.06 hours, 104 discoveries across 5 datasets and 6 model types.

---

### Configuration D: Cross-Institutional Reproducibility

Demonstrate bit-identical reproducibility of a reference pipeline across two compute environments (e.g., lab workstation + cloud instance).

**Requirements:** Two environments with Python 3.11+ and GPU access.

**Deliverables:**
- Identical outputs confirmed across environments (metrics, artifacts, model weights)
- Reproducibility report with artifact hash comparison
- Documentation of environment configuration for future replication

---

## Technical Requirements

### Minimum Compute

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 cores | 8+ cores |
| RAM | 16 GB | 32 GB |
| GPU | NVIDIA with 8 GB VRAM | A10G (23 GB) or better |
| Storage | 50 GB | 200 GB |
| Python | 3.11+ | 3.11.14 |
| OS | Ubuntu 22.04+ | Ubuntu 22.04 LTS |

### Software Stack (Installed via Requirements)

- PyTorch 2.5.1+cu121
- Snakemake, MLflow, DVC
- scikit-learn, XGBoost, pandas, scanpy
- Streamlit (for dashboard)
- All dependencies pinned in `requirements.txt`

---

## Engagement Model

### Phase 1: Scoping (1–2 weeks)

- Identify research question and available datasets
- Select pilot configuration (A, B, C, or D)
- Confirm compute environment compatibility
- Define success criteria

### Phase 2: Deployment (2–4 weeks)

- Configure SNPTX for lab-specific data
- Write custom dataset adapters if needed
- Execute reference pipeline and validate results
- Deploy dashboard for interactive exploration

### Phase 3: Evaluation (2–4 weeks)

- Run experimental campaigns on lab data
- Compare results against lab baselines
- Generate validation reports with 8-check audit
- Document findings for potential publication

### Phase 4: Continuation (ongoing)

- Framework remains operational for continued use
- Optional: extend to additional datasets or modalities
- Optional: contribute findings to SNPTX validation corpus

---

## What SNPTX Does Not Provide

- **Clinical decision-making tools.** SNPTX is a research framework, not a medical device. Results require expert interpretation.
- **Pre-trained models for clinical deployment.** Models are trained from scratch on provided data; no black-box predictions.
- **Data hosting or transfer.** Data remains on lab infrastructure. SNPTX processes data in-place.
- **Guaranteed performance on arbitrary tasks.** Reported metrics are benchmarks; performance on new data depends on data quality and task difficulty.

---

## Research Collaboration

Pilot deployments can serve as the technical foundation for:

- **Joint publications** — Framework methodology papers, domain-specific application papers
- **Benchmark contributions** — Adding validated results to the SNPTX multi-modal benchmark
- **Extension development** — Building domain-specific analytical modules through the extension system
- **Graduate research** — The framework provides infrastructure for thesis-level research in biomedical ML, systems engineering, or computational biology

---

## Contact

Dan Russell  
Harvard ALM in Data Science, '27 | MITx Statistics and Data Science  
drr508@g.harvard.edu | dan@snptx.ai

For technical documentation: [ARCHITECTURE.md](ARCHITECTURE.md) | [METHODS.md](METHODS.md) | [ROADMAP.md](ROADMAP.md)
