# SNPTX Core Assessment — Pre-Public Release

**Date:** 2026-02-16
**Classification:** Internal — Not for Public Distribution
**Scope:** snptx-core (main), snptx-extensions (tier1-evaluation-summary-report-spec)

---

## Executive Summary

SNPTX is an architecturally mature prototype with genuine structural strengths in pipeline orchestration, extension governance, and artifact-first design. The separation between core and extensions is conceptually rigorous and partially enforced. However, several infrastructure components remain aspirational or scaffolded rather than operational, and the documentation occasionally presents planned capabilities as if they were complete. This assessment identifies what is solid, what is partial, and what must be carefully framed for any public release.

**Recommended Positioning Tier: Architecturally Mature Prototype**

---

## 1. Architectural Maturity

### 1.1 Core–Extension Separation

**Finding: Materially enforced with minor inconsistencies.**

- The core repository (`snptx-core`) and extensions (`snptx-extensions`) are physically separated across Git repositories.
- Extension execution is mediated through a dedicated agentic execution layer (`workflow/scripts/run_extension.py`) that validates contracts, captures logs, and produces auditable manifests.
- Direct execution of extensions is forbidden by design — `run.py` files require import-based invocation through the runner.
- The Snakefile defines extension rules as opt-in (not part of `rule all`), maintaining clean separation.

**Gap:** The enforcement is by convention and auditability, not by sandboxing. The runner itself explicitly acknowledges this: *"This runner enforces architectural boundaries by convention and auditability, not by sandboxing."*

### 1.2 Deterministic Execution

**Finding: Intentional and partially implemented.**

- The Snakefile uses a deterministic split seed (configurable or derived from config hash).
- Extensions declare deterministic output contracts.
- Artifact semantics documentation states that identical inputs must produce identical outputs.

**Gap:** Determinism is a design constraint, not a verified property. There is no automated determinism check (e.g., running the same pipeline twice and comparing outputs byte-for-byte). The training script uses `LogisticRegression` without explicitly setting `random_state` on the model, though the solver convergence for logistic regression is typically deterministic. No randomness audit exists.

### 1.3 Artifact Contracts

**Finding: Partially structured, inconsistently schematized.**

- `docs/artifact_semantics.md` in snptx-extensions provides a Tier-1 frozen contract covering datasets, models, evaluation artifacts, reports, and MLflow artifacts.
- Extension contracts use `extension.yaml` with declared inputs, outputs, and types.

**Gap:** Two distinct schema formats coexist across extensions:
  - **v1 schema** (template, calibration_diagnostics, evaluation_summary_report): uses `version`, `abi_version`, `category`, `owner`, flat `inputs`/`outputs` maps. Minor inconsistency between `compliance` vs `provenance` key naming.
  - **v2 schema** (metric_aggregation, artifact_contract_checks): uses `tier`, `execution` block, `constraints` block, array-style input declarations. No `version` or `abi_version`.

This means the runner's contract validation may not consistently apply to all extensions. Schema convergence is needed before claiming stable Tier-1 contracts.

---

## 2. Reproducibility Integrity

### 2.1 DVC Integration

**Finding: Scaffolded but not operationally integrated with the primary pipeline.**

- `.dvc/` directory exists with configured remotes (S3: `s3://snptx-dvc-prod`, GCS: `gs://snptx-dvc-storage`).
- `dvc.yaml` defines a single `train` stage referencing `data/iris.csv` — a legacy placeholder unrelated to the active Synthea readmission pipeline.
- `.dvc` tracking files exist only for stub verification artifacts (empty files, zero bytes).
- `dvc.lock` exists at the repo root.

**Assessment:** DVC infrastructure is present but the primary Snakemake pipeline (Synthea readmission) runs independently of DVC. The two orchestration systems (Snakemake and DVC) are not integrated. This is a significant gap for reproducibility claims.

### 2.2 MLflow Run Consistency

**Finding: Runs are logged but with limited metadata richness.**

- 28 runs exist under experiment `readmission_prediction`.
- All sampled runs show consistent structure: FINISHED status, programmatic logging, correct experiment association.
- Run names indicate two categories: `summary_plot_accuracy` and `model_report`.
- Tags are empty across all runs — no git commit hashing, no version tagging, no dataset version references.

**Gap:** The README claims "Git commit embedding in reports" and the ROADMAP lists "Log Git hash for DVC" as complete, but MLflow run tags contain no git commit references. Model registration appears to log to MLflow, but version-to-commit linkage is not confirmed in the run metadata.

### 2.3 Artifact–Git Linkage

**Finding: Not implemented.**

Artifacts are not version-linked to specific git commits via MLflow tags or DVC references. While reports embed commit hashes (via `generate_model_report.py`), MLflow runs themselves carry no commit provenance.

---

## 3. Extension Governance

### 3.1 Agentic Execution Boundary

**Finding: Technically enforced by convention, architecturally sound.**

- The execution layer validates contracts, captures stdout/stderr, produces manifests, and computes deterministic run IDs from hashed inputs.
- The contract validation checks ABI version, required keys, input/output types, and artifact type declarations.
- Extensions cannot be directly executed — they must be imported and invoked via the runner.

**Gap:** Enforcement is by convention, not containment. On a shared machine, any user could bypass the runner. This is acknowledged in the code but should not be presented as security isolation.

### 3.2 Tier-1 Contract Stability

**Finding: Partially stable, schema convergence incomplete.**

- The artifact semantics contract (`docs/artifact_semantics.md`) is well-written and declares a Tier-1 freeze scope.
- Extension contracts exist for all four non-template extensions.
- However, schema formats diverge (v1 vs v2 as described above).
- The `artifact_contract_checks` extension exists in snptx-extensions but has not been merged to `main` — it remains on a feature branch.

**Assessment:** The governance model is architecturally mature in concept. The schema inconsistency undermines the claim of stable, schema-driven contracts. Convergence to a single schema format is a prerequisite for public positioning as "stable."

---

## 4. Self-Learning Position

### 4.1 Feedback Loop

**Finding: Not implemented. Entirely aspirational.**

- The architecture diagram shows a "Feedback Loop" between reporting and automation layers.
- Phase 5 of the ROADMAP lists "Intelligent summarization extension" and "Insight synthesis" as planned.
- No code, no extension, no artifact structure, and no contract exists for any feedback mechanism.
- No active learning, no automated retraining, no hypothesis suggestion infrastructure.

### 4.2 Requirements for "Intelligent Layer" Readiness

To credibly claim intelligent-layer readiness, the following would be minimally required:

1. A stable, schema-defined feedback artifact (e.g., `feedback.json`) emitted by evaluation extensions.
2. A feedback ingestion interface in the core framework.
3. At least one reference implementation demonstrating closed-loop artifact flow.
4. Governance controls ensuring feedback does not violate determinism guarantees.

None of these exist. The "Feedback Loop" shown in the architecture diagram is a design intent, not a capability.

---

## 5. Differentiation

### 5.1 vs. Standard MLflow + Snakemake Project

**Meaningful differences:**
- Physically separated extension repository with contract-driven execution.
- Owner-mediated runner with manifest-based auditability.
- Artifact semantics contract formalizing Tier-1 stability guarantees.
- Explicit execution boundary preventing unauthorized modification of core pipeline state.

**Not meaningfully different:**
- Training pipeline (logistic regression with parameter sweeps).
- MLflow tracking integration (standard programmatic logging).
- Snakemake DAG structure (conventional rules with shell commands).
- CI configuration (dry-run only, no actual pipeline execution in CI).

### 5.2 vs. Simple DVC-Based Workflow

**Meaningful differences:**
- Snakemake orchestration layer provides DAG-based execution beyond DVC's pipeline stages.
- Extension system adds an agentic execution surface not present in DVC workflows.
- Artifact semantics formalize expectations beyond DVC's file-tracking model.

**Not meaningfully different:**
- DVC is scaffolded but not integrated with the primary pipeline. The Snakefile is the actual orchestrator.
- DVC remote storage is configured but not actively used for the Synthea pipeline's artifacts.

### 5.3 Genuine Differentiator

The primary architectural contribution is the **extension execution model**: a contract-driven, agent-mediated execution surface that separates analytical interpretation from pipeline execution. This is a non-trivial design decision that distinguishes SNPTX from standard MLOps tooling assemblies.

---

## 6. Public Narrative Alignment

### 6.1 What Can Be Confidently Presented as Complete

| Capability | Evidence |
|---|---|
| Snakemake pipeline orchestration | Fully operational Snakefile with 8 rules, deterministic seed |
| MLflow experiment tracking | 28 runs logged, experiment metadata, model artifacts |
| Extension execution model | Agentic execution layer with contract validation, manifest generation, log capture |
| Artifact semantics specification | Frozen Tier-1 contract document |
| Separate core/extension repositories | Physical separation enforced |
| Three reference Tier-1 extensions | calibration_diagnostics, metric_aggregation, evaluation_summary_report |
| Configuration-driven pipeline | YAML-based config with dataset, feature, and parameter definitions |
| CI pipeline (dry-run validation) | GitHub Actions workflow exists |

### 6.2 What Must Be Framed as Roadmap

| Capability | Actual Status |
|---|---|
| DVC artifact management (integrated) | Scaffolded only; not integrated with primary pipeline |
| Artifact-to-git-commit linkage | Not implemented in MLflow metadata |
| Self-learning / feedback loop | Design intent only; no code or contracts |
| Intelligent summarization | Not started |
| Dashboards and APIs | Not started |
| Active learning | Not started |
| Multi-dataset expansion | Config supports it; only Synthea readmission is operational |
| Schema-driven Tier-1 contracts | Two incompatible schema formats coexist |
| Artifact contract validation extension | Exists but not merged to main |
| Full CI pipeline execution | CI only performs dry-run |

### 6.3 What Must Not Be Implied

| Claim | Risk |
|---|---|
| "DVC fully integrated" | DVC pipeline definition is a placeholder (Iris dataset); primary pipeline bypasses DVC |
| "Deterministic execution guaranteed" | Determinism is a design goal, not a verified property |
| "Feedback loop operational" | No code exists |
| "Commercial-ready" | No deployment infrastructure, no API layer, no security hardening |
| "Deep learning framework" | Training is logistic regression only; no deep learning code exists |
| "Knowledge graph support" | No knowledge graph code, embeddings, or NLP infrastructure exists |
| "Multi-modality support" | Only tabular clinical classification is implemented |
| "Schema-driven contracts" | Schema formats are inconsistent across extensions |

---

## 7. Structural Strengths

1. **Extension governance model** is architecturally sound and genuinely novel relative to standard MLOps assemblies.
2. **Artifact semantics documentation** is rigorous and well-written.
3. **Agentic execution layer** is production-quality code with deterministic run IDs, contract validation, and audit logging.
4. **Documentation coherence** across README, ARCHITECTURE, ROADMAP, and DEVKIT_NOTES is high — documents reference each other consistently.
5. **Pipeline structure** is clean, well-organized, and follows Snakemake best practices.
6. **Separation of concerns** between orchestration and interpretation is genuine, not cosmetic.

---

## 8. Structural Gaps

1. **DVC–Snakemake integration gap**: The two orchestration/versioning systems operate independently. The primary pipeline uses Snakemake; DVC is configured but idle for the main workflow.
2. **Extension schema divergence**: Two incompatible contract formats undermine stability claims.
3. **MLflow metadata poverty**: Runs lack git commit tags, dataset version references, and promotion stage metadata.
4. **CI is dry-run only**: No actual pipeline execution or artifact validation in CI.
5. **Single model type**: Only logistic regression is implemented; claims of supporting "CNN and transformer-based pipelines" or "deep learning" are aspirational.
6. **Single dataset**: Only Synthea readmission is operational; Iris is a DVC placeholder.
7. **Verification infrastructure empty**: `verified_index.json` is `[]`; `verified_summary.md` has header only.
8. **snptx-extensions main branch behind**: Feature branch is 5 commits ahead of main; latest extensions not merged.

---

## 9. Narrative Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Claiming DVC integration when it's scaffolded | High | Frame as "DVC-ready" or "DVC infrastructure configured" |
| Implying deep learning capability | High | Remove or explicitly mark as future |
| Presenting feedback loop as existing | Medium | Present as architectural design target, not capability |
| Claiming multi-modality support | High | Describe as configuration-extensible, not implemented |
| Overstating determinism guarantees | Medium | Describe as design principle, not verified property |
| Implying commercial readiness | High | Position as research framework, not production system |

---

## 10. Recommended Positioning Tier

### **Architecturally Mature Prototype**

**Rationale:**

- The core pipeline is operational and produces real artifacts.
- The extension governance model is architecturally novel and partially enforced.
- Documentation quality exceeds typical prototype-level projects.
- However, key infrastructure components (DVC integration, commit linkage, schema convergence, CI execution, feedback loops) remain incomplete.
- No deployment, API, or user-facing interface exists.
- Single model type and single dataset limit generalizability claims.

This positions SNPTX above "early prototype" (the pipeline works, the governance model is implemented) but below "pre-deployment research framework" (too many infrastructure gaps for lab deployment without significant additional work).

**For public presentation**, the honest and strategically optimal framing is:

> *A research-grade ML orchestration framework with a novel extension governance model, designed for reproducible biomedical workflows. Core pipeline operational; extension ecosystem and infrastructure integrations under active development.*

---

**End of Assessment**
