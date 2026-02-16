# SNPTX Roadmap

This document defines the phased development plan for SNPTX, aligning technical milestones with the architectural separation between core framework and governed extension ecosystem.

The roadmap is a living reference. Completed phases reflect operational capabilities. Planned phases reflect committed architectural targets with defined technical prerequisites.

---

## Guiding Objectives

1. Build a reproducible, deterministic data-to-insight ML framework for biomedical applications.
2. Separate pipeline orchestration from analytical interpretation at every layer.
3. Enable safe collaboration through constrained, artifact-only extensions.
4. Maintain lineage integrity and execution guarantees as the system scales.
5. Progress toward intelligent, interactive layers only after core interfaces stabilize.

---

## Roadmap Overview

| Phase | Focus | Status |
|---|---|---|
| **Phase 1** | Deterministic Core | Complete |
| **Phase 2** | Training + Registry | Complete |
| **Phase 3** | Evaluation + Promotion | Complete |
| **Phase 4** | Self-Learning Loop | Planned |
| **Phase 5** | Deployment + Interface Layer | Planned |
| **Phase 6** | Multi-Dataset Expansion | Planned |

![SNPTX Roadmap](assets/SNPTX_Project_roadmap.png)

---

## Phase 1: Deterministic Core (Complete)

**Objective:** Establish a stable, reproducible execution foundation.

- [x] Cloud infrastructure setup with persistent environment
- [x] Repository scaffolding and Python environment isolation
- [x] Snakemake DAG for training, evaluation, and reporting
- [x] MLflow experiment tracking and model registry integration
- [x] DVC dataset and model versioning with remote storage
- [x] YAML-based configuration for datasets, features, and parameters
- [x] CI validation via GitHub Actions
- [x] Deterministic train/test splitting with fixed seeds

**Outcome:** A reproducible orchestration core where identical inputs produce identical outputs. Pipeline execution is fully defined by configuration and DAG structure.

---

## Phase 2: Training + Registry (Complete)

**Objective:** End-to-end reproducible pipeline from raw data to registered models and reports.

- [x] Parameter sweeps via Snakemake wildcards
- [x] Automatic model evaluation and metric comparison
- [x] Confusion matrix and structured metric generation
- [x] Model registration in MLflow with version tracking
- [x] Markdown and HTML report generation with commit provenance
- [x] Git commit embedding in artifacts for traceability
- [x] Multi-dataset ingestion support via configuration
- [x] Fully materialized `results/` outputs

**Outcome:** Complete data-to-report pipeline. Every training run produces versioned models, structured metrics, and traceable reports.

---

## Phase 3: Evaluation + Promotion (Complete)

**Objective:** Establish a governed extension surface for downstream analysis and collaboration.

- [x] Separate `snptx-extensions` repository established
- [x] Tier-1 extension specification format defined
- [x] Owner-runner execution boundary enforced with contract validation
- [x] Artifact-only, downstream execution model implemented
- [x] Shared artifact semantics documented and frozen for Tier 1
- [x] Reference Tier-1 modules implemented:
  - [x] `calibration_diagnostics` — evaluation diagnostics
  - [x] `metric_aggregation` — canonical metric normalization
  - [x] `evaluation_summary_report` — human-readable reporting
- [x] Deterministic outputs written to core results directory
- [x] Auditable manifest generation for every extension invocation

**Outcome:** A safe, reviewable collaboration surface. Contributors can build analytical modules without touching orchestration, training logic, or execution control.

---

## Phase 4: Self-Learning Loop (Planned)

**Objective:** Enable structured feedback from evaluation outputs to upstream pipeline behavior.

**Next Technical Horizon**

- [ ] Artifact contract validation extension (schema enforcement at runtime)
- [ ] Extension schema convergence (single canonical contract format)
- [ ] Canonical artifact adoption across all reporting modules
- [ ] Intelligent summarization extension (artifact-driven insight synthesis)
- [ ] Structured hypothesis suggestion based on cross-run result patterns
- [ ] Feedback artifact specification (`feedback.json` contract)
- [ ] Closed-loop demonstration: evaluation → feedback → configuration adjustment

**Design Constraints:**
- Feedback must operate through the artifact-driven, contract-governed interface
- No automatic retraining without explicit owner authorization
- Determinism guarantees must be preserved through the feedback path
- Feedback signals are optional and advisory, not prescriptive

**Outcome:** Insight augmentation layered atop stable analytical artifacts. The system becomes capable of learning from its own evaluation outputs without sacrificing reproducibility.

---

## Phase 5: Deployment + Interface Layer (Planned)

**Objective:** Expose deterministic artifacts through user-facing interfaces and structured APIs.

**Commercialization Horizon**

- [ ] Dashboard backed by deterministic, canonical artifacts
- [ ] API endpoints for reports, metrics, and model metadata
- [ ] Read-only external access patterns (no mutation through interfaces)
- [ ] Slice-aware metric aggregation and cohort comparison extensions
- [ ] Per-cohort comparison artifacts with deterministic delta outputs
- [ ] User testing and feedback integration
- [ ] Enterprise packaging and multi-team deployment support

**Outcome:** User-facing access to SNPTX outputs without compromising execution guarantees. The interface layer consumes artifacts; it does not produce them.

---

## Phase 6: Multi-Dataset Expansion (Planned)

**Objective:** Demonstrate framework generality across biomedical data modalities.

- [ ] Clinical classification and survival analysis datasets
- [ ] Omics data integration (embeddings, biomarker discovery)
- [ ] Imaging pipeline support (CNN and transformer architectures)
- [ ] Knowledge graph and NLP-derived embedding workflows
- [ ] Cross-modality comparison through standardized artifact interfaces
- [ ] Enhanced model registry workflows with multi-model tracking

**Outcome:** SNPTX operates across data modalities through configuration, not code branching. Each track emits standardized artifacts for cross-modality comparability.

---

## Milestone Summary

| Milestone | Phase | Status |
|---|---|---|
| Reproducible orchestration core | 1 | Complete |
| End-to-end training pipeline | 2 | Complete |
| Governed extension ecosystem | 3 | Complete |
| Schema-driven contract convergence | 4 | Next |
| Intelligent summarization | 4 | Planned |
| API and dashboard layer | 5 | Planned |
| Multi-modality support | 6 | Planned |

---

## Ongoing Practices

- Architecture, README, and roadmap are kept synchronized
- Artifact schemas are treated as public contracts with versioned stability guarantees
- New capabilities are added as extensions, never as core modifications
- Milestones are tagged in Git for historical reference

---

## Contact

Roadmap questions or collaboration inquiries:
- drr508@g.harvard.edu
- dan@snptx.ai
