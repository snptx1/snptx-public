# SNPTX Architecture

This document defines the technical architecture of the SNPTX system. It describes the core framework, the extension governance model, artifact flows, deterministic execution guarantees, and the collaboration philosophy that together enable reproducible, auditable biomedical machine learning workflows.

---

## 1. Design Principles

SNPTX is built on six architectural commitments that constrain all design decisions:

**Modularity.** Core orchestration and downstream analytical logic are strictly separated. Each extension operates as an independent, testable unit with explicit inputs and outputs. No extension may import, modify, or depend on core implementation details.

**Reproducibility.** Data, models, and analytical artifacts are versioned. Identical inputs must yield identical outputs across runs. This is a structural guarantee, not a best-practice suggestion.

**Determinism by Design.** Downstream extensions emit stable, schema-defined artifacts with deterministic ordering and formatting. Random seeds are fixed. Configuration state is explicit. Implicit dependencies are prohibited.

**Traceability.** Every artifact can be traced to upstream inputs, configuration state, and execution context without ambiguity. Provenance is preserved through lineage metadata, not through inspection of intermediate state.

**Collaboration Without Loss of Control.** External contributors work within a constrained extension surface. Execution remains owner-mediated. The system maximizes collaboration while preserving architectural integrity.

**Extensibility.** The system supports future learning methods, intelligent analysis layers, and interactive interfaces without refactoring the core. New capabilities are added as extensions, not as core modifications.

---

## 2. System Layers

SNPTX is organized into distinct layers, each with a defined purpose and clear boundaries:

| Layer | Purpose | Implementation |
|---|---|---|
| **Data & Knowledge Ingestion** | Ingest, validate, and version raw datasets and derived features | CSV pipelines, DVC, cloud storage |
| **Core Framework** | Orchestrate pipelines, manage experiments, control configuration and lineage | Snakemake, MLflow, DVC, YAML |
| **Evaluation & Diagnostics Extensions** | Downstream evaluation, diagnostics, and metric normalization | Tier-1 governed extensions |
| **Reporting & Interface Extensions** | Human-readable summaries and machine-consumable reports | Tier-1 governed extensions |
| **Governance & Audit Layer** | Provenance tracking, contract validation, system documentation | Built into core and runner |
| **Automation & Optimization** *(Future)* | Feedback-driven learning, resource optimization | Planned |
| **Model & Learning Extensions** *(Future)* | Advanced learning methods and model libraries | Planned |

![SNPTX Architecture](assets/snptx_architecture.png)

---

## 3. Core Framework

The SNPTX Core Framework is intentionally narrow in scope. It is responsible for exactly four concerns:

### 3.1 Pipeline Orchestration

Snakemake defines reproducible Directed Acyclic Graphs (DAGs) connecting data ingestion, validation, training, evaluation, summarization, and reporting. Each pipeline stage is an independent rule with explicit inputs and outputs. The DAG enforces execution ordering and enables incremental re-execution when inputs change.

The pipeline includes:
- Data validation against schema definitions
- Deterministic train/test splitting with fixed seeds
- Parameterized model training with configurable sweeps
- Automated evaluation and metric generation
- Summary aggregation and report generation

### 3.2 Experiment Tracking

MLflow records parameters, metrics, artifacts, and model versions for every training and evaluation run. The experiment registry provides:
- Run-level parameter and metric logging
- Artifact association (models, plots, confusion matrices)
- Model version tracking and promotion stages
- Programmatic access to historical run data

### 3.3 Artifact Management

DVC manages datasets, trained models, and derived artifacts across local and remote storage. Artifact management ensures:
- Large file versioning outside of Git
- Remote storage synchronization
- Reproducible artifact retrieval across environments

### 3.4 Configuration and Lineage Control

YAML-based configuration files define datasets, features, target columns, parameter ranges, and experiment metadata. Configuration is the sole mechanism for controlling pipeline behavior — no code branching for dataset or parameter changes.

**The core does not implement analytical interpretation logic.** That responsibility belongs exclusively to the extension layer.

---

## 4. Execution Isolation Model

The boundary between core and extensions is the central architectural constraint of SNPTX.

### What the Core Controls
- Pipeline DAG definition and execution ordering
- Model training and basic evaluation
- Artifact generation and versioning
- Experiment metadata logging
- Extension invocation (via the owner-runner)

### What Extensions Control
- Downstream evaluation and diagnostics
- Metric normalization and aggregation
- Human-readable report generation
- Cohort and slice analysis *(future)*
- Intelligent summarization *(future)*

### What Neither May Do
- Extensions may not modify upstream artifacts
- Extensions may not import core code
- The core may not embed analytical interpretation logic
- Neither may introduce implicit state dependencies

This separation is enforced through physical repository isolation, contract validation at runtime, and owner-mediated invocation.

---

## 5. Tier-1 Extension System

Tier-1 extensions are the primary mechanism for safe, structured collaboration.

### 5.1 Location and Isolation

All Tier-1 extensions live in a separate repository (`snptx-extensions`). They are never implemented inside the core repository. This physical separation ensures that analytical contributions cannot inadvertently modify orchestration, training, or configuration logic.

### 5.2 Execution Model

Extensions are:
- **Strictly downstream** — they consume outputs of the core pipeline
- **Artifact-only** — no shared memory, no runtime coupling, no database access
- **Owner-mediated** — invoked exclusively through the SNPTX owner-runner
- **Deterministic** — identical inputs must produce identical outputs
- **Auditable** — every invocation generates a manifest, captured logs, and hashed run IDs

Direct execution of extensions is prohibited. The owner-runner validates contracts, captures stdout/stderr, computes deterministic run identifiers from hashed inputs, and writes auditable manifests.

### 5.3 Contract System

Each extension declares an `extension.yaml` contract specifying:
- Input artifact requirements (names, types, schemas)
- Output artifact declarations (names, types, locations)
- Configuration parameters
- ABI version compatibility
- Compliance and provenance metadata

The owner-runner validates these contracts at invocation time. Missing inputs, type mismatches, and schema violations are rejected before execution begins.

### 5.4 Current Tier-1 Modules

| Extension | Category | Primary Output |
|---|---|---|
| `calibration_diagnostics` | Evaluation & Diagnostics | Calibration diagnostics JSON |
| `metric_aggregation` | Evaluation & Diagnostics | Canonical `metrics.json` (aggregated) |
| `evaluation_summary_report` | Reporting & Interface | Human-readable Markdown report |
| `artifact_contract_checks` | Governance | Contract validation report |

These modules form a coherent downstream analytical pipeline from raw evaluation outputs through normalized metrics to human-readable summaries.

---

## 6. Artifact Flow

The SNPTX artifact flow enforces a strict linear progression from computation to interpretation:

```
1. Upstream Training & Evaluation
   └─ Core pipeline emits evaluation artifacts (metrics JSON, confusion matrices, model files)

2. Diagnostics & Normalization
   └─ Tier-1 evaluation extensions analyze and normalize metrics

3. Aggregation Layer
   └─ Canonical artifacts (metrics.json) provide stable interfaces between analytics and reporting

4. Reporting
   └─ Reporting extensions consume canonical artifacts to generate human-readable summaries
```

Each stage boundary is defined by persisted, immutable artifacts. No stage may reach into the implementation of another. This architecture enables independent verification, reproducible replay, and safe extension development.

![Workflow DAG](assets/workflow_dag.png)

---

## 7. Governance and Audit

### 7.1 Provenance Tracking

Every extension invocation produces:
- A `manifest.json` containing runner version, timestamp, extension metadata, input hashes, output paths, and computed run ID
- Captured `stdout.log` and `stderr.log`
- Deterministic run IDs computed from the hash of extension identity, input content, and configuration

### 7.2 Artifact Immutability

Artifacts are immutable once written. The artifact semantics contract (documented separately) guarantees:
- Artifact paths remain stable for the duration of each contract tier
- Schema and semantic meaning do not change within a tier
- Breaking changes require a new contract version

### 7.3 Violation Policy

Extensions that assume undocumented artifact behavior, mutate upstream artifacts, or infer semantics from core source code are rejected without exception.

---

## 8. The Feedback Loop (Architectural Design Target)

The SNPTX architecture is designed to support a future closed-loop feedback mechanism between evaluation outputs and upstream pipeline behavior. This feedback loop is an architectural target, not a current capability.

When implemented, the feedback loop will enable:
- Insight synthesis across multiple evaluation runs
- Structured hypothesis suggestion based on result patterns
- Optional learning feedback signals (without automatic retraining)
- Intelligent summarization driven by accumulated analytical artifacts

The critical design constraint is that feedback must operate through the same artifact-driven, contract-governed interface as all other extensions. Feedback will not bypass determinism guarantees or introduce implicit state.

---

## 9. Directory Structure

### Core Repository

| Directory | Contents |
|---|---|
| `configs/` | Dataset, feature, and parameter definitions |
| `workflow/` | Snakemake DAG and core execution scripts |
| `data/` | DVC-managed datasets (raw, processed, splits) |
| `results/` | Models, metrics, plots, extension outputs |
| `metrics/` | Evaluation metrics and summaries |
| `reports/` | Generated reports (Markdown, HTML) |

### Extensions Repository

| Directory | Contents |
|---|---|
| `extensions/` | Tier-1 extension implementations |
| `docs/tier1/` | Tier-1 module specifications |
| `docs/artifact_semantics.md` | Shared artifact definitions and contract |

---

## 10. Development Roadmap

![SNPTX Roadmap](assets/SNPTX_Project_roadmap.png)

## 11. Future Extension Directions

The following capabilities are planned as additive extensions. None requires changes to the core framework:

- **Cohort and slice analysis** — per-cohort metric comparison and deterministic ranking
- **Explainability modules** — SHAP values, feature importance, and interpretability reports
- **Intelligent summarization** — artifact-driven insight synthesis across runs
- **Interactive dashboards** — read-only visualization backed by canonical artifacts
- **API deployment layer** — deterministic artifact serving through structured endpoints
- **Multi-modality support** — clinical, omics, imaging, and knowledge graph workflows via configuration

---

## Contact

For architectural questions or collaboration inquiries:
- drr508@g.harvard.edu
- dan@snptx.ai
