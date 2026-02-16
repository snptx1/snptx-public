# SNPTX Extension Governance Model

This document describes the governance architecture that controls how analytical extensions are developed, reviewed, and executed within the SNPTX framework. It defines the owner-mediated execution model, Tier-1 module contracts, artifact boundary discipline, and the collaboration philosophy that enables contribution without surrendering control.

---

## 1. Design Philosophy

SNPTX enables collaboration through constraint, not through openness. The framework distinguishes between two roles:

**Owner.** Controls pipeline orchestration, training logic, configuration, and extension invocation. The owner decides what runs, when it runs, and how results are consumed.

**Contributor.** Develops analytical extensions that operate strictly downstream of the core pipeline. Contributors define interpretation logic — how to analyze, normalize, summarize, or visualize evaluation artifacts. Contributors never control execution.

This separation is not organizational hierarchy; it is an architectural boundary. It exists because reproducibility, determinism, and provenance guarantees require that execution authority be concentrated and auditable.

---

![SNPTX Architecture](assets/snptx_architecture.png)

---

## 2. Owner-Mediated Execution

### How It Works

All extension execution flows through a single canonical entrypoint: the **owner-runner**. This runner:

1. Loads the extension contract (`extension.yaml`) from the extensions repository
2. Validates declared inputs and outputs against the contract schema
3. Verifies that all required input artifacts exist and satisfy type constraints
4. Invokes the extension module programmatically (not as a subprocess)
5. Captures `stdout` and `stderr` into persistent log files
6. Computes a deterministic run ID from a hash of extension metadata, input content, and configuration
7. Writes an auditable `manifest.json` recording provenance, timing, and hashes

### What Is Prohibited

- **Direct execution** of any extension script is forbidden. Running `python run.py` directly will fail by design.
- **Auto-discovery** of extensions is prohibited. Every extension invocation is explicit.
- **Dynamic rule generation** is not supported. Extension integration into the pipeline DAG is opt-in and owner-controlled.
- **Shared state** between extensions and core is prohibited. All communication occurs through persisted artifacts.

![Workflow DAG](assets/workflow_dag.png)

### Why This Matters

Owner-mediated execution ensures that the provenance chain is unbroken. Every extension output can be traced to specific inputs, specific configuration, and a specific invocation timestamp — without relying on the contributor's execution environment or runtime behavior.

---

## 3. Tier-1 Module Contracts

### Contract Structure

Every Tier-1 extension must declare an `extension.yaml` contract containing:

| Field | Purpose |
|---|---|
| `name` | Canonical identifier for the extension |
| `version` | Semantic version of the extension |
| `abi_version` | Contract ABI version (currently: 1) |
| `category` | Functional classification (e.g., evaluation, reporting) |
| `inputs` | Typed declaration of required input artifacts |
| `outputs` | Typed declaration of produced output artifacts |
| `config` | Schema for extension-specific configuration parameters |

### Input and Output Types

Artifact types are declared explicitly. The runner enforces basic type validation at invocation time:

- `evaluation_json` — must be valid JSON conforming to evaluation artifact schema
- `metrics_json` — canonical aggregated metrics
- `markdown` — human-readable Markdown output
- Custom types must be documented and stable within a contract tier

### Contract Stability

For the duration of each contract tier:
- Artifact paths remain valid
- Artifact semantics remain stable
- Breaking changes are not introduced
- Schema modifications require a version increment

Extensions that assume undocumented behavior, mutate upstream artifacts, or infer semantics from core source code are rejected.

---

## 4. Artifact Boundary Discipline

### Immutability

Artifacts are immutable once written. No extension may modify, overwrite, or delete artifacts produced by the core pipeline or by other extensions.

### Directionality

Data flows in one direction: from core to extensions. Extensions consume upstream artifacts; they do not feed back into the core pipeline (until the feedback loop architecture is implemented through its own governed contract).

### Isolation

Extension outputs are written to a dedicated directory within the core results structure:

```
results/extensions/<extension_name>/<run_id>/
├── <output_artifacts>
├── manifest.json
├── stdout.log
└── stderr.log
```

This isolation ensures that extension outputs are clearly distinguished from core pipeline outputs and can be independently audited, versioned, or discarded.

### Provenance

Every `manifest.json` records:
- Runner version and component identifier
- UTC timestamp
- Extension name, version, and ABI version
- SHA-256 hashes of all input artifacts
- Output paths and status
- Computed deterministic run ID
- Configuration snapshot

---

## 5. Collaboration Model

### How to Contribute

1. **Define the contract.** Write an `extension.yaml` declaring inputs, outputs, types, and configuration schema.
2. **Implement the module.** Write a `run.py` that accepts artifacts and configuration, produces deterministic outputs, and has no side effects beyond declared outputs.
3. **Submit for review.** Extensions are reviewed against the contract specification, determinism requirements, and artifact boundary rules.
4. **Execution is owner-mediated.** Upon acceptance, the owner integrates the extension into the execution workflow. The contributor does not control when or how the extension runs.

### What Contributors Control

- Analytical methodology within the extension
- Output formatting and structure (within declared types)
- Configuration parameters (within declared schema)
- Documentation and interpretation guidance

### What Contributors Do Not Control

- Execution timing and frequency
- Input artifact selection
- Integration with other extensions or pipeline stages
- Runtime environment and dependencies beyond declared requirements

### Why This Model Works

This governance model resolves a persistent tension in collaborative ML: the need for diverse analytical perspectives and the need for reproducible, auditable execution. By constraining the collaboration surface to deterministic, artifact-only, contract-governed modules, SNPTX enables contribution without introducing execution risk.

---

## 6. Current Extension Ecosystem

| Extension | Category | Status | Primary Output |
|---|---|---|---|
| `calibration_diagnostics` | Evaluation & Diagnostics | Complete | Calibration diagnostics JSON |
| `metric_aggregation` | Evaluation & Diagnostics | Complete | Canonical `metrics.json` |
| `evaluation_summary_report` | Reporting & Interface | Complete | Markdown summary report |
| `artifact_contract_checks` | Governance | In Development | Contract validation report |
| `template` | Reference | Complete | Demo output for onboarding |

---

## 7. Extension Development Guidelines

1. **Determinism is mandatory.** Identical inputs must produce identical outputs. No randomness, no network calls, no clock-dependent behavior.
2. **Artifacts are the only interface.** No shared databases, no environment variables, no implicit file paths.
3. **Contracts are the only specification.** Do not assume behavior not declared in `extension.yaml` or `artifact_semantics.md`.
4. **Outputs belong to the run directory.** All outputs must be written within the designated run directory unless explicitly overridden by the owner.
5. **Logging is captured, not controlled.** Write to stdout/stderr; the runner captures both.

---

## Contact

Extension governance questions or contribution proposals:
- drr508@g.harvard.edu
- dan@snptx.ai
