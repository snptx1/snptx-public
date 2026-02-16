# SNPTX Technical Positioning

![SNPTX Architecture](assets/snptx_architecture.png)

---

## Why SNPTX Is Not Just MLOps

MLOps tooling solves deployment efficiency: how to move models from notebooks to production. SNPTX solves a different problem: how to ensure that the analytical lifecycle — from data ingestion through interpretation — remains reproducible, auditable, and governable as complexity scales.

The distinction is architectural, not cosmetic.

---

## SNPTX vs. MLflow-Only Workflows

MLflow provides experiment tracking, model registration, and artifact logging. These are essential utilities. SNPTX uses MLflow for exactly these purposes.

What MLflow does not provide:

| Capability | MLflow | SNPTX |
|---|---|---|
| Pipeline orchestration (DAG-based execution) | No | Yes (Snakemake) |
| Deterministic execution guarantees | No | Yes (fixed seeds, versioned config, DAG ordering) |
| Artifact immutability as architectural constraint | No | Yes |
| Extension governance model | No | Yes (contract-driven, owner-mediated) |
| Separation of execution from interpretation | No | Yes (core vs. extension boundary) |
| Audit trail for downstream analysis | No | Yes (manifests, log capture, deterministic run IDs) |

An MLflow-only workflow tracks what happened. SNPTX controls what is allowed to happen and guarantees that it can be reproduced.

---

## SNPTX vs. DVC-Based Workflows

DVC provides dataset and model versioning with remote storage integration. SNPTX uses DVC for artifact persistence.

What DVC does not provide:

| Capability | DVC | SNPTX |
|---|---|---|
| DAG orchestration with parameterized sweeps | Limited (single-stage pipelines) | Yes (Snakemake multi-rule DAGs) |
| Experiment tracking with metric history | No (DVC experiments are separate) | Yes (MLflow integration) |
| Governed extension system | No | Yes |
| Contract-driven downstream analysis | No | Yes |
| Audit manifests for extension invocations | No | Yes |

A DVC-based workflow versions data. SNPTX versions data, orchestrates computation, tracks experiments, and governs downstream analysis within a unified architecture.

---

## Why Orchestration + Artifact Contracts Matter

The reproducibility crisis in biomedical ML is not caused by bad models. It is caused by:

1. **Uncontrolled analytical drift** — downstream analysis modifies interpretation without versioned inputs
2. **Implicit state dependencies** — pipelines rely on environment variables, undeclared files, or runtime ordering
3. **Absent governance** — multiple collaborators modify analytical logic without review boundaries
4. **Missing provenance** — results cannot be traced to specific inputs, configurations, and code versions

SNPTX addresses each of these root causes:

| Root Cause | SNPTX Response |
|---|---|
| Analytical drift | Extensions are downstream, artifact-only, and immutable |
| Implicit state | Configuration is explicit; DAG ordering is deterministic |
| Absent governance | Owner-mediated execution with contract validation |
| Missing provenance | Manifests, input hashing, git commit embedding, MLflow logging |

Orchestration provides the execution structure. Artifact contracts provide the governance boundary. Together, they create an environment where reproducibility is structural, not aspirational.

---

## Competitive Landscape

| System | Orchestration | Tracking | Versioning | Extension Governance | Determinism Guarantees |
|---|---|---|---|---|---|
| MLflow | No | Yes | Partial | No | No |
| DVC | Partial | Partial | Yes | No | No |
| Kubeflow | Yes | Yes | Partial | No | No |
| Metaflow | Yes | Yes | No | No | No |
| Weights & Biases | No | Yes | No | No | No |
| **SNPTX** | **Yes** | **Yes** | **Yes** | **Yes** | **Yes** |

SNPTX is the only framework in this landscape that combines orchestration, tracking, versioning, extension governance, and determinism guarantees within a single architectural design.

---

## Who This Is For

- **Research labs** conducting reproducible computational studies across collaborators
- **Pharma and biotech R&D teams** requiring auditable ML pipelines for regulatory submissions
- **Clinical AI developers** building tools that must satisfy institutional review and FDA expectations
- **Consortium projects** where multiple institutions contribute analysis without compromising execution integrity
- **Technical founders** building ML infrastructure for biomedical applications

---

## Contact

- drr508@g.harvard.edu
- dan@snptx.ai
