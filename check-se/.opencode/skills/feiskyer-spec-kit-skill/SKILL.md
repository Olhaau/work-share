---
name: feiskyer-spec-kit-skill
description: Use ONLY when creating or revising a spec-kit constitution, project charter, engineering governance baseline, or migration constitution (keywords: constitution, spec-kit, charter, principles, governance, quality gates, ADR).
---

# Feiskyer Spec-Kit Skill

This skill defines a practical constitution format for software modernization projects.

## Goal

Produce constitutions that are:

- explicit about constraints and non-negotiables,
- implementation-oriented (not abstract policy prose),
- measurable through gates and exit criteria,
- maintainable through change control.

## Required Output Shape

Every constitution must include these sections in order:

1. Document Control (name, version, status, owners, last updated)
2. Intent and Scope (in scope / out of scope)
3. Success Criteria (business and technical)
4. Principles (5-9 non-negotiable principles)
5. Architecture Rules (state model, layering, boundaries, extension model)
6. Dependency Policy (required, optional, prohibited)
7. Quality and Release Gates (must-pass checks)
8. Security and Data Handling Rules
9. Delivery Phases and Exit Criteria
10. Governance and Change Control (ADR, versioning, exceptions)
11. Definition of Done

## Writing Rules

- Use normative language: MUST, MUST NOT, SHOULD, MAY.
- Keep each rule testable and observable.
- Separate policy from implementation notes.
- Avoid GUI-centric assumptions for automation-first projects.
- Preserve legacy parity goals but avoid locking into obsolete runtime dependencies.

## Modernization Focus (Legacy Desktop -> Package)

When the project is a migration from legacy tooling to a package ecosystem:

- prioritize behavioral parity over binary/runtime parity,
- codify reproducibility and deterministic pipelines,
- require cross-platform checks,
- ban runtime dependency installation from inside package code,
- include compatibility benchmark datasets and tolerance windows.

## Minimal Constitution Checklist

Before finalizing, verify:

- Non-negotiables are explicit and numbered.
- Gates are executable by CI.
- Phases have clear exits.
- Exception path exists and is auditable.
- Definition of done is concrete and user-outcome based.
