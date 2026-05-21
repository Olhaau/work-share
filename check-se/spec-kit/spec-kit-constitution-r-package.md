# Constitution: SearchEngine Modernization to R Package

## 1) Document Control

- Name: SearchEngine Modernization Constitution (R Package)
- Version: 2.0.0
- Status: Active
- Owners: SearchEngine Modernization Working Group
- Last Updated: 2026-05-21

This constitution defines mandatory engineering policy for modernizing the legacy SearchEngine desktop workflow into a production-grade R package.

## 2) Intent and Scope

### Intent

The program MUST deliver a cross-platform R package that preserves SearchEngine behavior while improving reproducibility, maintainability, and automation readiness.

### In Scope

- Matching engine APIs for import, indexing/registry, weighted retrieval, scoring, filtering, and export.
- Reproducible strategy execution for incremental and compound workflows.
- Optional ML post-filter interfaces for R-native pipelines.
- Packaging quality: tests, docs, benchmarks, CI gates, and release process.

### Out of Scope

- Rebuilding the legacy FoxPro desktop GUI.
- Shipping or requiring Visual FoxPro runtime binaries.
- Command-by-command compatibility with legacy macro scripts.

## 3) Success Criteria

### Business Success

- Users MUST be able to run end-to-end matching pipelines fully in R without desktop runtime dependencies.
- Core legacy strategy concepts MUST be preserved: weights, thresholds, containment, and multi-run orchestration.

### Technical Success

- Parity suite MUST pass within published tolerance windows.
- CI MUST pass on Linux, macOS, and Windows.
- Runs MUST be deterministic when seed, inputs, and config are fixed.
- Release artifacts MUST be reproducible from tagged source.

## 4) Principles

1. Parity First: behavior parity MUST be delivered before net-new matching features.
2. Reproducibility by Default: deterministic execution MUST be the default for all core workflows.
3. Explicit State: hidden global state MUST NOT drive matching behavior.
4. Clear Boundaries: matching logic MUST remain isolated from I/O and orchestration concerns.
5. Cross-Platform Confidence: core behavior MUST be validated on Linux, macOS, and Windows.
6. No Runtime Mutation: package code MUST NOT install, update, or remove dependencies at runtime.
7. Auditability: inputs, config, run metadata, and outputs MUST be serializable and inspectable.
8. Performance With Safety: optimization MUST NOT bypass correctness and parity controls.

## 5) Architecture Rules

### State Model

- Engine state MUST live in explicit R objects (S3, S4, or R6).
- Stateful transitions MUST occur only through named package APIs.
- State objects MUST support `saveRDS()` and `readRDS()` round-trip without semantic loss.

### Layering and Boundaries

- API Layer MUST expose user-facing functions and stable contracts only.
- Domain Layer MUST own tokenization, registry weighting, scoring, and filtering semantics.
- Execution Layer MUST orchestrate iteration and optional parallelism without changing domain semantics.
- I/O Layer MUST handle import/export and MUST NOT contain matching decisions.
- Cross-layer calls MUST follow API -> Domain -> Execution/I-O; reverse dependencies MUST NOT be introduced.

### Extension Model

- Extension points MUST be explicit (for example: custom scorers, post-filters, and exporters).
- Extensions MUST use versioned interfaces and capability checks.
- Optional extensions MAY add behavior but MUST NOT alter default scoring semantics.
- ML adapters MUST be optional and MUST degrade gracefully when not installed.

## 6) Dependency Policy

### Required

- R >= 4.2
- `data.table`
- `stringi`
- `jsonlite`
- `cli`

### Optional

- `arrow` for high-volume interchange.
- `future` and `future.apply` for optional parallel execution.
- `xgboost`, `torch`, or `keras` for optional post-filtering adapters.

### Prohibited

- Runtime package installation inside package functions.
- Hard dependency on FoxPro runtime, `searchengine.exe`, or Windows-only DLL bundles.
- Silent network fetches during core matching execution.

## 7) Quality and Release Gates

All gates MUST pass before merge to release branch and before tag creation.

1. `R CMD check --as-cran` passes on Linux, macOS, and Windows in CI.
2. Core domain test coverage MUST be at least 85%.
3. Parity suite MUST show zero critical deviations outside versioned tolerances.
4. Benchmark suite MUST meet published runtime and memory SLOs.
5. Every exported function MUST include examples and reference docs.
6. At least two vignettes MUST exist: incremental strategy and compound strategy.
7. Build reproducibility check MUST recreate equivalent artifacts from the same commit.

Implementation notes:

- Gate implementations SHOULD be automated in CI and executable locally.
- Any temporary gate waiver MUST follow Section 10 exception handling.

## 8) Security and Data Handling Rules

- Package code MUST NOT execute external shell commands implicitly.
- File writes MUST require explicit destination paths provided by the caller.
- Test and example datasets MUST be synthetic or anonymized.
- Sensitive fields SHOULD support masking helpers in examples and logs.
- Telemetry MUST be opt-in and MUST NOT collect raw record payloads.

## 9) Delivery Phases and Exit Criteria

### Phase 0: Discovery and Parity Baseline

- Define mapping from legacy concepts to R interfaces.
- Freeze benchmark datasets and expected outputs with tolerance windows.
- Exit criteria: approved parity specification and baseline CI job.

### Phase 1: Core Engine MVP

- Implement import, registry build, weighted scoring, and threshold filtering.
- Exit criteria: incremental toy-corpus workflow passes parity checks.

### Phase 2: Strategy Completeness

- Implement containment, multi-run orchestration, and refinement hooks.
- Exit criteria: compound and self-referential workflows pass acceptance tests.

### Phase 3: Performance and Parallelism

- Optimize known bottlenecks and add optional parallel backend.
- Exit criteria: benchmark SLOs pass with no parity regressions.

### Phase 4: Packaging and Release Readiness

- Finalize docs, vignettes, checks, release automation, and migration notes.
- Exit criteria: CRAN-ready release candidate and signed release checklist.

## 10) Governance and Change Control

- Architecture decisions MUST be recorded as ADRs in `docs/adr/`.
- Breaking API changes MUST include a mini-RFC and semver-major plan.
- Constitution exceptions MUST include rationale, owner, expiry date, and rollback plan.
- Expired exceptions MUST be closed or renewed by a new decision record.
- Constitution changes MUST increment version and include a change note.

## 11) Definition of Done

Modernization is complete only when all statements below are true:

- Users can execute complete matching pipelines in R without legacy runtime dependencies.
- Parity targets are met and continuously validated in CI.
- Cross-platform checks pass for supported operating systems.
- Maintainers can evolve the package through documented, test-backed, reproducible workflows.
- Release and rollback procedures are documented and exercised at least once.
