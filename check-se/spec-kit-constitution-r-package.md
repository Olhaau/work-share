# Constitution: SearchEngine Modernization to R Package

## 1) Document Control

- Name: SearchEngine R Migration Constitution
- Version: 1.0.0
- Status: Active Draft
- Owners: Migration Working Group
- Last Updated: 2026-05-21

This constitution defines mandatory engineering rules for migrating the legacy SearchEngine tool into a production-grade R package.

## 2) Intent and Scope

### Intent

Deliver a cross-platform R package that preserves the core matching behavior of SearchEngine while improving reproducibility, maintainability, and testability.

### In Scope

- Matching engine APIs for import, indexing/registry, weighted retrieval, scoring, filtering, and export.
- Reproducible search strategy execution (incremental and compound workflows).
- Optional ML post-filtering interfaces compatible with R workflows.
- Package quality, docs, tests, benchmarks, and release automation.

### Out of Scope (v1)

- Rebuilding the original FoxPro desktop GUI.
- Shipping or requiring Visual FoxPro runtime binaries.
- Full command-by-command legacy scripting compatibility.

## 3) Success Criteria

### Product Success

- Users can execute end-to-end matching pipelines entirely in R.
- Legacy strategy concepts remain available (weights, thresholds, containment, multi-run orchestration).

### Technical Success

- Core parity suite passes within agreed tolerance windows.
- Package passes cross-platform checks (Linux, macOS, Windows).
- Pipeline runs are deterministic when seed and config are fixed.

## 4) Non-Negotiable Principles

1. **Parity First**  
   The project MUST preserve core matching behavior before adding new algorithmic features.

2. **Reproducibility by Default**  
   All major operations MUST be scriptable and deterministic under fixed seeds/config.

3. **Explicit State**  
   Engine state MUST be represented in explicit R objects; hidden global state MUST NOT be required.

4. **Cross-Platform Execution**  
   Core package functionality MUST run on Linux, macOS, and Windows.

5. **No Runtime Dependency Mutation**  
   Package code MUST NOT install dependencies at runtime.

6. **Auditability**  
   Inputs, configuration, run metadata, and outputs MUST be inspectable and serializable.

7. **Performance With Guardrails**  
   Performance improvements MUST be backed by correctness tests and parity checks.

## 5) Architecture Rules

### 5.1 Layer Boundaries

- API Layer MUST expose stable user functions.
- Domain Layer MUST implement tokenization, registry weighting, scoring, and run logic.
- Execution Layer MUST handle iteration/parallelism without changing scoring semantics.
- I/O Layer MUST isolate import/export concerns from matching logic.

### 5.2 State Model

- The engine MUST use explicit objects (S3/S4/R6 acceptable).
- Objects MUST support `saveRDS()`/`readRDS()` round-trip without semantic loss.
- Mutable state transitions MUST be represented by explicit function calls.

### 5.3 Compatibility Contract

- A fixed benchmark corpus MUST be maintained for parity checks.
- Tolerance windows for ranking/identity deviations MUST be versioned and documented.

## 6) Dependency Policy

### Required

- R >= 4.2
- `data.table`
- `stringi`
- `jsonlite`
- `cli`

### Optional

- `arrow` for large-data interchange
- `future`/`future.apply` for parallel execution
- ML adapters (`xgboost`, `torch`, or `keras`) for optional post-filtering workflows

### Prohibited

- Runtime installation of packages from package functions.
- Hard dependency on FoxPro runtime, `searchengine.exe`, or Windows-only DLL bundles.

## 7) Quality and Release Gates

All gates MUST pass before release:

1. `R CMD check --as-cran` passes on Linux/macOS/Windows.
2. Core-domain test coverage is at least 85%.
3. Parity suite has no critical deviations outside published tolerances.
4. Benchmark suite satisfies documented runtime/memory SLOs.
5. Every exported function has examples and reference docs.
6. At least two vignettes exist: incremental strategy and compound strategy.

## 8) Security and Data Rules

- Package MUST NOT execute external shell commands implicitly.
- File writes MUST require explicit destination paths.
- Sample data in docs/tests MUST be synthetic or anonymized.
- Sensitive columns (if present) SHOULD support masking helpers in examples.

## 9) Delivery Phases and Exit Criteria

### Phase 0: Discovery and Parity Baseline

- Define API map from legacy concepts to R interfaces.
- Freeze benchmark datasets and expected outputs/tolerances.
- Exit: approved parity specification.

### Phase 1: Core Engine MVP

- Implement import, registry build, weighted scoring, threshold filtering.
- Exit: toy-corpus incremental flow passes parity checks.

### Phase 2: Strategy Completeness

- Implement containment, multi-run orchestration, refinement hooks.
- Exit: compound/self-referential workflows meet acceptance tests.

### Phase 3: Performance and Parallelism

- Optimize bottlenecks and add optional parallel backend.
- Exit: benchmark SLOs and regression tests pass.

### Phase 4: Packaging and Release Readiness

- Complete docs, vignettes, checks, and release process.
- Exit: CRAN-ready package candidate.

## 10) Governance and Change Control

- Architecture decisions MUST be recorded as ADRs under `docs/adr/`.
- Breaking API changes MUST include a mini-RFC and semver major bump.
- Any exception to this constitution MUST be documented with:
  - rationale,
  - owner,
  - expiry date,
  - rollback/closure plan.

## 11) Definition of Done

The migration is done when:

- users can run complete matching pipelines in R without legacy runtime dependencies,
- parity targets are met and continuously validated,
- cross-platform checks pass in CI,
- maintainers can evolve the package via documented, test-backed workflows.
